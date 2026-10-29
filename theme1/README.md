# テーマ1：HTTPリクエストとレスポンスの基礎

[← 概要（テーマ0）](../README.md)

課題10〜19の作業内容と証跡をまとめます．コマンドの生の出力は [`evidence/theme1/`](../evidence/theme1/) に保存しています．

- 実行環境：macOS，`curl 8.7.1`（実行日：2026-10-30 JST）
- 動作確認用サービス：https://httpbin.org
- レスポンス中の `origin`（自分のグローバルIPアドレス）は `xxx.xxx.xxx.xxx` に伏せています．

| 課題 | 内容 | 状態 |
|---|---|---|
| 10 | HTTPとは何か | ✅ |
| 11 | デベロッパーツールで通信を観察 | ✅（スクリーンショットは追加予定） |
| 12 | curlでGET | ✅ |
| 13 | curlでPOST | ✅ |
| 14 | HTTPステータスコード | ✅ |
| 15 | HEADリクエスト | ✅ |
| 16 | クエリパラメータとURLエンコード | ✅ |
| 17 | HTTPメソッドの違い | ✅ |
| 18 | レスポンスのContent-Type | ✅ |
| 19 | ヘッダーとボディの違い | ✅ |

---

## 10. HTTPとは何か

HTTP（HyperText Transfer Protocol）は，クライアント（ブラウザ等）がサーバーに**リクエスト**を送り，サーバーが**レスポンス**を返す，という1往復でやり取りするWebの通信規約である．各リクエストは独立しており，サーバーは前回のやり取りを覚えない（**ステートレス**）ため，ログイン状態などはCookieで補う．

- **HTTP/1.1**：テキスト形式．1本の接続で基本的に1つずつ順番に処理するため，前の応答が遅いと後ろが待たされる．
- **HTTP/2**：バイナリ形式になり，1本のTCP接続上で複数のリクエストを同時に流せる（多重化）．ヘッダー圧縮もある．
- **HTTP/3**：TCPではなくUDPベースのQUICを使い，接続確立が速く，パケット損失の影響を受けにくい．

**HTTPS**はHTTPの通信をTLSで暗号化したもので，盗聴・改ざん・なりすましを防ぐ．HTTPの中身（メソッドやヘッダー）は同じで，その下にTLSの層が入る．HTTP/2・HTTP/3は実質的にHTTPS前提で使われている．

実際に `curl -v https://httpbin.org/get` を見ると，`SSL connection using TLSv1.3`（HTTPS）と `ALPN: server accepted h2`（HTTP/2で通信）が確認できた（課題12）．また example.com のレスポンスには `alt-svc: h3=":443"` があり，「HTTP/3でも接続できる」とサーバーが案内していた（課題15）．

---

## 11. ブラウザのデベロッパーツールで通信を観察する

**手順：** Chromeで「表示 → 開発 / 管理 → デベロッパーツール」（`⌥⌘I`）を開き，**Network**タブを選んだ状態で https://example.com を読み込む．一覧の先頭にある `example.com`（Type: document）をクリックし，**Headers**タブでリクエスト・レスポンスヘッダーを確認する．

<!-- DevToolsのスクリーンショットを evidence/theme1/11_devtools.png に保存したら，次の行のコメントを外す -->
<!-- ![DevToolsのNetworkタブ](../evidence/theme1/11_devtools.png) -->

同じページを `curl -v` で取得したときのヘッダーは次の通り（`>` がリクエスト，`<` がレスポンス）．ブラウザで見た場合もレスポンス側はほぼ同じ内容になる．

```
$ curl -s -v -o /dev/null https://example.com
> GET / HTTP/2
> Host: example.com
> User-Agent: curl/8.7.1
> Accept: */*
>
< HTTP/2 200
< date: Thu, 29 Oct 2026 16:57:46 GMT
< content-type: text/html; charset=utf-8
< server: cloudflare
< last-modified: Sat, 24 Oct 2026 11:34:40 GMT
< allow: GET, HEAD
< accept-ranges: bytes
< age: 3317
< cf-cache-status: HIT
< cf-ray: a523b8a4aba4c571-KIX
< alt-svc: h3=":443"; ma=86400
```

### 主要なヘッダーの解説

**リクエストヘッダー**

