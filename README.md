This tutorial to explain every step in FFMPEG Compile and how to run the compiled files on another setup
----------------

Here is my setup:

HP Z440
intel xeon 2680v4
16 GB DDR4 RAM
NVIDIA Quadro P620
OS: ubuntu 20 server
----------------

target from this file is to know how to compile ffmpeg latest version with NVIDIA GPU things like [nvenc, nvdec, cuda and npp filters]
and it will work anytime if you managed to understand what it needs
----------------

1- First of all we need a Nvidia GPU (surprise 😂) that supports Nvidia encoder and decoder (NVENC , NVDEC) in my case i have P620 2GB
you can check if you GPU support that from NVIDIA Matrix : https://developer.nvidia.com/video-encode-and-decode-gpu-support-matrix-new

2- then you need to install driver of NVIDIA GPU
    there is too many ways to do that GUI(if ubuntu desktop), or with cli
    i will use very manual way and we will use Driver 550.90.07 (not the latest to show how to deal with all versions)

    check the connected GPU`s on the setup :
        sudo lshw -C display
    install some liberaries:
        apt-get install systemd build-essential -y
        sudo apt-get install build-essential yasm cmake libtool libc6 libc6-dev unzip wget libnuma1 libnuma-dev -y

    remove old nvidia if any:
        apt-get purge nvidia-* -y
        apt-get autoremove -y

    install dkms headers to prevent sudden uninstall nvidia drivers:
        sudo apt install build-essential dkms linux-headers-$(uname -r) -y
    
    Disable the Nvidia nouveau open source driver:
        echo "blacklist nouveau" > /etc/modprobe.d/nvidia-installer-disable-nouveau.conf
        echo "options nouveau modeset=0" >> /etc/modprobe.d/nvidia-installer-disable-nouveau.conf
        update-initramfs -u

        reboot

    Install the Nvidia driver:
        service lightdm stop
        nvidia-uninstall --silent
        sudo apt install build-essential dkms linux-headers-$(uname -r) -y

    Download Driver you want from https://www.nvidia.com/en-us/drivers/
    in our case we will use 550.90.07 :
        wget https://us.download.nvidia.com/XFree86/Linux-x86_64/550.90.07/NVIDIA-Linux-x86_64-550.90.07.run
        chmod +x NVIDIA-Linux-x86_64-550.90.07.run
        sudo bash NVIDIA-Linux-x86_64-550.90.07.run --silent --install-libglvnd --dkms
    

Now check if the Driver Installed :
    nvidia-smi
    as you can see in nvidia-smi that the cuda version supported is 12.4
    so we will download and install cuda-toolkit-12-4

Download and install cuda-toolkit-12-4
    Add NVIDIA package repository
        wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-keyring_1.0-1_all.deb
        sudo dpkg -i cuda-keyring_1.0-1_all.deb
        sudo apt-get update

    Install CUDA toolkit (version 12.4)
        sudo apt-get install cuda-toolkit-12-4
    
    Add CUDA to your PATH:
        echo 'export PATH=/usr/local/cuda/bin:$PATH' >> ~/.bashrc
        echo 'export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
        source ~/.bashrc
    
    Check CUDA Installed :
        nvcc --version

Download Compatible nv-codec-headers Version:
    Clone and checkout a compatible version
        git clone https://git.videolan.org/git/ffmpeg/nv-codec-headers.git
        cd nv-codec-headers

    Checkout a version compatible with driver 550.x
    In README file of the nv-codec-headers you will find supported Driver Verion:
        git checkout n12.2.72.0
    after download Check the README file in nv-codec-headers dir
    in our case you will find:
            "Minimum required driver versions:
            Linux: 550.54.14 or newer
            Windows: 551.76 or newer"

        make install
        cd ..

Install some development packages :
    sudo apt-get install libfreetype6-dev libfribidi-dev libxml2-dev \
        liblzma-dev libfontconfig1-dev libharfbuzz-dev libvorbis-dev \
        libpulse-dev libxcb1-dev libxcb-shape0-dev libxcb-xfixes0-dev \
        libaribb24-dev chromaprint-tools libgme-dev libass-dev \
        libbluray-dev libmp3lame-dev libopus-dev libssh-dev \
        libtheora-dev libvpx-dev libwebp-dev libzmq3-dev libopenal-dev \
        libopencore-amrnb-dev libopencore-amrwb-dev libopenjpeg2-dev \
        libopenmpt-dev librubberband-dev libsdl2-dev libsoxr-dev \
        libsrt-openssl-dev libtwolame-dev libdrm-dev libva-dev \
        libvidstab-dev libx264-dev libx265-dev libasound2-dev
    
    mostly you will get some errors like :
        E: Unable to locate package chromaprint-tools
        E: Unable to locate package libopenjpeg2-dev
        E: Unable to locate package libsrt-openssl-dev
    
    So Install Available Packages First:
        sudo apt-get install libfreetype6-dev libfribidi-dev libxml2-dev \
            liblzma-dev libfontconfig1-dev libharfbuzz-dev libvorbis-dev \
            libpulse-dev libxcb1-dev libxcb-shape0-dev libxcb-xfixes0-dev \
            libaribb24-dev libgme-dev libass-dev \
            libbluray-dev libmp3lame-dev libopus-dev libssh-dev \
            libtheora-dev libvpx-dev libwebp-dev libzmq3-dev libopenal-dev \
            libopencore-amrnb-dev libopencore-amrwb-dev \
            libopenmpt-dev librubberband-dev libsdl2-dev libsoxr-dev \
            libtwolame-dev libdrm-dev libva-dev \
            libvidstab-dev libx264-dev libx265-dev libasound2-dev

    Check What's Available if Still Missing: (you can you AI bot to help you in package names)
    # Search for alternative package names
        apt-cache search chromaprint | grep dev
        apt-cache search openjpeg | grep dev
        apt-cache search srt | grep dev
    
    Install the correctly named packages:
        sudo apt-get install libchromaprint-dev libopenjp2-7-dev libsrt-dev

Now Download FFMPEG:
git clone https://git.ffmpeg.org/ffmpeg.git ffmpeg

Now Run the Complete Configure Command:
    ** Notice this Configuration will make the output to /opt/ffmpeg **
    ** In my case i want to Configure ffmpeg with all of these features Like [ALSA, Cuda, npp, srt, etc...] , you can use it or delete what you don`t want **

    cd ffmpeg

    # Clean any previous build
    make clean

    # Configure with all features
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

If Configure Succeeds, Compile :
    # Compile (this will take sometime depends on CPU)
    make -j$(nproc)

    # Install
    make install

if all good , Congrates 🎉❤ , you will find the files in /opt/ffmpeg

