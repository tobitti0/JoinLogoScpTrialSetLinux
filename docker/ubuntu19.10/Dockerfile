FROM ubuntu:19.10

ARG NASM_VER="2.14"
ARG YASM_VER="1.3.0"
ARG LAME_VER="3.100"

RUN set -xe && \
    apt-get update && \
    apt-get -y install software-properties-common && \
    add-apt-repository ppa:ubuntu-toolchain-r/test && \
    apt-get update && \
    apt-get install -y aptitude && \
    aptitude update && \
    aptitude install -y \
        wget build-essential automake autoconf git libtool libvorbis-dev \
        libass-dev libfreetype6-dev libsdl2-dev libva-dev libvdpau-dev \
        libxcb1-dev libxcb-shm0-dev libxcb-xfixes0-dev \
        mercurial libnuma-dev texinfo zlib1g-dev \
        cmake qtbase5-dev checkinstall gcc-9 g++-9 curl \
        python3-pip ninja-build \
    &&\
    pip3 install meson && \
    apt-get clean && \
    aptitude clean

# nasm
RUN set -xe && \
    cd /tmp && \
    wget -O /tmp/nasm.tar.bz2 \
    	 https://www.nasm.us/pub/nasm/releasebuilds/${NASM_VER}/nasm-${NASM_VER}.tar.bz2 && \
    mkdir -p /tmp/nasm && \
    tar jxvf /tmp/nasm.tar.bz2 \
        -C /tmp/nasm --strip-components 1 && \
    cd /tmp/nasm && \
    ./autogen.sh && \
    ./configure --prefix=/usr/local && \
    make -j$(nproc) && \
    make install && \
    ldconfig && \
    rm -r /tmp/*

# yasm
RUN set -xe && \
    cd /tmp && \
    wget -O /tmp/yasm.tar.gz \
    	 https://www.tortall.net/projects/yasm/releases/yasm-${YASM_VER}.tar.gz && \
    mkdir -p /tmp/yasm && \
    tar xzvf /tmp/yasm.tar.gz \
        -C /tmp/yasm --strip-components 1 && \
    cd /tmp/yasm && \
    ./configure \
      --prefix=/usr/local && \
    make -j$(nproc) && \
    make install && \
    ldconfig && \
    rm -r /tmp/*

# l-smash
RUN set -xe && \
    cd /tmp && \
    git clone https://github.com/l-smash/l-smash.git && \
    cd l-smash && \
    ./configure \
      --enable-shared \
      && \
    make && \
    make install && \
    ldconfig && \
    rm -r /tmp/*

# libx264
RUN set -xe && \
    cd /tmp && \
    git clone --depth 1 https://code.videolan.org/videolan/x264.git && \
    cd /tmp/x264 && \
    ./configure \
      --prefix=/usr/local \
      --enable-shared \
      --enable-pic \
    && \
    make && \
    make install && \
    ldconfig && \
    rm -r /tmp/*

# libvpx
RUN set -xe && \
    cd /tmp && \
    git clone --depth 1 https://chromium.googlesource.com/webm/libvpx.git && \
    cd /tmp/libvpx && \
    ./configure \
      --disable-examples \
      --disable-unit-tests \
      --enable-vp9-highbitdepth \
      --as=yasm \
    && \
    make -j$(nproc) && \
    make install && \
    ldconfig && \
    rm -r /tmp/*

# libfdk-aac
RUN set -xe && \
    cd /tmp && \
    git clone --depth 1 https://github.com/mstorsjo/fdk-aac && \
    cd fdk-aac && \
    autoreconf -fiv && \
    ./configure \
    && \
    make -j$(nproc) && \
    make install && \
    ldconfig && \
    rm -r /tmp/*

# libmp3lame
RUN set -xe && \
    cd /tmp && \
    wget -O /tmp/lame.tar.gz \
         https://downloads.sourceforge.net/project/lame/lame/${LAME_VER}/lame-${LAME_VER}.tar.gz && \
    mkdir -p /tmp/lame && \
    tar xzvf /tmp/lame.tar.gz \
         -C /tmp/lame --strip-components 1 && \
    cd /tmp/lame && \
    ./configure \
      --enable-shared \
      --enable-nasm \
    && \
    make -j$(nproc) && \
    make install && \
    ldconfig && \
    rm -r /tmp/*

# libopus
RUN set -xe && \
    cd /tmp && \
    git clone --depth 1 https://github.com/xiph/opus && \
    cd opus && \
    ./autogen.sh && \
    ./configure \
      --enable-shared \
    && \
    make -j$(nproc) && \
    make install && \
    ldconfig && \
    rm -r /tmp/*

#AviSynth+
RUN set -xe && \
    cd /tmp && \
    git clone --depth 1 git://github.com/AviSynth/AviSynthPlus.git && \
    cd AviSynthPlus && \
    mkdir avisynth-build && \
    cd avisynth-build && \
    CC=gcc-9 CXX=gcc-9 LD=gcc-9 cmake ../ -G Ninja && \
    ninja && \
    checkinstall \
      --pkgname=avisynth \
      --pkgversion="$(grep -r \
      Version avs_core/avisynth.pc | cut -f2 -d " ")-$(date --rfc-3339=date | \
      sed 's/-//g')-git" \
      --backup=no \
      --deldoc=yes \
      --delspec=yes \
      --deldesc=yes \
      --strip=yes \
      --stripso=yes \
      --addso=yes \
      --fstrans=no \
      --default ninja install \
    && \
    rm -r /tmp/*

# FFmpeg
RUN set -xe && \
    sed -Ei 's/^# deb-src /deb-src /' /etc/apt/sources.list && \
    apt-get update && \
    apt-get -y build-dep ffmpeg && \
    cd /tmp &&\
    git clone --depth 1 git://git.ffmpeg.org/ffmpeg.git && \
    cd /tmp/ffmpeg && \
    ./configure \
      --enable-gpl\
      --enable-version3 \
      --disable-doc \
      --disable-debug \
      --enable-pic \
      --enable-avisynth \
      --enable-libx264 \
      --enable-libx265 \
      --enable-libfdk-aac \
      --enable-libfreetype \
      --enable-libmp3lame \
      --enable-libopus \
      --enable-libvorbis \
      --enable-libvpx \
      --enable-nonfree \
      && \
    make CC=gcc-9 CXX=gcc-9 LD=gcc-9 -j$(nproc) && \
    checkinstall \
      --pkgname=ffmpeg \
      --pkgversion="7:$(git rev-list \
        --count HEAD)-g$(git rev-parse --short HEAD)" \
    --backup=no \
    --deldoc=yes \
    --delspec=yes \
    --deldesc=yes \
    --strip=yes \
    --stripso=yes \
    --addso=yes \
    --fstrans=no \
    --default \
    && \
    apt-get clean all && \
    rm -r /tmp/*

#FFMS2
RUN set -xe && \
    cd /tmp &&\
    git clone --depth 1 git://github.com/ffms/ffms2.git && \
    cd ffms2 && \
    PKG_CONFIG_PATH=$HOME/ffavx_build/lib/pkgconfig \
    CPPFLAGS="-I/usr/local/include/avisynth" \
    ./autogen.sh --enable-shared --enable-avisynth && \
    make -j$(nproc) && \
    checkinstall --pkgname=ffms2 --pkgversion="1:$(./version.sh)-git" \
    --backup=no --deldoc=yes --delspec=yes --deldesc=yes --strip=yes --stripso=yes \
    --addso=yes --fstrans=no --default && \
    ldconfig && \
    rm -r /tmp/*

# l-smash-source
RUN set -xe && \
    cd /tmp &&\
    git clone https://github.com/HolyWu/L-SMASH-Works.git && \
    cd /tmp/L-SMASH-Works/AviSynth && \
    git checkout 72d3eac802eebcfc9080009c1a8d47a747e3a306 &&\
    CC=gcc-9 CXX=gcc-9 LD=gcc-9 LDFLAGS="-Wl,-Bsymbolic" meson build && \
    cd build && \
    ninja -v && \
    ninja install && \
    ldconfig && \
    rm -r /tmp/*

# nodejs
RUN set -xe && \
    curl -sL https://deb.nodesource.com/setup_10.x | bash - && \
    apt-get update && \
    apt-get install -y nodejs && \
    node -v

# join_logo_scp_trial
ADD modules /tmp/modules
RUN set -xe && \
    cd /tmp/modules/chapter_exe/src && \
    make && \
    mv chapter_exe /tmp/modules/join_logo_scp_trial/bin/ && \
    cd /tmp/modules/logoframe/src && \
    make && \
    mv logoframe /tmp/modules/join_logo_scp_trial/bin/ && \
    cd /tmp/modules/join_logo_scp/src && \
    make && \
    mv join_logo_scp /tmp//modules/join_logo_scp_trial/bin/ && \    
    mv /tmp/modules/join_logo_scp_trial /join_logo_scp_trial && \
    cd /join_logo_scp_trial && \
    npm install && \
    rm -r /tmp/*

WORKDIR /join_logo_scp_trial
ENTRYPOINT ["npm", "start", "--" ,"-i"]
CMD ["--help"]
