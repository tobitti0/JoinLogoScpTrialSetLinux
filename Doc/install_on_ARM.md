# ARM環境でのインストール
Dockerでの動作ができないため、ローカルにインストールを行います。join_logo_scp以外にも依存関係のあるパッケージやソフトウェアをビルドする必要があります。  
Raspberry Pi 4B + Raspbian 32bitと　Orange Pi3 + Armbian Bionicの環境でセットアップが可能なことを確認しています。
今回はRaspberry Pi + Raspbian 32bit を例に説明を行います。

# 必要なパッケージなどをインストール
```
sudo apt-get install build-essential cmake ninja-build libmp3lame-dev libopus-dev libvorbis-dev libvpx-dev libx265-dev libx264-dev libavcodec-dev libavformat-dev libswscale-dev libatomic-ops-dev automake libtool autoconf nodejs sudo libxft-dev
sudo apt-get install meson npm

```

# fdk-aac
ffmpegのビルドの際に必要となるのでこちらもビルドを行います。
```
git clone https://github.com/mstorsjo/fdk-aac.git
cd fdk-aac
./autogen.sh
./configure
make -j4
sudo make install 
sudo /sbin/ldconfig
```

# l-smash
```
git clone https://github.com/l-smash/l-smash.git
cd l-smash
./configure --enable-shared
make
sudo make install
sudo ldconfig
```

# AviSynthPlus
```
git clone --depth 1 git://github.com/AviSynth/AviSynthPlus.git
cd AviSynthPlus
mkdir avisynth-build
cd avisynth-build
cmake -DCMAKE_CXX_FLAGS=-latomic ../ -G Ninja
ninja
sudo ninja install
```

# ffmpeg
今回は32bitOSの手順で説明します。64bitOSだと./configure以降を少し変更する必要があります。
```
git clone --depth 1 https://github.com/FFmpeg/FFmpeg.git
cd FFmpeg
./configure --extra-ldflags="-latomic" --extra-cflags="-I/usr/local/include" --extra-ldflags="-L/usr/local/lib" --arch=armel --target-os=linux --enable-gpl --disable-doc --disable-debug --enable-pic --enable-avisynth --enable-libx264 --enable-libx265 --enable-libfdk-aac --enable-libfreetype --enable-libmp3lame --enable-libopus --enable-libvorbis --enable-libvpx --enable-nonfree --enable-mmal --enable-omx-rpi --enable-omx --extra-libs=-ldl
 make -j4
sudo make install
```

# L-SMASH-Works
説明の簡略化のため、ホームディレクトリでビルド作業を行います。

```
cd
git clone https://github.com/HolyWu/L-SMASH-Works.git
git clone https://github.com/tobitti0/chapter_exe.git -b arm-test
sudo cp chapter_exe/src/sse2neon.h L-SMASH-Works/AviSynth/emmintrin.h
cd L-SMASH-Works/AviSynth
```
ソースファイルの改変を行います。
`meson.build`の中身のsources=[]内の以下の記述を削除してください。
```
'../common/lwsimd.c',
'../common/lwsimd.h',
```
さらに70行目付近に、`if host_machine.cpu_family().startswith('x86')...~endif`という部分があると思うのでその次の行に以下の文を追記。
```
if host_machine.cpu_family().startswith('arm')
add_project_arguments('-mfpu=neon', language : ['c', 'cpp'])
endif
```
これで`meson.build`の編集は終了で、次に`video_output.cpp`を編集します。
`video_output.cpp`内部の以下の一文を削除してください。
```
#include "../common/lwsimd.h"
```
これで`video_output.cpp`の編集は終了です。<br><br>
ここからビルドなのですが、Raspberry Piではgccのバージョンが指定しなくても正常にインストールが可能です。しかしながらOrange Piを用いてセットアップした際にはgccのバージョンが足りていなかったため、エラーが出ました。そのため必要であればgccのバージョンを新しくするか、明示を行って対応して下さい。gcc9系をであれば間違いなく大丈夫です。
```
CC=gcc CXX=gcc LD=gcc LDFLAGS="-Wl,-Bsymbolic,-L/opt/vc/lib" meson build
cd build
ninja -v
sudo ninja install
sudo ldconfig
```
gccのバージョンを明示する場合は1行目のすべての`gcc`を`gcc-9`のような形で指定してください。
`meson`が古いと言われたら`meson`を更新してください。公式ドキュメント通りですが以下のコマンドで簡単に`meson`の更新を行えます。
```
sudo apt purge meson
sudo apt-get install pyhton3-pip python3-setuptools
python3 -m pip install meson
```

