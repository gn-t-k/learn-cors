# CORSについて理解する

## CORSとは

### 概要

Cross Origin Resource Sharingの略。

通常、ブラウザは同一オリジンポリシーによって、オリジンAの文書やスクリプトなどのリソースからオリジンBのリソースにはアクセスできないように制限されている。CORSは、追加のHTTPヘッダを使用することで、同一オリジンポリシーによるリソース間のアクセスの制限を緩和するためのブラウザの仕組み。XMLHttpRequestやFetch APIを使用してクロスドメインのリソースにリクエストを送信する場合は、CORSの仕様に則ってリクエストを送信する必要がある。

### 何を解決したか

Ajaxの普及により異なるオリジンのAPIを呼び出したいという需要が生まれたが、CORSの仕組みがないブラウザでは同一オリジンポリシーによって異なるオリジンのリソースへのアクセスは拒否されていた。こういった状況の中で、クロスドメインアクセスを実現したいという要求に答えるため考案されたのがCORS。CORSの規定に則ってブラウザとサーバーでアクセス制御に関する情報をやりとりすれば、安全にクロスドメインアクセスを実現ができる。

### CORSを使用したリクエストのシナリオ

CORSの仕様に則ってクロスドメインのリソースへアクセスする方法は2パターンがある。

- クロスドメインのリソースにアクセスするリクエストを直接送信する「**シンプルなリクエスト**」のパターン
- クロスドメインアクセスが可能か確認するリクエスト（**プリフライトリクエスト**）を送信して、そのレスポンスを受けた後に改めてクロスドメインのリソースアクセスを行うパターン

CORSで定義された条件を満たせばシンプルなリクエストが送信され、そうでなければプリフライトリクエストからやりとりが始まる。

#### シンプルなリクエスト

以下の条件をすべて満たすリクエストは、クロスドメインのリソースに直接送信できる。

- メソッドが以下のいずれかである。
  - GET
  - HEAD
  - POST
- 以下のHTTPヘッダ以外のHTTPヘッダが設定されていない（ブラウザによって自動的に追加されたものを除く）
  - Accept
  - Accept-Language
  - Content-Language
  - Content-type
  - DPR
  - Downlink
  - Save-Data
  - Viewport-Width
  - Width
- Content-Typeヘッダに以下の値以外の値が設定されていない
  - application/x-www-form-urlencoded
  - multipart/form-data
  - text/plain
- リクエストに使用されるどのXMLHttpRequestUploadにもイベントリスナーが登録されていない
- リクエストにReadableStreamオブジェクトが使用されていないこと

以下は、`https://foo.example`のコンテンツが`https://bar.other`にあるコンテンツを呼び出すときのコードの例。

```javascript
const xhr = new XMLHttpRequest();
const url = 'https://bar.other/resources/public-data/';

xhr.open('GET', url);
xhr.onreadystatechange = someHandler;
xhr.send();
```

このとき送信されるリクエストは以下の通り。

```http
GET /resources/public-data/ HTTP/1.1
Host: bar.other
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:71.0) Gecko/20100101 Firefox/71.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Connection: keep-alive
Origin: https://foo.example
```

`Origin`ヘッダで`https://foo.example`からのリクエストであることをサーバーに伝えている。
サーバーから返ってくるレスポンスは以下の通り。

```http
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 00:23:53 GMT
Server: Apache/2
Access-Control-Allow-Origin: *
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Transfer-Encoding: chunked
Content-Type: application/xml

[…XML データ…]
```

`Access-Control-Allow-Origin`ヘッダで全てのドメインからのアクセスを許可することをクライアントに伝えている。つまり、クライアントからのリクエストは成功する。サーバー側でリソースへのアクセスを制限したい場合、例えば以下のように設定すれば`https://foo.example`からのリクエストのみリソースへアクセスできるよう制限することができる。

```http
Access-Control-Allow-Origin: https://foo.example
```

#### プリフライトリクエスト

## モックの実装
