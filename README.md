# FFmpeg Compilation Tutorial with NVIDIA GPU Support

![FFmpeg](https://img.shields.io/badge/FFmpeg-Latest-green.svg)
![NVIDIA](https://img.shields.io/badge/NVIDIA-CUDA%20Enabled-76B900.svg)
![Ubuntu](https://img.shields.io/badge/Ubuntu-20.04%20LTS-E95420.svg)
![License](https://img.shields.io/badge/License-GPL--3.0-blue.svg)

A comprehensive guide to compiling FFmpeg with NVIDIA GPU acceleration (NVENC, NVDEC, CUDA, and NPP filters) on Ubuntu.

## 📋 Table of Contents

- [System Requirements](#-system-requirements)
- [Overview](#-overview)
- [Prerequisites](#-prerequisites)
 - [NVIDIA GPU Compatibility](#1-nvidia-gpu-compatibility)
 - [NVIDIA Driver Installation](#2-nvidia-driver-installation)
 - [CUDA Toolkit Installation](#3-cuda-toolkit-installation)
 - [NVIDIA Codec Headers](#4-nvidia-codec-headers)
 - [Development Packages](#5-development-packages)
- [FFmpeg Compilation](#-ffmpeg-compilation)
- [Verification](#-verification)
- [Troubleshooting](#-troubleshooting)
- [Contributing](#-contributing)
- [License](#-license)

## 🖥️ System Requirements

| Component | Specification |
|-----------|---------------|
| **Hardware** | HP Z440 |
| **CPU** | Intel Xeon 2680v4 |
| **RAM** | 16GB DDR4 |
| **GPU** | NVIDIA Quadro P620 |
| **OS** | Ubuntu 20 Server |

## 🎯 Overview

This tutorial covers compiling the latest FFmpeg version with NVIDIA GPU features including:

- ✅ **NVENC** - Hardware-accelerated encoding
- ✅ **NVDEC** - Hardware-accelerated decoding  
- ✅ **CUDA** - GPU compute acceleration
- ✅ **NPP** - NVIDIA Performance Primitives filters

Following this guide will help you understand the compilation process and dependencies required for optimal performance.

## 📦 Prerequisites

### 1. NVIDIA GPU Compatibility

> ⚠️ **Important**: Ensure your GPU supports NVIDIA encoder and decoder (NVENC, NVDEC)

Check compatibility at the [NVIDIA Video Encode/Decode GPU Support Matrix](https://developer.nvidia.com/video-encode-and-decode-gpu-support-matrix-new).

**Check connected GPUs:**
```bash
sudo lshw -C display
2. NVIDIA Driver Installation
2.1 Install Required Libraries
bashapt-get install systemd build-essential -y
sudo apt-get install build-essential yasm cmake libtool libc6 libc6-dev unzip wget libnuma1 libnuma-dev -y
2.2 Remove Old NVIDIA Drivers
bashapt-get purge nvidia-* -y
apt-get autoremove -y
2.3 Install DKMS Headers
bashsudo apt install build-essential dkms linux-headers-$(uname -r) -y
2.4 Disable Nouveau Driver
bashecho "blacklist nouveau" > /etc/modprobe.d/nvidia-installer-disable-nouveau.conf
echo "options nouveau modeset=0" >> /etc/modprobe.d/nvidia-installer-disable-nouveau.conf
update-initramfs -u
Reboot the system:
bashreboot
2.5 Install NVIDIA Driver
Using version 550.90.07 as example:
bashservice lightdm stop
nvidia-uninstall --silent
sudo apt install build-essential dkms linux-headers-$(uname -r) -y

# Download driver from NVIDIA
wget https://us.download.nvidia.com/XFree86/Linux-x86_64/550.90.07/NVIDIA-Linux-x86_64-550.90.07.run
chmod +x NVIDIA-Linux-x86_64-550.90.07.run
sudo bash NVIDIA-Linux-x86_64-550.90.07.run --silent --install-libglvnd --dkms
2.6 Verify Installation
bashnvidia-smi
Expected output should show GPU information and CUDA version (e.g., CUDA 12.4).
3. CUDA Toolkit Installation
Based on nvidia-smi output, install matching CUDA toolkit version:
3.1 Add NVIDIA Repository
bashwget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-keyring_1.0-1_all.deb
sudo dpkg -i cuda-keyring_1.0-1_all.deb
sudo apt-get update
3.2 Install CUDA Toolkit
bashsudo apt-get install cuda-toolkit-12-4
3.3 Update Environment Variables
bashecho 'export PATH=/usr/local/cuda/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
source ~/.bashrc
3.4 Verify CUDA Installation
bashnvcc --version
4. NVIDIA Codec Headers
4.1 Clone Repository
bashgit clone https://git.videolan.org/git/ffmpeg/nv-codec-headers.git
cd nv-codec-headers
4.2 Checkout Compatible Version
bash# For driver 550.x, use version n12.2.72.0
git checkout n12.2.72.0

📝 Note: Check the README file for minimum driver version requirements:

Linux: 550.54.14 or newer
Windows: 551.76 or newer


4.3 Install Headers
bashmake install
cd ..
5. Development Packages
5.1 Install Core Development Packages
bashsudo apt-get install libfreetype6-dev libfribidi-dev libxml2-dev \
    liblzma-dev libfontconfig1-dev libharfbuzz-dev libvorbis-dev \
    libpulse-dev libxcb1-dev libxcb-shape0-dev libxcb-xfixes0-dev \
    libaribb24-dev libgme-dev libass-dev \
    libbluray-dev libmp3lame-dev libopus-dev libssh-dev \
    libtheora-dev libvpx-dev libwebp-dev libzmq3-dev libopenal-dev \
    libopencore-amrnb-dev libopencore-amrwb-dev \
    libopenmpt-dev librubberband-dev libsdl2-dev libsoxr-dev \
    libtwolame-dev libdrm-dev libva-dev \
    libvidstab-dev libx264-dev libx265-dev libasound2-dev
5.2 Handle Missing Packages
Some packages might have different names. Search for alternatives:
bash# Search for correct package names
apt-cache search chromaprint | grep dev
apt-cache search openjpeg | grep dev
apt-cache search srt | grep dev
Install packages with correct names:
bashsudo apt-get install libchromaprint-dev libopenjp2-7-dev libsrt-dev
🔧 FFmpeg Compilation
1. Download FFmpeg Source
bashgit clone https://git.ffmpeg.org/ffmpeg.git ffmpeg
cd ffmpeg
2. Configure Build

📍 Installation Path: This configuration installs FFmpeg to /opt/ffmpeg

bash# Clean any previous build
make clean

# Configure with full feature set
./configure --prefix=/opt/ffmpeg \
    --enable-gpl --enable-version3 \
    --disable-debug --enable-iconv --enable-zlib \
    --enable-libfreetype --enable-libfribidi \
    --enable-libxml2 --enable-lzma \
    --enable-fontconfig --enable-libharfbuzz --enable-libvorbis \
    --enable-opencl --enable-libpulse \
    --enable-libxcb --enable-xlib \
    --enable-libaribb24 --enable-chromaprint \
    --disable-libfdk-aac \
    --enable-ffnvcodec --enable-cuda --enable-cuda-nvcc \
    --enable-cuvid --enable-nvenc --enable-libnpp \
    --enable-libgme --enable-libass \
    --enable-libbluray --enable-libmp3lame --enable-libopus \
    --enable-libssh --enable-libtheora --enable-libvpx \
    --enable-libwebp --enable-libzmq \
    --enable-openal --enable-libopencore-amrnb \
    --enable-libopencore-amrwb --enable-libopenjpeg \
    --enable-libopenmpt --enable-librubberband \
    --disable-schannel --enable-sdl2 --enable-libsoxr \
    --enable-libsrt --enable-libtwolame \
    --enable-libdrm --enable-vaapi \
    --enable-libvidstab \
    --disable-libvvenc --enable-libx264 --enable-libx265 \
    --enable-alsa \
    --enable-nonfree \
    --enable-static --disable-shared \
    --pkg-config-flags="--static" \
    --extra-cflags="-I/usr/local/cuda/include -DLIBTWOLAME_STATIC" \
    --extra-ldflags="-L/usr/local/cuda/lib64 -pthread" \
    --extra-libs='-ldl -lgomp' \
    --extra-ldexeflags=-pie
3. Compile and Install
bash# Compile (duration depends on CPU cores)
make -j$(nproc)

# Install to /opt/ffmpeg
make install
✅ Verification
Test FFmpeg Installation
bash/opt/ffmpeg/bin/ffmpeg -version
Test NVIDIA Hardware Acceleration
bash# List available NVIDIA encoders
/opt/ffmpeg/bin/ffmpeg -encoders | grep nvenc

# List available NVIDIA decoders  
/opt/ffmpeg/bin/ffmpeg -decoders | grep cuvid
Sample Encoding Commands
bash# Hardware-accelerated H.264 encoding
/opt/ffmpeg/bin/ffmpeg -i input.mp4 -c:v h264_nvenc -preset fast -b:v 5M output.mp4

# Hardware-accelerated HEVC encoding
/opt/ffmpeg/bin/ffmpeg -i input.mp4 -c:v hevc_nvenc -preset fast -b:v 3M output_hevc.mp4
🎉 Success!
If compilation completes successfully, you'll find the FFmpeg binaries in /opt/ffmpeg/bin/:

ffmpeg - Main conversion tool
ffprobe - Media analysis tool
ffplay - Simple media player

🔧 Troubleshooting
Common Issues and Solutions
IssueSolutionGPU not detectedVerify GPU compatibility and driver installationCUDA not foundCheck CUDA installation and PATH variablesMissing packagesUse apt-cache search to find correct package namesConfigure failsCheck all dependencies are installedCompilation errorsEnsure sufficient disk space and memory
Debug Commands
bash# Check GPU status
nvidia-smi

# Verify CUDA
nvcc --version

# Check library paths
ldconfig -p | grep cuda

# Verify codec headers
pkg-config --list-all | grep ffnvcodec
Performance Tips

Use -j$(nproc) for faster compilation
Ensure adequate cooling during compilation
Consider using tmpfs for build directory if you have sufficient RAM

🚀 Advanced Configuration
Custom Installation Path
To install to a different location, change the --prefix option:
bash./configure --prefix=/usr/local/ffmpeg \
    # ... rest of configuration
Minimal Configuration
For a lighter build with only NVIDIA features:
bash./configure --prefix=/opt/ffmpeg \
    --enable-gpl --enable-nonfree \
    --enable-cuda --enable-nvenc --enable-cuvid \
    --enable-libnpp --enable-ffnvcodec \
    --extra-cflags="-I/usr/local/cuda/include" \
    --extra-ldflags="-L/usr/local/cuda/lib64"
📚 Additional Resources

FFmpeg Official Documentation
NVIDIA Video Codec SDK
CUDA Toolkit Documentation
Hardware Acceleration Guide

🤝 Contributing
Contributions are welcome! Please feel free to submit issues, fork the repository, and create pull requests for any improvements.
How to Contribute

Fork the project
Create your feature branch (git checkout -b feature/AmazingFeature)
Commit your changes (git commit -m 'Add some AmazingFeature')
Push to the branch (git push origin feature/AmazingFeature)
Open a Pull Request

📝 Notes

⏱️ Compilation time varies based on CPU cores and selected features
💾 Ensure sufficient disk space (minimum 5GB free)
🔄 Keep driver and CUDA versions compatible
📦 Remove unwanted features from configure to reduce build time
🐧 Instructions tested on Ubuntu 20.04 LTS

📄 License
This project is licensed under the GPL-3.0 License - see the LICENSE file for details.

<div align="center">
⭐ Star this repository if it helped you!
Made with ❤️ for the FFmpeg community
</div>
```