| ヘッダー | 意味 |
|---|---|
| `Host` | どのサイト宛てのリクエストか．1台のサーバーで複数サイトを扱うために必須． |
| `User-Agent` | クライアントの種類．curlでは `curl/8.7.1`，Chromeでは `Mozilla/5.0 (Macintosh; ...) Chrome/...` のような長い文字列になる． |
| `Accept` | 受け取りたいデータ形式．curlは `*/*`（何でも），ブラウザは `text/html,...` のようにHTMLを優先する． |
| `Accept-Encoding` / `Accept-Language` | ブラウザが付ける．対応する圧縮形式（gzip, br等）や希望する言語（ja等）． |

**レスポンスヘッダー**

| ヘッダー | 意味 |
|---|---|
| ステータス `200` | 成功． |
| `content-type: text/html; charset=utf-8` | ボディはUTF-8のHTMLである．ブラウザはこれを見て描画方法を決める． |
| `date` / `last-modified` | 応答した日時と，ページが最後に更新された日時． |
| `server: cloudflare` | 応答したサーバー（ここではCDNのCloudflare）． |
| `age` / `cf-cache-status: HIT` | CDNのキャッシュから返したこと，キャッシュされてから何秒経ったか． |
| `alt-svc: h3=":443"` | HTTP/3でも接続できるという案内． |

ブラウザとcurlの違いとして，ブラウザは `Accept-Language` や `Cookie` など多くのヘッダーを自動で付ける点が挙げられる．

---

## 12. curlコマンドでGETリクエストを送る

### オプションなし

```
$ curl -s https://httpbin.org/get
{
  "args": {},
  "headers": {
    "Accept": "*/*",
    "Host": "httpbin.org",
    "User-Agent": "curl/8.7.1"
  },
  "method": "GET",
  "origin": "xxx.xxx.xxx.xxx",
  "url": "https://httpbin.org/get"
}
```

httpbinは「受け取ったリクエストの内容」をJSONで返してくれる．

- `args`：クエリパラメータ（今回は付けていないので空）
- `headers`：curlが自動で付けたリクエストヘッダー（`Host`，`User-Agent`，`Accept`）
- `origin`：送信元のIPアドレス
- `url`：リクエストしたURL

オプションなしでは**ボディ（JSON）だけ**が表示される．

### `-i` を付けた場合

```
$ curl -s -i https://httpbin.org/get
HTTP/2 200
access-control-allow-credentials: true
access-control-allow-origin: *
access-control-expose-headers: Retry-After, RateLimit, RateLimit-Policy
content-length: 200
content-type: application/json
x-content-type-options: nosniff
date: Thu, 29 Oct 2026 16:56:41 GMT

{
  "args": {},
  ...（同じJSON）
}
```

`-i` では**レスポンスのステータス行とヘッダー**がボディの前に表示される．`content-type: application/json` からボディがJSONであること，`content-length: 200` からボディが200バイトであることがわかる．

### `-v` を付けた場合（抜粋）

```
$ curl -s -v https://httpbin.org/get
* Connected to httpbin.org (3.137.48.32) port 443
* ALPN: curl offers h2,http/1.1
* SSL connection using TLSv1.3 / AEAD-CHACHA20-POLY1305-SHA256 / [blank] / UNDEF
* ALPN: server accepted h2
* Server certificate:
*  subject: CN=httpbin.hbin.postmanlabs.com
*  subjectAltName: host "httpbin.org" matched cert's "httpbin.org"
*  issuer: C=US; O=Let's Encrypt; CN=YE1
*  SSL certificate verify ok.
* using HTTP/2
> GET /get HTTP/2
> Host: httpbin.org
> User-Agent: curl/8.7.1
> Accept: */*
>
* Request completely sent off
< HTTP/2 200
< content-length: 200
< content-type: application/json
< date: Thu, 29 Oct 2026 16:56:42 GMT
< ...
<
{ ...（同じJSON） }
```

全文：[`12_get_v.txt`](../evidence/theme1/12_get_v.txt)

`-v`（verbose）では通信の全過程が見える．

- `*` の行：接続先IP・TLSハンドシェイク・証明書の検証・HTTPバージョンの決定（ALPNでHTTP/2に決定）など，curl自身の情報
- `>` の行：**送ったリクエスト**（リクエスト行とヘッダー）
- `<` の行：**受け取ったレスポンス**（ステータス行とヘッダー）

`-i` はレスポンスヘッダーまでしか見えないが，`-v` は自分が送ったリクエストも確認できるのでデバッグに便利．なお `-v` の情報は標準エラー出力に出るため，`-o /dev/null` でボディを捨ててもヘッダーは見える．

