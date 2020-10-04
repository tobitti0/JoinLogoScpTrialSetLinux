# インストール
## 目次
- [avisynth+](#avisynth+)
- [l-smash](#l-smash)
- [FFmpeg](#FFmpeg)
- [l-smash-works](#l-smash-works)
- [join_logo_scp_trial](#join_logo_scp_trial)
- [delogo](#delogo)

## はじめに
join_logo_scp_trial_linuxは単体では動作しません。  
様々なものに依存し動作を行います。  
簡単に利用するにはDocker-composeを用いて同梱しているものを実行することを勧めます。
なお、書きかけですので不備があればプルリクや、Issueいただけると助かります。

## avisynth+
詳細は[公式手順](https://github.com/AviSynth/AviSynthPlus/blob/master/distrib/docs/english/source/avisynthdoc/contributing/posix.rst)を参照すること。
```
add-apt-repository ppa:ubuntu-toolchain-r/test
apt update
apt install build-essential cmake git ninja-build gcc-9 g++-9 checkinstall
git clone --depth 1 git://github.com/AviSynth/AviSynthPlus.git
cd AviSynthPlus
mkdir build
cd build
CC=gcc-9 CXX=gcc-9 LD=gcc-9 cmake ../ -G Ninja
ninja
checkinstall \
  --pkgname=avisynth \
  --pkgversion="$(grep -r Version avs_core/avisynth.pc | cut -f2 -d " ")-$(date --rfc-3339=date | sed 's/-//g')-git" \
  --backup=no \
  --deldoc=yes \
  --delspec=yes \
  --deldesc=yes \
  --strip=yes \
  --stripso=yes \
  --addso=yes \
  --fstrans=no \
  --default ninja install
```

## l-smash
l-smash
```
git clone --depth 1 https://github.com/l-smash/l-smash.git
cd l-smash
./configure --enable-shared
make
make install
ldconfig
```

## FFmpeg
FFmpeg
```
apt install nasm libfdk-aac-dev libx264-dev libx265-dev
git clone --depth 1 git://git.ffmpeg.org/ffmpeg.git
cd ffmpeg
./configure \
  --enable-gpl \
  --enable-version3 \
  --disable-doc \
  --disable-debug \
  --enable-avisynth \
  --enable-libfdk-aac \
  --enable-libx264 \
  --enable-libx265 \
  --enable-nonfree
make CC=gcc-9 CXX=gcc-9 LD=gcc-9
checkinstall \
  --pkgname=ffmpeg \
  --pkgversion="7:$(git rev-list --count HEAD)-g$(git rev-parse --short HEAD)" \
  --backup=no \
  --deldoc=yes \
  --delspec=yes \
  --deldesc=yes \
  --strip=yes \
  --stripso=yes \
  --addso=yes \
  --fstrans=no \
  --default
```

## l-smash-works
l-smash-works
```
apt install python3-pip
pip3 install meson
git clone --depth 1 https://github.com/HolyWu/L-SMASH-Works.git
cd L-SMASH-Works/AviSynth
CC=gcc-9 CXX=gcc-9 LD=gcc-9 LDFLAGS="-Wl,-Bsymbolic" meson build
cd build
ninja -v
ninja install
ldconfig
```

## join_logo_scp_trial
join_logo_scp_trial  
nodejs10以上じゃないと動かないと思います。
```
git clone --recursive https://github.com/tobitti0/JoinLogoScpTrialSetLinux.git
cd JoinLogoScpTrialSetLinux/modules/chapter_exe/src
make
mv chapter_exe ../../join_logo_scp_trial/bin
cd ../../logoframe/src
make
mv logoframe ../../join_logo_scp_trial/bin
cd ../../join_logo_scp/src
make
mv join_logo_scp ../../join_logo_scp_trial/bin
cp -r ../../join_logo_scp_trial /opt
cd /opt/join_logo_scp_trial
npm install
npm link
jlse --help
```
logoファイル(*.lgd)は`/opt/join_logo_scp_trial/logo`に入れる。

## delogo
delogo
```
git clone --recursive 1 https://github.com/tobitti0/delogo-AviSynthPlus-Linux.git
cd delogo-AviSynthPlus-Linux/src
make
make install
ldconfig
```


