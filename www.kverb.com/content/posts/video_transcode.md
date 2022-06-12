---
title: "video transcoding"
date: 2022-06-11T12:45:58-04:00
draft: false
showTableOfContents: true
category:
  - notes
---

## meta

These are some notes gathered on my exploration of transcoding videos for more reliable playback.
This doesn’t cover the downloading of the video files; just post-processing them into different formats for playback, primarily on an Apple TV 4k (2019).

Hardware used: 

- AMD Ryzen 7 3800X 8-Core Processor + NVIDIA GeForce RTX 2060 SUPER
- 11th Gen Intel(R) Core(TM) i5-11400 @ 2.60GHz + (iGPU)

## overview

The main take-aways are:

- Most titles downloaded from the internet are h264 encoded, mkv container files.
- Audio codecs vary and subtitles are complete anarchy.
- AppleTV (my primary playback client) can play the h264 encoded stream generally without issue.
  - It requires on-the-fly remux from MKV to MP4 and (usually) audio transcoding, but generally uses very little cpu on the plex server.
- HEVC / h265 *can* be more space efficient for the same quality, so converting the library to it is worthwhile.
  - However, there are observable limits to how much space can be saved for a given quality level.
  - The main heuristic is thus:
    - if the source file is a high-bitrate, h264 or original iso, then convert to HEVC at about 10mbps (for 1080p).
    - In general, the quality trade off (as well as the energy required to process the transcode) just doesn’t make sense if the file is already below an avg bitrate of 15 mbps.
    - Also, a “how much do i like this movie” factor should be considered for if and how much quality loss is acceptable.
