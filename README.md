# JoinLogoScpTrialSet for Linux and Avisynth+3.5.x
## 概要

[sogaani][1]氏が移植された[Linux対応版join_logo_scp][2]を元に改造し  
Native Linuxに対応した[AviSynth+][3][3.5.x][4]で動作できるようにしたもののセット。  
DockerとDocker-composeを用いて動作させます。  

[1]:https://github.com/sogaani
[2]:https://github.com/sogaani/JoinLogoScp
[3]:https://github.com/AviSynth/AviSynthPlus
[4]:https://github.com/AviSynth/AviSynthPlus/releases/

### 確認環境
同梱しているDocker環境にて動作を確認しました。  

## 動作確認用環境セットアップ方法
このセットアップにはDockerとDocker-composeが必要です。  
ローカルにインストールしたい場合はDockerファイルを読んで手順をなぞってください。  
あくまで、動作確認用ですので使い込みたい方は自前でDockerfileを作成することや、他のDockerfileに組み込むことを検討してください。
初回は次の通りに実行します。
````
git clone --recursive https://github.com/tobitti0/JoinLogoScpTrialSetLinux.git
cd JoinLogoScpTrialSetLinux
cp -r modules/join_logo_scp_trial/JL .
cp -r modules/join_logo_scp_trial/setting .
cp -r modules/join_logo_scp_trial/src .
docker-compose up --build
````
[docker-avisynthplus](https://github.com/users/tobitti0/packages/container/package/docker-avisynthplus)をベースイメージとして使用します。
ある程度のFFmpegが使用できると思います。

次のログが出たら完了です。  
````bash
Successfully tagged join_logo_scp_trial:latest
Recreating join_logo_scp_trial ... done
Attaching to join_logo_scp_trial
join_logo_scp_trial    |
join_logo_scp_trial    | > join_logo_scp_trial@1.0.0 start /join_logo_scp_trial
join_logo_scp_trial    | > node src/jlse.js "-i" "--help"
join_logo_scp_trial    |
join_logo_scp_trial    | Options:
join_logo_scp_trial    |   --version      Show version number                                   [boolean]
join_logo_scp_trial    |   --input, -i    path to ts file                             [string] [required]
join_logo_scp_trial    |   --filter, -f   enable to ffmpeg filter output       [boolean] [default: false]
join_logo_scp_trial    |   --encode, -e   enable to ffmpeg encode              [boolean] [default: false]
join_logo_scp_trial    |   --target, -t   select encord target
join_logo_scp_trial    |                         [choices: "cutcm", "cutcm_logo"] [default: "cutcm_logo"]
join_logo_scp_trial    |   --option, -o   set ffmpeg option                        [string] [default: ""]
join_logo_scp_trial    |   --outdir, -d   set encorded file dir                    [string] [default: ""]
join_logo_scp_trial    |   --outname, -n  set encorded file name                   [string] [default: ""]
join_logo_scp_trial    |   --remove, -r   remove avs files                     [boolean] [default: false]
join_logo_scp_trial    |   --help         Show help                                             [boolean]
join_logo_scp_trial exited with code 0
````
logoフォルダが生成されていると思うので、そこにロゴデータを入れておきます。
## 使用方法
docker-compose.ymlのある場所で、次のコマンドを入力して実行します。
````
docker-compose run --rm -v 「TSファイルのフォルダの絶対パス」:/ts \
                            join_logo_scp_trial /ts/「TSファイルの名前（拡張子含む）
````
(上のは見やすくするために改行してますが、別に一行でもいいです。）  

例:~/record/ts/局名_タイトル第1話.tsを解析する場合  
`docker-compose run --rm -v ~/record/ts:/ts join_logo_scp_trial /ts/局名_タイトル第1話.ts`  
resultフォルダの中のファイル名のフォルダに解析結果と、カット用のavsが保存されます。  
join_logo_scp_trialの詳しい使用方法は、[こちら][5]を確認してください。 

[5]:https://github.com/tobitti0/join_logo_scp_trial/blob/master/README.md

## EPGStationで使用する

LinuxなEPGStationでDocker環境の場合の導入方法は[こちら][6]

[6]:https://tobitti.net/blog/Ubuntu-EPGStation-JoinLogoScpTrial/

（私はEPGStationで呼び出し、CM解析をし、ロゴ消し、CMカット、エンコードまで動作させています。）   
Dockerで動作しているEPGStationを利用していますが、動作にはHOMEの環境変数が必須です。  
ないとchapter_exe,logoframe,join_logo_scpから、avsファイルを見つけることができず動作しません。  
Dockerでの動作しか確認していませんが、spawnする際に次のようにすることで動作します。  
```
var env = Object.create( process.env );
env.HOME = '/root';
const child = spawn('jlse', jlse_args, {env: env});
```
（Dockerで動作させていない場合はHOMEの値は異なると思います。Dockerだといじっていなければrootです。）

## ファイル構成
* docker              : join_logo_scp動作確認環境構築用Dockerfile

以下はmodulesの中にsubmoduleとして入っています。個別に利用する場合はそちらのReadmeを見てください。
* logoframe           : 透過ロゴ表示区間検出 ver1.16（要AviSynth環境）
* chapter_exe         : 無音＆シーンチェンジ検索chapter_exeの改造版（要AviSynth環境）
* join_logo_scp       : ロゴと無音シーンチェンジを使ったCM自動カット位置情報作成
* join_logo_scp_trial : join_logo_scp動作確認用スクリプト

## 謝辞
各種ツールを作成された方々、  
Linuxに移植されたsogaani氏に深く感謝いたします。
## 履歴
* 2020/05/30 エンコードまで一括で行えるようにしたjoin_logo_scp_trialに更新
* 2020/05/06 公開