---

## 13. curlコマンドでPOSTリクエストを送る

送信データはどれも同じJSON `{"name":"foo","age":20}`（23バイト）．

### ① `Content-Type: application/json` を付けた場合

```
$ curl -s -X POST https://httpbin.org/post -H 'Content-Type: application/json' -d '{"name":"foo","age":20}'
{
  "args": {},
  "headers": {
    "Accept": "*/*",
    "Content-Length": "23",
    "Content-Type": "application/json",
    "Host": "httpbin.org",
    "User-Agent": "curl/8.7.1"
  },
  "method": "POST",
  "origin": "xxx.xxx.xxx.xxx",
  "url": "https://httpbin.org/post",
  "data": "{\"name\":\"foo\",\"age\":20}",
  "files": {},
  "form": {},
  "json": {
    "age": 20,
    "name": "foo"
  }
}
```

### ② ヘッダーを付けない場合

```
$ curl -s -X POST https://httpbin.org/post -d '{"name":"foo","age":20}'
{
  "args": {},
  "headers": {
    "Accept": "*/*",
    "Content-Length": "23",
    "Content-Type": "application/x-www-form-urlencoded",
    "Host": "httpbin.org",
    "User-Agent": "curl/8.7.1"
  },
  "method": "POST",
  "origin": "xxx.xxx.xxx.xxx",
  "url": "https://httpbin.org/post",
  "data": "",
  "files": {},
  "form": {
    "{\"name\":\"foo\",\"age\":20}": ""
  },
  "json": null
}
```

### （参考）③ フォーム形式で送った場合

```
$ curl -s -X POST https://httpbin.org/post -d 'name=foo&age=20'
  ...
  "form": {
    "age": "20",
    "name": "foo"
  },
  "json": null
```

### 比較とまとめ

| | `Content-Type`（送信時） | `json` 欄 | `form` 欄 |
|---|---|---|---|
| ① ヘッダーあり | `application/json` | `{"age": 20, "name": "foo"}` | 空 |
| ② ヘッダーなし | `application/x-www-form-urlencoded`（curlの既定） | `null` | JSON文字列全体が1つのキーになる |
| ③ フォーム形式 | `application/x-www-form-urlencoded` | `null` | `{"age": "20", "name": "foo"}` |

- 送ったバイト列は①と②でまったく同じ（`Content-Length: 23`）だが，サーバーは**`Content-Type` ヘッダーを見て中身の解釈方法を決める**．
- ①ではJSONとして解析され，`age` は数値 `20` のまま届いた．`data` 欄には生の文字列も残っている．
- ②では `-d` の既定によりフォーム形式と見なされ，`key=value&...` として解析しようとした結果，JSON全体が「値が空のキー」として扱われてしまった．
- ③のようにフォームで送ると，値はすべて文字列（`"20"`）になる．
- **教訓：** JSONを送るときは必ず `Content-Type: application/json` を付ける（curl 7.82以降なら `--json '{...}'` でも同じことができる）．

---

## 14. HTTPステータスコード

先頭の数字で大まかな意味が決まる：**1xx** 情報，**2xx** 成功，**3xx** リダイレクト，**4xx** クライアント側のエラー，**5xx** サーバー側のエラー．

| コード | 名前 | 意味・よくある場面 |
|---|---|---|
| 200 | OK | 成功．GETでデータが返ってきた． |
| 201 | Created | 作成成功．POSTで新しいデータが作られた． |
| 204 | No Content | 成功したが返すボディはない．DELETEの成功時など． |
| 301 | Moved Permanently | 恒久的に移転した．`Location` ヘッダーの新URLへ．ブックマークや検索エンジンも新URLに置き換える． |
| 302 | Found | 一時的に別のURLへ．ログイン後の画面遷移など． |
| 304 | Not Modified | 前回から変わっていないので，手元のキャッシュを使ってよい（ボディなし）． |
| 400 | Bad Request | リクエストの形式が不正（JSONが壊れている，必須項目がない等）． |
| 401 | Unauthorized | 認証が必要／認証に失敗した（ログインしていない，トークンが無効）． |
| 403 | Forbidden | 誰かはわかっているが，権限がないので拒否． |
| 404 | Not Found | 指定したリソースが存在しない． |
| 429 | Too Many Requests | 短時間にリクエストを送りすぎ（レート制限）．`Retry-After` で待ち時間を示すことがある． |
| 500 | Internal Server Error | サーバー内部のエラー（プログラムのバグ等）． |
| 503 | Service Unavailable | 過負荷やメンテナンスで一時的に利用できない． |

