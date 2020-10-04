# EPGStationとの連携
## 目次
- [はじめに](#はじめに)
- [インストール](#インストール)
- [jsでの連携例](#jsでの連携例)
- [shでの連携例](#shでの連携例)

## はじめに
不備があればプルリクや、Issueいただけると助かります。  
このドキュメント制作時点でのEPGStationでは、外部コマンド実行時に環境変数HOMEを渡しません。  
JoinLogoScpLinuxでは動作にHOMEの環境変数が必須です。  
そのため連携させるには自前で定義する必要があります。  
jsファイルでは次のようにして自前で定義することができます。
```
var env = Object.create( process.env );
env.HOME = '/root';
const child = spawn('jlse', jlse_args, {env: env});
```
shファイルでは次のようにして定義することができます。
```
export HOME="/root"
```
定義する値はDockerで実行している場合は`/root`ですが、ローカル環境だと環境により異なります。  
`echo $HOME`を叩いて出てきた文字を`/root`と置き換えてください。  
これらは以下の連携例においても同じです。ローカル環境では`/root`を置き換えてください。

## インストール
Avisynth+3.5.X以上、FFmpeg、l-smash、l-smash-worksを導入の上で本ツールをビルドしてください。  
ビルドについては他ドキュメントを参照してください。  
連携例を使用するにあたっては、`jlse --help`を実行し、実行できることを確認してください。  
(Dockerで実行している場合はDocker内で実行できることを確認してください。)

## jsでの連携例
`epgstation/config/jlse.js`
```
const spawn = require('child_process').spawn;
const ffmpeg = process.env.FFMPEG;

const input = process.env.INPUT;
const output = process.env.OUTPUT;
const analyzedurationSize = '10M'; // Mirakurun の設定に応じて変更すること
const probesizeSize = '32M'; // Mirakurun の設定に応じて変更すること
const maxMuxingQueueSize = 1024;
const dualMonoMode = 'main';
const videoHeight = parseInt(process.env.VIDEORESOLUTION, 10);
const isDualMono = parseInt(process.env.AUDIOCOMPONENTTYPE, 10) == 2;
const audioBitrate = videoHeight > 720 ? '192k' : '128k';
const preset = 'medium';
const codec = 'libx264'; 
const crf = 23;

const args = ['-y', '-analyzeduration', analyzedurationSize, '-probesize', probesizeSize];

const path = require('path');
const output_name = path.basename(output, path.extname(output));
const output_dir = path.dirname(output);

// dual mono 設定
if (isDualMono) {
    Array.prototype.push.apply(args, ['-dual_mono_mode', dualMonoMode]);
}

// メタ情報を先頭に置く
Array.prototype.push.apply(args,['-movflags', 'faststart']);

// 字幕データを含めたストリームをすべてマップ
//Array.prototype.push.apply(args, ['-map', '0', '-ignore_unknown', '-max_muxing_queue_size', maxMuxingQueueSize, '-sn']);

// video filter 設定
let videoFilter = 'yadif';
if (videoHeight > 720) {
    videoFilter += ',scale=-2:720'
}
Array.prototype.push.apply(args, ['-vf', videoFilter]);

// その他設定
Array.prototype.push.apply(args,[
    '-preset', preset,
    '-aspect', '16:9',
    '-c:v', codec,
    '-crf', crf,
    '-f', 'mp4',
    '-c:a', 'aac',
    '-ar', '48000',
    '-ab', audioBitrate,
    '-ac', '2'
]);

let str = '';
for (let i of args) {
    str += ` ${ i }`
}
console.error(str);

const jlse_args = ['-i', input, '-e', '-o', str,'-r', '-d', output_dir, '-n', output_name];
console.error(jlse_args);

var env = Object.create( process.env );
env.HOME = '/root';
console.error(env);
const child = spawn('jlse', jlse_args, {env:env});

child.stderr.on('data', (data) => { console.error(String(data)); });

child.on('error', (err) => {
    console.error(err);
    throw new Error(err);
});

process.on('SIGINT', () => {
    child.kill('SIGINT');
});
```
`epgstation/config/config.json`
encode
```
{
    "name": "jls",
    "cmd": "%NODE% %ROOT%/config/jlse.js",
    "suffix": ".mp4",
    "default": true
}
```

## shでの連携例
`epgstation/config/jlse.sh`
```
#!/bin/bash
export HOME="/root"

input=$1
outpath=$2
outfilename=`basename "$2" .mp4`
outdir=`dirname "$2"`

analyzedurationSize='10M' #Mirakurun の設定に応じて変更すること
probesizeSize='32M' #Mirakurun の設定に応じて変更すること
maxMuxingQueueSize=1024
dualMonoMode='main'
preset='medium'
codec='libx264'
crf=23

FFOPTION="-y -analyzeduration $analyzedurationSize -probesize $probesizeSize -movflags faststart -vf yadif -preset $preset -aspect 16:9 -c:v $codec -crf $crf -f mp4 -c:a aac -ar 48000 -ab 192k -ac 2"

jlse -i "$1" -e -o " $FFOPTION" -r -d "$outdir" -n "$outfilename"
```
`epgstation/config/config.json`
encode
```
{
    "name": "jls",
    "cmd": "/bin/bash %ROOT%/config/jlse.sh %INPUT% %OUTPUT%",
    "suffix": ".mp4",
    "default": true
}
```

