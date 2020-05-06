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

## セットアップ方法
このセットアップにはDockerとDocker-composeが必要です。  
ローカルにインストールしたい場合はDockerファイルを読んで手順をなぞってください。  
初回は次の通りに実行します。
````
git clone --recursive https://github.com/tobitti0/JoinLogoScpTrialSetLinux.git
cd JoinLogoScpTrialSetLinux
cp -r modules/join_logo_scp_trial/JL .
cp -r modules/join_logo_scp_trial/setting .
cp -r modules/join_logo_scp_trial/src .
docker-compose up --build
````
FFmpegその他ライブラリをビルドしますのではやくても10分程度はかかります。  
環境次第ではもっとかかると思います。  
気長に待ってください。
こんな感じのnodejsのエラーが出たら完了です。  
````bash
join_logo_scp_trial    | > node src/jlse.js "-i"
join_logo_scp_trial    |
join_logo_scp_trial    | invalid file extension .
join_logo_scp_trial    | Options:
join_logo_scp_trial    |   --version     Show version number                                    [boolean]
join_logo_scp_trial    |   --input, -i   path to ts file                              [string] [required]
join_logo_scp_trial    |   --filter, -f  enable to ffmpeg filter output        [boolean] [default: false]
join_logo_scp_trial    |   --help        Show help                                              [boolean]
（下にも続く）
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
* 2020/05/06 公開