**401と403の違い：** 401は「あなたが誰かわからない」，403は「あなたが誰かはわかるが許可しない」．

### 実際に受け取って確認

`https://httpbin.org/status/<コード>` は指定したステータスコードをそのまま返す．`-w '%{http_code}'` で受け取ったコードだけを表示した．

```
$ for c in 200 201 204 301 302 304 400 401 403 404 429 500 503; do
    echo "$c -> $(curl -s -o /dev/null -w '%{http_code} %{redirect_url}' https://httpbin.org/status/$c)"
  done
200 -> 200
201 -> 201
204 -> 204
301 -> 301 https://httpbin.org/redirect/1
302 -> 302 https://httpbin.org/redirect/1
304 -> 304
400 -> 400
401 -> 401
403 -> 403
404 -> 404
429 -> 429
500 -> 500
503 -> 503
```

404と302はヘッダーも確認した．302には転送先を示す `location` ヘッダーが付いている．curlは既定ではリダイレクトを追わない（追うには `-L`）．

```
$ curl -s -i https://httpbin.org/status/404
HTTP/2 404
content-length: 0
...

$ curl -s -i https://httpbin.org/status/302
HTTP/2 302
content-length: 0
location: /redirect/1
...
```

---

## 15. HEADリクエストを送る

`curl -I`（`--head`）でHEADリクエストを送ると，**ボディなしでヘッダーだけ**が返ってくる．

```
$ curl -s -I https://example.com
HTTP/2 200
date: Thu, 29 Oct 2026 16:56:52 GMT
content-type: text/html; charset=utf-8
server: cloudflare
last-modified: Sat, 24 Oct 2026 11:34:40 GMT
allow: GET, HEAD
accept-ranges: bytes
age: 3263
cf-cache-status: HIT
cf-ray: a523b750ed02dfa1-KIX
alt-svc: h3=":443"; ma=86400
```

同じサイトに `http://`（暗号化なし）で送った場合：

```
$ curl -s -I http://example.com
HTTP/1.1 200 OK
Date: Thu, 29 Oct 2026 16:56:52 GMT
Content-Type: text/html; charset=utf-8
Connection: keep-alive
Server: cloudflare
...
```

GitHubにも送ってみた（抜粋）：

```
$ curl -s -I https://github.com
HTTP/2 200
content-type: text/html; charset=utf-8
etag: W/"6df91c1271fb31fb98fb2dba5b2d16b5"
cache-control: max-age=0, private, must-revalidate
strict-transport-security: max-age=31536000; includeSubdomains; preload
x-frame-options: deny
x-content-type-options: nosniff
server: github.com
```

### 気づいた点

- ボディを受け取らないので速い．「ページが存在するか」「更新されたか（`last-modified`）」「ファイルの種類やサイズ」だけ知りたいときに使える．
- `allow: GET, HEAD` から，example.com はGETとHEADだけを受け付けることがわかる．
- `https://` ではHTTP/2で，`http://` ではHTTP/1.1で応答した．HTTP/2ではヘッダー名がすべて小文字になり，ステータス行の `OK` のような説明文もなくなる．
- `cf-ray` の末尾 `KIX` は関西のCloudflareの拠点から返ってきたことを示しているようだ．
- GitHubは `strict-transport-security`（今後はHTTPSでのみ接続せよ），`x-frame-options: deny`（他サイトに埋め込ませない）など，セキュリティ関連のヘッダーを多く返していた．また `etag` はキャッシュの検証（304を返すかどうかの判定）に使われる．
- （GitHubは `set-cookie` も返していたが，セッション情報なので記録から除いた．）

---

## 16. クエリパラメータ付きのリクエスト

### 基本

```
$ curl -s 'https://httpbin.org/get?name=foo&age=20'
{
  "args": {
    "age": "20",
    "name": "foo"
  },
  "headers": { ... },
  "method": "GET",
  "origin": "xxx.xxx.xxx.xxx",
  "url": "https://httpbin.org/get?name=foo&age=20"
}
```

- `?` 以降が**クエリ文字列**で，`キー=値` を `&` でつないで渡す．
- サーバーはこれを分解し，`args` に `{"name": "foo", "age": "20"}` として受け取っている（課題12では空だった）．
- `age` は `"20"` と文字列になっている．クエリパラメータには型がなく，すべて文字列として届く．
- URLに `&` を含むので，シェルに解釈されないよう `'...'` で囲む必要がある（囲まないと `&` でコマンドがバックグラウンド実行になってしまう）．