# join_logo_scp
こちらも説明を簡略化するためにホームディレクトリでビルド作業を行います。
```
cd
git clone https://github.com/tobitti0/chapter_exe.git -b arm-test
git clone --recursive https://github.com/tobitti0/JoinLogoScpTrialSetLinux.git
cd JoinLogoScpTrialSetLinux/modules/logoframe/src
make
cd && cp JoinLogoScpTrialSetLinux/modules/logoframe/src/logoframe JoinLogoScpTrialSetLinux/modules/join_logo_scp_trial/bin/logoframe
cd JoinLogoScpTrialSetLinux/modules/join_logo_scp/src
make
cd && cp JoinLogoScpTrialSetLinux/modules/join_logo_scp/src/join_logo_scp JoinLogoScpTrialSetLinux/modules/join_logo_scp_trial/bin/join_logo_scp
cd chapter_exe/src
make
cd && cp chapter_exe/src/chapter_exe JoinLogoScpTrialSetLinux/modules/join_logo_scp_trial/bin/chapter_exe
sudo cp -r JoinLogoScpTrialSetLinux/modules/join_logo_scp_trial /opt
cd /opt/join_logo_scp_trial
sudo npm install
sudo npm link
jlse --help
```
**ロゴデータ(.lgd)を`logo`フォルダに入れる必要があります(入れないとロゴ消しができません)。具体的には`/opt/join_logo_scp_trial/logo/`フォルダにロゴデータをあらかじめ入れておいてください。**  
  
最後のコマンドで以下のような表示になっていればインストール完了です。
```
$ jlse --help
オプション:
  --version     バージョンを表示                                          [真偽]
  --input, -i   path to ts file                                  [文字列] [必須]
  --filter, -f  enable to ffmpeg filter output        [真偽] [デフォルト: false]
  --encode, -e  enable to ffmpeg encode               [真偽] [デフォルト: false]
  --target, -t  select encord target
            [選択してください: "cutcm", "cutcm_logo"] [デフォルト: "cutcm_logo"]
  --option, -o  set ffmpeg option                      [文字列] [デフォルト: ""]
  --name, -n    set encordet file name                 [文字列] [デフォルト: ""]
  --remove, -r  remove avs files                      [真偽] [デフォルト: false]
  --help        ヘルプを表示       
```
※64bitOS環境ではchapter_exeのインストールで"-mfpu=neon"が定義されていないというエラーが出ることがあります。その場合は、`chapter_exe/src`内のMakefileの5行目の`-mfpu=neon`を削除することでmakeが可能です。

# delogo-AviSynthPlus-Linux
ロゴ消し用のプラグインをインストールします。
```
git clone https://github.com/tobitti0/delogo-AviSynthPlus-Linux.git
cd delogo-AviSynthPlus-Linux/src
make
sudo make install
```
以上でjoin_logo_scpのインストールは完了です。
# 補足
join_logo_scpの各種保存先は以下のディレクトリです。
```
/opt/join_logo_scp/
```
ロゴデータを入れる`logo`フォルダや、放送局別の設定ファイルを入れる`JL`フォルダ、
join_logo_scpの実行時の結果(.avs)が保存されている`result`フォルダなどが上述の階層に存在しています。  
<br>
また、`/opt/join_logo_scp/*`以外のビルドなどで使用したファイルやフォルダは削除しても問題ありません。
# EPGStationとの連携について

EPGStationとの連携に関しては以下のドキュメントをご覧ください。  
 - [EPGStationとの連携](EPGStation.md)
