---
created: 2024-03-13T16:30
updated: 2024-03-13T16:33
tags:
  - Debian
  - Linux
  - HardwareVideoAcceleration
related link: https://wiki.debian.org/HardwareVideoAcceleration
---
With modern graphics cards, it's often possible to [offload](https://en.wikipedia.org/wiki/Video_processing_unit#GPU_accelerated_video_decoding) the jobs of video encoding and decoding to them from the CPU in order to reduce power usage and make more resources available to the rest of the system. Compared to CPUs, GPUs are much more efficient at the job. However, both hardware as well as software support are required for this offloading, and the latter in particular (at least in the Linux world) has undergone much evolution in recent years. Online documentation is therefore sparse and incomplete, inconsistent, and often outdated. Furthermore, support for hardware video acceleration under Linux world is unfortunately split across different APIs with different levels of support.

## Benefits

Historically, the benefits of hardware acceleration under Linux have been [uncertain](https://github.com/mpv-player/mpv/commit/dbef5b737e2f994f02923c8214cba368b663a655), but it seems likely that support today has improved drastically. In at least some relatively typical scenarios, the performance gains of using hardware decoding can be huge, [with reductions in CPU usage of around 90%](https://lists.debian.org/debian-user/2018/06/msg00306.html).

## APIs and Hardware / Software Support

The three main APIs that are in use are VA-API, VDPAU, and NVENC/NVDEC.

-   VA-API - Supported on Intel, AMD, and NVIDIA (only via the open-source Nouveau drivers). Widely supported by software, including Kodi, VLC, MPV, Chromium, and Firefox. Main limitation is lacking any support in the [proprietary NVIDIA drivers.](https://wiki.debian.org/NvidiaGraphicsDrivers)
-   VDPAU - Supported fully on AMD and NVIDIA (both proprietary and Nouveau). Supported by most desktop applications like Kodi, VLC, and MPV, but has no support at all in Chromium or Firefox. Main limitations are poor and incomplete Intel support and not working with browsers for web video acceleration.
-   NVENC/NVDEC - A proprietary API supported exclusively by NVIDIA. Only supported in a few major applications (FFmpeg and OBS Studio for encoding, FFmpeg and MPV for decoding). Main limitation is limited software and hardware support across the board because of its proprietary nature.

---
## Installation

### VA-API

VA-API sees broad software support and is even used by default in applications like MPV when it's available.

For Nouveau and the various AMD drivers, support can be added simply by installing the [mesa-va-drivers](https://packages.debian.org/mesa-va-drivers "DebianPkg") package.

For Intel, it's split generationally, and into free and non-free drivers. The non-free drivers **are necessary to encode media** while the free drivers can only decode.

For Gen 8+ Intel hardware, the free driver can be installed with the [intel-media-va-driver](https://packages.debian.org/intel-media-va-driver "DebianPkg") package. You can find the non-free driver in the [intel-media-va-driver-non-free](https://packages.debian.org/intel-media-va-driver-non-free "DebianPkg") package after adding a non-free component to your [apt sources](https://wiki.debian.org/SourcesList).

For older Intel hardware, the free driver can be installed with the [i965-va-driver](https://packages.debian.org/i965-va-driver "DebianPkg") package. The non-free driver can be installed with the [i965-va-driver-shaders](https://packages.debian.org/i965-va-driver-shaders "DebianPkg") package after adding a non-free component to your [apt sources](https://wiki.debian.org/SourcesList). This driver supports up to Gen 9 GPUs. If both drivers are installed, the newer driver from [intel-media-va-driver](https://packages.debian.org/intel-media-va-driver "DebianPkg") is preferred over [i965-va-driver](https://packages.debian.org/i965-va-driver "DebianPkg") (since Debian 11/Bullseye).

Driver selection can be overridden by setting the environment variable LIBVA_DRIVER_NAME to a specific driver, e.g., LIBVA_DRIVER_NAME=i965 (to use the driver from [i965-va-driver](https://packages.debian.org/i965-va-driver "DebianPkg") on Bullseye) or LIBVA_DRIVER_NAME=iHD (to use the driver from [intel-media-va-driver](https://packages.debian.org/intel-media-va-driver "DebianPkg") on Debian 10/Buster). See [EnvironmentVariables](https://wiki.debian.org/EnvironmentVariables) for more details on how to set this environment variable system-wide or per user.

### VDPAU

VDPAU faces considerably more limitations compared to VA-API, but nonetheless, it's the only option for some users. Particularly, anyone using the NVIDIA proprietary drivers. It's not supported in any major browser except for GNOME Web but is useful for local playback. MPV is recommended for this.

To enable VDPAU support for the AMD drivers (radeon and amdgpu), along with the open-source Nouveau driver for NVIDIA cards, install the [vdpau-driver-all](https://packages.debian.org/vdpau-driver-all "DebianPkg") package.

This will also enable VDPAU support over the OpenGL/VA-API backend for Intel GPUs. However, this has severe stability issues and may not work at all on some Intel devices. If possible, you are heavily recommended to use VA-API instead with Intel.

To enable VDPAU support for the proprietary NVIDIA drivers, you must choose the relevant package for your driver version. If you installed the latest drivers via the [nvidia-driver](https://packages.debian.org/nvidia-driver "DebianPkg") package, then you can simply install the [nvidia-vdpau-driver](https://packages.debian.org/nvidia-vdpau-driver "DebianPkg") package.

Otherwise, if you're using a legacy driver, choose the appropriate package from the following: [nvidia-legacy-304xx-vdpau-driver](https://packages.debian.org/nvidia-legacy-304xx-vdpau-driver "DebianPkg"), [nvidia-legacy-340xx-vdpau-driver](https://packages.debian.org/nvidia-legacy-340xx-vdpau-driver "DebianPkg"), [nvidia-legacy-390xx-vdpau-driver](https://packages.debian.org/nvidia-legacy-390xx-vdpau-driver "DebianPkg")

### NVENC/NVDEC

NVDEC is supported by the [libnvcuvid1](https://packages.debian.org/libnvcuvid1 "DebianPkg") package.

For the legacy drivers, instead choose the [libnvidia-legacy-304xx-nvcuvid1](https://packages.debian.org/libnvidia-legacy-304xx-nvcuvid1 "DebianPkg"), [libnvidia-legacy-340xx-nvcuvid1](https://packages.debian.org/libnvidia-legacy-340xx-nvcuvid1 "DebianPkg"), or [libnvidia-legacy-390xx-nvcuvid1](https://packages.debian.org/libnvidia-legacy-390xx-nvcuvid1 "DebianPkg") package.

NVENC is supported by the [libnvidia-encode1](https://packages.debian.org/libnvidia-encode1 "DebianPkg") package.

For the legacy drivers, instead choose either the [libnvidia-legacy-340xx-encode1](https://packages.debian.org/libnvidia-legacy-340xx-encode1 "DebianPkg") or [libnvidia-legacy-390xx-encode1](https://packages.debian.org/libnvidia-legacy-390xx-encode1 "DebianPkg") package.

These are only non-free runtime libraries, however. Applications in Debian's main archive including FFmpeg (starting with [libavcodec58](https://packages.debian.org/libavcodec58 "DebianPkg") 7:4.4.1-2) and users of FFmpeg's libraries will load the libraries from the non-free driver if they are available.

---

## Checking hardware support

You can find a full report of whether or not VA-API or VDPAU are functional, and what codecs they support, by installing the [vainfo](https://packages.debian.org/vainfo "DebianPkg") and [vdpauinfo](https://packages.debian.org/vdpauinfo "DebianPkg") packages and running their commands. vainfo and vdpauinfo

### Intel

[vainfo](https://packages.debian.org/vainfo "DebianPkg") will display codecs supported for decoding and encoding.

VAEntrypointVLD = decode
VAEntrypointEnc* = encode

OR

Wikipedia has an easy to read chart for intel GPU support: [https://en.wikipedia.org/wiki/Intel_Quick_Sync_Video#Hardware_decoding_and_encoding](https://en.wikipedia.org/wiki/Intel_Quick_Sync_Video#Hardware_decoding_and_encoding)

Intel GPU hardware acceleration can be confirmed via intel_gpu_top found in [intel-gpu-tools](https://packages.debian.org/intel-gpu-tools "DebianPkg"). Anything greater than 0% in the "ENGINE Video" section confirms it is working in whatever application and codec you are testing e.g:

# intel_gpu_top
intel-gpu-top -  173/ 174 MHz;   71% RC6;  0.16 Watts;      403 irqs/s
      IMC reads:     1020 MiB/s
     IMC writes:      226 MiB/s
          ENGINE      BUSY                             MI_SEMA MI_WAIT
     Render/3D/0    5.12% |█▎                        |      0%      0%
       Blitter/0    0.00% |                          |      0%      0%
         Video/0    6.47% |█▋                        |      0%      0%
  VideoEnhance/0    0.00% |                          |      0%      0%

Note: If using intel_gpu_top, test each application individually.

---

## Application Support

Application support for hardware acceleration varies, and each application requires individual configuration. Following are details for various applications.

### mpv

mpv has good hardware acceleration support, although it is not enabled by default. To enable it, use the --hwdec command line switch. It can also be made the default by adding a line like “hwdec” to the mpv configuration file (e.g., $HOME/.config/mpv/mpv.conf). [hwdec can also take various values, although ideally supplying the switch with no value should be sufficient. See the mpv manpage for more details (which recommends not to set the switch).]

If hardware acceleration is being used, mpv’s output will contain lines like the following:

libva info: VA-API version 0.39.4
libva info: va_getDriverName() returns 0
libva info: Trying to open /usr/lib/x86_64-linux-gnu/dri/i965_drv_video.so
libva info: Found init function __vaDriverInit_0_39
libva info: va_openDriver() returns 0
AO: [alsa] 48000Hz stereo 2ch float
Using hardware decoding (vaapi).
VO: [opengl] 1920x816 vaapi

### VLC

Hardware acceleration in VLC is controlled in the GUI via “Tools → Preferences → Input / Codecs → Hardware-accelerated decoding”, or via the CLI option –avcodec-hw value [‘value’ is mandatory].

If hardware acceleration is being used, VLC’s output will contain lines like the following:

libva info: VA-API version 0.39.4
libva info: va_getDriverName() returns 0
libva info: Trying to open /usr/lib/x86_64-linux-gnu/dri/i965_drv_video.so
libva info: Found init function __vaDriverInit_0_39
libva info: va_openDriver() returns 0
[00007f082ce97280] avcodec decoder: Using Intel i965 driver for Intel(R) Broadwell - 1.7.3 for hardware decoding

Please beaware that VLC 3.0.x in bookworm has limited VA-API support. Hardware accelerating using VDPAU is expected to work.

### Browser support

Web video is one of the most important use-cases for hardware video acceleration, as without it, sites like YouTube will cause heavy CPU usage (and therefore power consumption), a particular concern on mobile devices such as laptops and tablets.

As of [Debian 11/Bullseye](https://wiki.debian.org/DebianBullseye):

-   Chromium or Chrome - VA-API support is enabled by default in Chromium.
-   Firefox - starting at version 95, it is supported but must be manually enabled. See [Firefox](https://wiki.debian.org/Firefox) on how to enable. Firefox-ESR is projected to be updated to version 102 sometime in 2Q/3Q 2022.
-   [GNOME Web](https://en.wikipedia.org/wiki/GNOME%20Web "WikiPedia") has VA-API support through the [gstreamer1.0-vaapi](https://packages.debian.org/gstreamer1.0-vaapi "DebianPkg") package if installed.

VDPAU isn't supported at all in Chromium or Firefox. The only browser that supports it is [GNOME Web](https://en.wikipedia.org/wiki/GNOME%20Web "WikiPedia"), available in the [epiphany-browser](https://packages.debian.org/epiphany-browser "DebianPkg") package. GNOME Web leverages GStreamer for this support, also requiring the [gstreamer1.0-plugins-bad](https://packages.debian.org/gstreamer1.0-plugins-bad "DebianPkg") package for VDPAU support to work properly.