同じキーを複数回指定すると配列になった：

```
$ curl -s 'https://httpbin.org/get?tag=a&tag=b'
  "args": {
    "tag": ["a", "b"]
  },
```

### 日本語・空白を含む値とURLエンコード

URLに使える文字は英数字と一部の記号だけなので，日本語や空白は**パーセントエンコーディング**（UTF-8のバイトを `%XX` で表す）に変換する必要がある．

**そのまま送るとエラーになる：**

```
$ curl -sS 'https://httpbin.org/get?name=山田 太郎'
curl: (3) URL rejected: Malformed input to a URL function
```

**手でエンコードして送る：**（`山`=`%E5%B1%B1`，空白=`%20`，`&`=`%26`）

```
$ curl -s 'https://httpbin.org/get?name=%E5%B1%B1%E7%94%B0%20%E5%A4%AA%E9%83%8E&q=a%26b'
  "args": {
    "name": "山田 太郎",
    "q": "a&b"
  },
```

**curlにエンコードさせる：**（`-G` でGETにし，`--data-urlencode` の値をクエリに付ける）

```
$ curl -s -G https://httpbin.org/get --data-urlencode 'name=山田 太郎' --data-urlencode 'q=a&b'
  "args": {
    "name": "山田 太郎",
    "q": "a&b"
  },
  "url": "https://httpbin.org/get?name=%e5%b1%b1%e7%94%b0+%e5%a4%aa%e9%83%8e&q=a%26b"
```

- サーバー側ではデコードされ，`args` に元の「山田 太郎」が戻っている．
- 値の中の `&` は区切り文字と区別するため `%26` にする必要がある．エンコードしないと `q=a` と `b=` に分かれてしまう．
- 空白は `%20` でも `+` でも同じく空白として解釈された（`--data-urlencode` は `+` を使う）．

---

## 17. HTTPメソッドの違い

| メソッド | 用途 | 例 | 安全 | 冪等 | ボディ |
|---|---|---|---|---|---|
| GET | リソースの取得 | 記事一覧を見る `GET /articles` | ○ | ○ | なし |
| HEAD | GETのヘッダーだけ取得 | 存在・更新日時の確認 | ○ | ○ | なし |
| POST | 新規作成・処理の実行 | 記事を投稿する `POST /articles` | × | × | あり |
| PUT | リソースを丸ごと置き換え（なければ作成） | 記事3を全項目書き換え `PUT /articles/3` | × | ○ | あり |
| PATCH | リソースの一部だけ変更 | 記事3のタイトルだけ変更 `PATCH /articles/3` | × | ×（※） | あり |
| DELETE | リソースの削除 | 記事3を削除 `DELETE /articles/3` | × | ○ | 通常なし |
| OPTIONS | 使えるメソッド等の問い合わせ | CORSの事前確認（プリフライト） | ○ | ○ | なし |

### 「安全（safe）」と「冪等（idempotent）」

- **安全**：サーバー上のデータを変更しない（読むだけ）．GET・HEAD・OPTIONSが該当．何度アクセスしても，リンクを先読みされても問題ない．
- **冪等**：同じリクエストを1回送っても何回送っても，**サーバーの最終的な状態が同じ**になる．
  - PUT：「記事3をこの内容にする」は何回やっても同じ結果．
  - DELETE：「記事3を削除」は1回目で消え，2回目以降も「消えている」状態は同じ（2回目のレスポンスが404になっても状態は同じなので冪等）．
  - POST：「記事を投稿」を2回送ると記事が2つできるので冪等ではない．通信が切れたときに**自動で再送してよいか**の判断基準になる．
- 安全なメソッドは必ず冪等だが，冪等だからといって安全とは限らない（PUT，DELETE）．
- ※PATCHは「タイトルを○○にする」なら冪等になるが，「閲覧数を1増やす」のような書き方もできるため，一般には冪等と保証されない．

### 実際に送って確認

httpbinにはメソッドごとのエンドポイントがあり，`method` 欄に受け取ったメソッドが表示される．

```
$ curl -s -X PUT https://httpbin.org/put -H 'Content-Type: application/json' -d '{"name":"foo"}'
  "method": "PUT",
  "json": { "name": "foo" }

$ curl -s -i -X DELETE https://httpbin.org/delete
HTTP/2 200
  "method": "DELETE",

$ curl -s -i -X OPTIONS https://httpbin.org/get
HTTP/2 200
access-control-allow-methods: GET, POST, HEAD, PUT, DELETE, PATCH, QUERY, OPTIONS
content-length: 0
```

