---
title: "nginx metrics and log collection with grafana, prometheus, loki"
date: 2022-06-11T12:45:58-04:00
draft: false
showTableOfContents: true
category:
  - notes
summary: "A primer on how to enable metrics collection and monitoring for Nginx using the grafana stack"
---

## meta

**Grafana** - is the front-end dashboard website, that supports querying datasources, and displaying visualizatoins of the data. It is basically a full-stack webapp that you can self-host.

**Prometheus** - is a metrics collection endpoint. It runs as an http server that also “scrapes” target services, periodically via http. It becomes the primary data source for counters/timeseries when using grafana dashboards. Conceptually, it is the main database for recording and facilitating queries against the metrics.

**Nginx Node exporter** - it’s a confusing name, but in order for prometheus to scrape any useful data, it needs to be available to scrape on the target. The “node exporter” is what makes metrics data available for prometheus to scrape. There are many node exporters available for popular software services, including nginx.

**Loki** - is like prometheus, but for text logs. I.e., it receives a stream of text, generally the recent lines of a log file. The encoding and format of each line can be arbitary. Loki will parse them as-is. Like prometheus, it becomes the primary datasource for log-based data and queries.

**Promtail** - is a service that runs on the target node. It is what pushes data into Loki (Unlike prometheus, loki does not operate on a pull/scrape model). You can also think of the target as the “source of the logs data”. In this case, our target/source of data is our nginx metrics and logs. Promtail needs read access to the raw log files, and then streams it to Loki. Because of this, I run promtail directly on the nginx container.

## assumptions / pre-conditions

- grafana, prometheus, and loki are already up and running correctly. These can/should be run on a separate container from nginx.
- nginx is on a tailscale network that can access prometheus and Loki nodes
  - in my setup, loki runs on the prometheus node
  - this isn’t necessary; it just makes resolving the hosts secure and easy
- You’re not using something like docker containers. If you are, there are plenty of guides out there on how to do that.

## Set up the node exporter

There are probably distro packages/procedures for installing the nginx node exporter. Here’s how you can manually do it on linux. I do this from a root shell on the nginx container. The overall process is: download the ‣ binary release, unzip and install the executable somewhere, create a systemd service definition.

```bash
# download the binary
cd ~/downloads
wget https://github.com/nginxinc/nginx-prometheus-exporter/releases/download/v0.10.0/nginx-prometheus-exporter_0.10.0_linux_amd64.tar.gz -O ~/downloads/npe.tar.gz
# un-archive
tar -xvf npe.tar.gz
# copy the binary to somewhere sane. Here i've chosen /opt/
cp nginx-prometheus-exporter /opt/
```

We’ll need to make a systemd service definition file. Here’s mine:

```bash
$ cat /etc/systemd/system/nginx-prometheus-exporter.service
[Unit]
Description=Prometheus Node Exporter for Nginx
ConditionFileIsExecutable=/opt/nginx-prometheus-exporter

After=syslog.target network-online.target 

[Service]
StartLimitInterval=5
StartLimitBurst=10
ExecStart=/opt/nginx-prometheus-exporter
SyslogIdentifier=prometheus

WorkingDirectory=/opt/

Restart=always

RestartSec=10

[Install]
WantedBy=multi-user.target
```

In order for the node exporter to provide useful metrics, we need to enable the `stub_status` module for nginx. It’s basically a special built-in http endpoint that can be exposed through your nginx conf. This is the relevant snippet, that should go in a `sites-enabled` or in the root conf:

```bash
server {
	server_name nginx;
	listen 8080;  # i'm not sure this port can be changed easily
	location = /stub_status {
		stub_status;
	}
}
```

On the prometheus node, enable the scraping target 

```bash
$ cat /etc/prometheus/prometheus.yml
< ... >
- job_name: "nginx_node"
    static_configs:
      - targets: ["nginx:9113"]  # NOTE: this needs to point to your nginx host
```

## Loki + Promtail

As stated in the preface, in order to make the the nginx logs accessible in grafana+loki, we need a promtail instance that can read the raw logs files. On docker-style containers, there’s various incantations to bind-mount the nginx logs directories into a promtail container, but I think it’s just much more straight-forward to install promtail directly in the nginx container. Again, there’s probably a simple and straighforward way to install promtail via your distro’s package manager, but here are the manual steps:

```bash
# download the binary
cd ~/downloads
wget https://github.com/grafana/loki/releases/download/v2.4.2/promtail-linux-amd64.zip -O ~/promtail.zip
# unzip and copy to a sane place
unzip promtail.zip
mkdir /opt/promtail
cp /opt/promtail/promtail
```

Promtail requires a small config file. You can put it anywhere, but I’ll put it in `/opt/promtail`

```bash
$ cat /opt/promtail/promtail.yml 
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
                # make sure this host is your LOKI hostname
  - url: http://prometheus:3100/loki/api/v1/push

scrape_configs:
  - job_name: system
    static_configs:
    - targets:
        - localhost
      labels:
        job: varlogs
        __path__: /var/log/*log
  - job_name: nginx
    static_configs:
    - targets:
        - localhost
      labels:
        job: nginx
        __path__: /var/log/nginx/*log
```

And create a systemd service definition:

```bash
$ cat /etc/systemd/system/promtail.service
[Unit]
Description=promtail is the agent responsible for gathering logs and sending them to Loki.
Documentation=https://grafana.com/docs/loki/latest/clients/promtail/
Requires=network-online.target
After=network-online.target

[Service]
Type=simple
ExecStart=/opt/promtail/promtail -positions.file /var/lib/promtail/positions.yml -config.file /opt/promtail/promtail.yml
User=promtail
TimeoutStopSec=30s

NoNewPrivileges=true
MemoryDenyWriteExecute=true
RestrictRealtime=true

ProtectHome=true
ProtectSystem=strict
ReadWritePaths=/var/lib/promtail
PrivateTmp=true

PrivateDevices=true
ProtectKernelTunables=true
ProtectKernelModules=true
ProtectControlGroups=true

[Install]
WantedBy=multi-user.target
```

Reload systemd and enable and start the new services:

```bash
$ systemctl daemon-reload
$ systemctl enable --now promtail
$ systemctl enable --now nginx-prometheus-exporter
```

You can verify nginx-prometheus-exporter is working by checking its http endpoint: [http://nginx:9113/metrics](http://nginx:9113/metrics)

And promtail: [http://nginx:9080/targets](http://nginx:9080/targets)

## Dashboards in grafana

I’m using dashboard queries and viz based on [this dashboard](https://grafana.com/grafana/dashboards/13865). It requires some slight modifications to the queries to get accurate data. A lot of the queries have extra filters that are specific to the original author.