- [Prefer libx265 (cpu / software) based transcodes to GPU-assisted transcodes.](https://codecalamity.com/hardware-encoding-4k-hdr10-videos/#has-hevc-hardware-encoding-caught-up-to-the-quality-of-software-encoding) GPU is ideal for real-time transcoding.
- Although it will take much longer, x265 will yield higher quality at smaller space.
- Consider GPU transcodes for “less important” titles; i.e., movies and series that don’t need “archival quality”.
- Subtitles are ... annoying if converting from mkv to mp4.
  - The [“other transcoding”](https://github.com/donmelton/other_video_transcoding) script seems to handle this, but because mp4 lacks support for lots of subtitle formats, there are still limitations.

## Donmelton's other_video_transcode tool

Use this script to generate an ffmpeg command that can be executed on another system, if needed.

[https://github.com/donmelton/other_video_transcoding](https://github.com/donmelton/other_video_transcoding)

This is basically an easier-to-understand wrapper for ffmpeg.
Install ruby, then install this with ruby gems.
It's helpful to read the script to get a better understanding of all the options and how they map to ffmpeg params.

It requires **ffmpeg** and the [mkvtoolnix-cli]([https://archlinux.org/packages/extra/x86_64/mkvtoolnix-cli/](https://archlinux.org/packages/extra/x86_64/mkvtoolnix-cli/)) packages.

Example with libx265 (software/CPU transcode):

```fish
other-transcode \
  --target 12000 \  # there are defaults for each resolution, but imo they're too low
  --x265 \  # change this to VAAPI for gpu hw transcode
  --hevc \  # no point in not doing HEVC 
  --mp4 \   # imo it's widely enough compatible.
  great for appleTV
  --add-audio all \  # without this, the default will strip out all the extraneous audio
  --add-subtitle eng \
  </path/to/file.mkv> \
  -n # "dry run" (just prints the ffmpeg commmand)mi
```

Example with iGPU (hardware accelerated transcode):

```fish
other-transcode \
  --vaapi \  # this is a reliable api for linux + intel w/ iGPU (QuickSync)
  --target 12000 \  # VAAPI doesn't have an easy equivalent of CRF ; this should be tweaked per title
  --hevc \
  --add-audio all \
  --add-subtitle eng \
  </path/to/file.mkv>  # note: omitted --mp4 here because of subtitle incompatability
```

[Feb 19, 2022] - there is currently a bug in `intel-media-driver` that results in corrputed video output when using VAAPI (intel iGPU hw accelerated transcode).

[Feb 20, 2022] - went down the rabbit hole of manually building libva, intel-media-driver (and all intermediate dependencies, yikes!) and ffmpeg with [this patch](https://github.com/intel-media-ci/cartwheel-ffmpeg/blob/master/patches/0025-lavc-vaapi_encode_h265-fix-max_transform_hierarchy_d.patch).


example `ffmpeg` command that is generated:

```fish
ffmpeg -loglevel error -stats -vaapi_device /dev/dri/renderD128 \
  -i /media/movies/Honeyland\ \(2019\)/Honeyland\ \(2019\)\ Remux-1080p.mkv \
  -map 0:0 -filter:v yadif,format\=nv12,hwupload -c:v hevc_vaapi -b:v 9000k \
  -color_primaries:v bt709 -color_trc:v bt709 -colorspace:v bt709 \
  -metadata:s:v title\= -disposition:v default -map 0:1 -c:​a:0 ac3 \
  -metadata:s:​a:0 title\= -disposition:​a:0 default -map 0:2 -c:​a:1 aac \
  -metadata:s:​a:1 title\= -disposition:​a:1 0 -sn \
  -metadata:g title\= -default_mode passthrough \
  Honeyland\ \(2019\)\ Remux-1080p.mkv
```

general ffmpeg param structure:

```fish
ffmpeg -i <input file> \
	-stats \
	-<extra params> \
	<output_file_name>
```

(likely rare), but remux + transcode BR disk .iso rip:

[https://unixsheikh.com/tutorials/remuxing-iso-dvd-or-bluray-using-cat-and-ffmpeg-on-linux.html](https://unixsheikh.com/tutorials/remuxing-iso-dvd-or-bluray-using-cat-and-ffmpeg-on-linux.html)

essentially, the process is 1) mount the iso, find the correct m2ts file, combine them if necessary, then ffmpeg to package it up into .mkv and transcode if desired.
Here’s an example of processing Apollo 11 blu-ray full disk:

```fish
  ffmpeg -i br/BDMV/STREAM/00006.m2ts -stats -map 0:0 -pix_fmt yuv420p10le -c:v libx265 -x265-params profile=main10:crf=20 -tag:v hvc1 -map 0:​a 'Apollo 11 (2019) Remux-2160p.mp4’
````

using CRF for 4k main10:

```fish
nohup ffmpeg -loglevel error -stats -i ../Jojo\ Rabbit\ \(2019\)\ Remux-2160p.mkv \
  -map 0:0 \
  -c:v libx265 -pix_fmt:v yuv420p10le -crf 17 -preset slower \
  -color_primaries:v bt2020 -color_trc:v smpte2084 -colorspace:v bt2020nc \
  -metadata:s:v title\= -disposition:v default \
  -map 0:1 -c:​a:0 ac3 -metadata:s:​a:0 title\= -disposition:​a:0 default \
  -sn -metadata:g title\= -default_mode passthrough \
  Jojo\ Rabbit\ \(2019\)\ Remux-2160p.mkv &
```

{{< lead >}}
Quality checking the transcoded output is essential.
{{< /lead  >}}


## further reading

[https://codecalamity.com/encoding-settings-for-hdr-4k-videos-using-10-bit-x265/](https://codecalamity.com/encoding-settings-for-hdr-4k-videos-using-10-bit-x265/) - tl;dr CRF 20 preset slow

> [Apple TV 4k hevc](https://aaron.cc/ffmpeg-hevc-apple-devices/) - Use the `-tag:v hvc1` parameter in your FFmpeg command make your HEVC file work on Apple devices.

[https://codecalamity.com/stop-re-encoding-videos/](https://codecalamity.com/stop-re-encoding-videos/)

[https://slhck.info/video/2017/02/24/crf-guide.html](https://slhck.info/video/2017/02/24/crf-guide.html)

[https://netflixtechblog.com/per-title-encode-optimization-7e99442b62a2](https://netflixtechblog.com/per-title-encode-optimization-7e99442b62a2) - very interesting.
Big takeaways are:

1. every netflix title is considered to have its own optimal bitrate + quality level for each resolution
2. Lower resolution + higher precision can result in greater quality than higher res + lower precision, but mostly at the low side of the quality curve.
3. netflix stores multiple encodes per resolution, per title. Storage tradeoff gives bandwidth + user experience wins

[Using VAAPI's hardware accelerated video encoding on Linux with Intel's hardware on FFmpeg and libav](https://gist.github.com/Brainiarc7/95c9338a737aa36d9bb2931bed379219)

## still TODO / open questions

- test performance / effect of performing conversion over NFS
    - for the ryzen + Nvidia system, the source file is first copied to the local machine
    - when a 3rd system is used for transcode directly via NFS, CPU on the NFS server rises to about 25% usage!
- audio codecs
    - probably can find a good preset for “basic 5.1”