OPTIONSでは，ボディは空で `access-control-allow-methods` ヘッダーに使えるメソッドの一覧が返ってきた．

---

## 18. レスポンスのContent-Type

`curl -s -o /dev/null -w '%{content_type}' <URL>` で，レスポンスの `Content-Type` だけを取り出した．

```
$ for u in https://example.com https://httpbin.org/json https://httpbin.org/html \
           https://httpbin.org/xml https://httpbin.org/image/png https://httpbin.org/image/jpeg \
           https://httpbin.org/image/svg https://httpbin.org/robots.txt https://www.google.com/favicon.ico; do
    printf "%-40s %s\n" "$u" "$(curl -s -o /dev/null -w '%{content_type}' $u)"
  done
```

| 種類 | URL | Content-Type |
|---|---|---|
| HTML | https://example.com | `text/html; charset=utf-8` |
| HTML | https://httpbin.org/html | `text/html; charset=utf-8` |
| JSON | https://httpbin.org/json | `application/json` |
| XML | https://httpbin.org/xml | `application/xml` |
| テキスト | https://httpbin.org/robots.txt | `text/plain` |
| 画像（PNG） | https://httpbin.org/image/png | `image/png` |
| 画像（JPEG） | https://httpbin.org/image/jpeg | `image/jpeg` |
| 画像（SVG） | https://httpbin.org/image/svg | `image/svg+xml` |
| 画像（アイコン） | https://www.google.com/favicon.ico | `image/x-icon` |

- 形式は `タイプ/サブタイプ`（MIMEタイプ）で，`text/`・`application/`・`image/` などの大分類がある．
- テキスト系には `; charset=utf-8` のように文字コードが付くことがある．
- クライアントは**URLの拡張子ではなくContent-Typeを見て**処理方法（HTMLとして描画する，画像として表示する，JSONとして解析する）を決める．課題13のように，リクエスト側でも同じヘッダーでデータ形式を伝える．
- 画像をそのままターミナルに出すと文字化けするので，`-o /dev/null` で捨てるか `-o file.png` で保存する．

---

## 19. レスポンスヘッダーとボディの違い

HTTP/1.1 で `curl -v` を実行した出力（`--http1.1` を指定，TLS部分は省略）：

```
$ curl -s -v --http1.1 https://httpbin.org/get
> GET /get HTTP/1.1                  ← ①開始行（リクエスト行）
> Host: httpbin.org                  ← ②ヘッダー
> User-Agent: curl/8.7.1
> Accept: */*
>                                    ← ③空行（ここでリクエスト終わり．GETなのでボディなし）
< HTTP/1.1 200 OK                    ← ①開始行（ステータス行）
< content-length: 200                ← ②ヘッダー
< content-type: application/json
< date: Thu, 29 Oct 2026 16:57:16 GMT
< ...
<                                    ← ③空行
{                                    ← ④ボディ（200バイト）
  "args": {},
  ...
}
```

### 解説

HTTPメッセージは，リクエストもレスポンスも「**開始行 → ヘッダー → 空行 → ボディ**」の順に並んでいる．**開始行**は1行目で，リクエストなら「メソッド・パス・バージョン」（`GET /get HTTP/1.1`），レスポンスなら「バージョン・ステータスコード・説明」（`HTTP/1.1 200 OK`）が書かれる．続く**ヘッダー**は `名前: 値` の行の並びで，ボディの形式（`content-type`）や長さ（`content-length`）など「中身についての付帯情報」を伝える．**空行**はヘッダーの終わりを示す区切りで，これより後ろが**ボディ**，つまり本当に届けたいデータ（HTMLやJSON，画像など）になる．GETのリクエストのようにボディがない場合もある．受け取る側はまずヘッダーを読み，`content-length` 分だけボディを読めば1つのメッセージを取り出せる．

curlの表示では `-i` でヘッダー＋ボディ，`-I` でヘッダーのみ，オプションなしでボディのみが出る．また HTTP/2 の `-v` 出力では `[:method: GET]` のような疑似ヘッダーも見えるが，これはHTTP/2が開始行をヘッダーの形でバイナリ送信しているためで，論理的な構造（開始行・ヘッダー・ボディ）は同じである．
