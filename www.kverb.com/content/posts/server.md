---
title: "server notes"
date: 2022-06-11T12:45:58-04:00
draft: false
toc: false
images:
tags:
  - untagged
category:
  - notes
---

Very loosely organized notes of server builds.

## Plex server

Ideally 1U or 2U. Probably should be an intel chip that supports "Quick Sync" for efficient video transcoding.
Supermicro barebones seems to have the ideal chassis+mobo combo.
Must have 10Gbe or SFP+

### 1U barebones/prebiult supermicro

- mini, low power [SYS-E300-9C](https://www.supermicro.com/en/products/system/Mini-ITX/SYS-E300-9C.cfm)
  - with coffee lake i7 can this handle transcodes?
- lol $5k [SYS-110P-FRDN2T](https://www.supermicro.com/en/products/system/IoT/1U/SYS-110P-FRDN2T)
  - 16" depth
  - uses [LGA-4189 socket](https://www.newegg.com/p/pl?N=100161260%20601386431)
- [SYS-110C-FHN4T](https://www.supermicro.com/en/products/system/IoT/1U/SYS-110C-FHN4T)
  - 12" depth
  - uses cheaper CPU [LGA-1200 socket](https://www.newegg.com/p/pl?N=100161260%20601356222)
- [5019c-l](https://www.thinkmate.com/system/superserver-5019c-l)
  - doesn't have 10Gbe nic ; can probably be added later

### DIY

Should probably go with intel 1151 socket, based on [serverbuilds.net](https://forums.serverbuilds.net/t/guide-hardware-transcoding-the-jdm-way-quicksync-and-nvenc/1408) and [Perfect Media Server guy](https://forums.serverbuilds.net/t/guide-hardware-transcoding-the-jdm-way-quicksync-and-nvenc/1408)

#### Final candidate build (~$2425)

- [$129 MSI B560M PRO-VDH WIFI LGA 1200 Intel B560 SATA 6Gb/s Micro ATX Intel Motherboard](https://www.newegg.com/p/N82E16813144397?Item=N82E16813144397)
- [$215 Intel Core i5-11400 - Core i5 11th Gen Rocket Lake 6-Core 2.6 GHz LGA 1200 65W](https://www.newegg.com/intel-core-i5-11400-core-i5-11th-gen/p/N82E16819118241?Item=9SIA12KE9X7943&quicklink=true)
  - [$69 Thermaltake Engine 27 1U Low-Profile 70W Intel 60mm Low Noise PWM Fan Forty Fan Blade CPU Cooler](https://www.newegg.com/thermaltake-engine-27-cl-p032-ca06sl-a/p/N82E16835106417?Item=N82E16835106417)
- [$272 (2x$136) Crucial 32GB Single DDR4 3200 MT/s CL22 DIMM 288-Pin Memory](https://www.newegg.com/crucial-32gb-288-pin-ddr4-sdram/p/N82E16820156237?Item=9SIAD8UFCD5687)
- [$99 SAMSUNG 980 M.2 2280 1TB PCI-Express 3.0 x4, NVMe 1.4 V-NAND MLC Internal Solid State Drive](https://www.newegg.com/samsung-1tb-980/p/N82E16820147804?Item=N82E16820147804)
- [$199 PLINKUSA 2U Rackmount (2x5.25+2x3.5 HD+2x2.5 HD)(15.16" Deep)](https://www.amazon.com/gp/product/B01LYQE6R5)
- [$95 CORSAIR RM650 650 W ATX 80 PLUS GOLD Certified Full Modular Power Supply](https://www.newegg.com/p/N82E16817139279?Item=N82E16817139279)
- [(already purchased) $1375 ($687x2) SAMSUNG 870 QVO Series 2.5" 8TB SATA III Samsung 4-bit MLC V-NAND Internal Solid State Drive (SSD)](https://www.newegg.com/samsung-8tb-870-qvo-series/p/N82E16820147784?Item=9SIA12KC574074)
- sata cables already ordered
- #TODO: 10Gbe/SFP+ card


#### Potential build:

- mobo/cpu opt 1
  - [$249 AsRock Rack E3C246D4U ](https://www.newegg.com/asrock-rack-e3c246d4u-supports-intel-xeon-e-2100processor/p/N82E16813140021?Item=N82E16813140021)
  - [$299 Intel Xeon E-2126G Coffee Lake](https://www.newegg.com/intel-xeon-e-2126g-lga-1151/p/N82E16819118002?Item=9SIA25VBCR6166)
- mobo/cpu opt 2
  - [$129 MSI socket 1200](https://www.newegg.com/intel-core-i5-10400-core-i5-10th-gen/p/N82E16819118135?quicklink=true)
  - [$175 core i5 10th gen (rocket lake) 65W](https://www.newegg.com/intel-core-i5-10400-core-i5-10th-gen/p/N82E16819118135?quicklink=true)
- [$99 SAMSUNG 980 M.2 2280 1TB](https://www.newegg.com/samsung-1tb-980/p/N82E16820147804?Item=N82E16820147804)
- [$199 2U PlinkUSA chassi](https://www.amazon.com/gp/product/B01LYQE6R5/)
  - I dont think the [Noctua low-profile](https://www.newegg.com/noctua-nh-l9i/p/N82E16835608029?Item=N82E16835608029) will fit in 1U
- [$280 4x16 GB RAM](https://www.amazon.com/gp/product/B0736W5BH2/)
- [$95 650W PSU](https://www.newegg.com/p/N82E16817139279)


## NAS

- Can probably DIY
- need to understand SAS vs SATA; physical disc connection limitations
- apparently [6 2.5" ssd can fit in 1 5.25" drive bay](https://www.amazon.com/ICY-DOCK-Mobile-Comparable-Tray-less/dp/B01M0BIPYC/)
- chassis:
  - rosewill, plinkusa, chenbro, silverstone seem to be the only short-depth ones
  - [stylish 2U 16" deep PLinkUSA](https://www.amazon.com/PLINKUSA-RACKBUY-Micro-ATX-Rackmount-IPC-E239B/dp/B01LYQE6R5)
  - [2 x 5.25" front drive bays 2U short PLinkUSA](https://www.amazon.com/PLINKUSA-RACKBUY-Rackmount-Micro-ATX-IPC-2022M/dp/B01M09ZO7P/)
    - 15" depth
    - uses full ATX PSU, which seems weird
  - [2U Athena power](https://www.newegg.com/black-athena-power-rm-2u200hr65u2/p/N82E16811192388)
    - 14" depth
    - **has PSU** AND 2 x 5.25" front drive bays; could usewith icy dock

- mobo
  - epyc 3000 series? SoC , lots of PCI "lanes", fuckton of RAM, and low power. Doesn't typically include 10Gbe
  - Intel Xeon D series SoC, typically includes 10Gbe
    - [Gigabyte Xeon D-1500](https://www.gigabyte.com/us/Enterprise/Server-Motherboard/MB10-DS4-rev-13)
  - most seem to use OCUlink or sas breakout cables to connect SATA drives
  - high end SoC: [AsRock Rack D2143D4I2-2T Mini-ITX Xeon D-2100](https://www.newegg.com/asrock-d2143d4i2-2t/p/N82E16813157904)
  - overkill option: [ASRock Rome Epyc 7000 series](https://www.newegg.com/asrock-rack-romed4id-2t-amd-epyc-7002-series-processors/p/N82E16813140059)
    - would require a 7000 series CPU; uses too much power
  - [AsRock Rack X570D4U-2L2T](https://www.newegg.com/asrock-rack-x570d4u-2l2t-supports-3rd-gen-amd-ryzen-processors-and-2nd-gen-amd-ryzen-processors-with/p/N82E16813140056?Description=X570D4U-2L2T&cm_re=X570D4U-2L2T-_-13-140-056-_-Product&quicklink=true)
    - ryzen zen 3
    - has dual 10Gbe & 8 SATA ports


### Premade systems

- [1U Supermicro SYS-5018D-FN4T](https://www.supermicro.com/en/products/system/1U/5018/SYS-5018D-FN4T.cfm)
  - dual 10Gbe
  - 6 SATA ports
  - M Key 2242/2280
  - might be able to just put SSDs in the empty space inside the chassis, or into an [Icy Dock cage](https://www.newegg.com/icy-dock-mb326sp-b-6-x-2-5-sata-6gbps-sas-hdd-ssd-mobile-rack-cage-in-1-x-5-25-bay/p/N82E16817198068?Description=icy%20dock%205.25&cm_re=icy_dock%205.25-_-17-198-068-_-Product)
- [1U ASRock ryzen in a supermicro chassis](https://www.newegg.com/p/2NS-000A-0FDT1)
  - kinda strange, but has dual 10Gbe & 8 SATA ports. 
