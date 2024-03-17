---
created: 2024-03-13T16:33
updated: 2024-03-13T16:34
tags:
  - Debian
  - Intel
  - HardwareVideoAcceleration
related link: https://packages.debian.org/bullseye/intel-media-va-driver-non-free
---
"The VA-API (Video Acceleration API) enables hardware accelerated video decode/encode at various entry-points (VLD, IDCT, Motion Compensation etc.) for the prevailing coding standards today (MPEG-2, MPEG-4 ASP/H.263, MPEG-4 AVC/H.264, and VC-1/WMV3). It provides an interface to fully expose the video decode capabilities in today's GPUs."

Note : Permet l'utilisation de tout les formats d'encodage par rapport à la version free. J'assume que la version non-free est gratuite pour un usage personnel. 

## You should be able to use any of the listed mirrors by adding a line to your /etc/apt/sources.list like this:

deb http://_ftp.de.debian.org/debian_ bullseye main non-free

Replacing _ftp.de.debian.org/debian_ with the mirror in question.
