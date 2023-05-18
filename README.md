# node-soap-doc-ja <!-- omit in toc -->

> [Node.js用SOAP通信モジュールnode-soapののREADME](https://github.com/vpulim/node-soap/blob/master/Readme.md)の日本語訳です

> node.js用SOAPクライアント・サーバー

このモジュールを使用することで、WebサービスへのSOAP通信やSOAPサービスを提供するサーバーの構築が可能になる。


- [機能](#機能)
- [インストール](#インストール)
- [サポート](#サポート)
- [モジュール](#モジュール)
  - [soap.createClient(url\[, options\], callback)](#soapcreateclienturl-options-callback)
  - [soap.createClientAsync(url\[, options\])](#soapcreateclientasyncurl-options)
  - [soap.listen(server, path, service, wsdl, callback)](#soaplistenserver-path-service-wsdl-callback)
  - [soap.listen(server, options)](#soaplistenserver-options)
  - [サーバーログ](#サーバーログ)
  - [サーバーイベント](#サーバーイベント)
  - [片道呼び出しでのサーバーレスポンス](#片道呼び出しでのサーバーレスポンス)

## 機能

- シンプルなAPI
- RPCとドキュメント、どちらのスキーマ型にもハンドル可能
- multiRef SOAPメッセージに対応 ([@kaven276](https://github.com/kaven276)提供)
- 同期と非同期、どちらのハンドラーにも対応
- WS-Security UsernameToken Profile 1.0に対応
- [Express](http://expressjs.com/)ベースのWebサーバーに対応 (body要素のparserミドルウェアが使用可能)

## インストール

```
npm install soap
```

## サポート

Glitterでコミュニティーサポートを受け付けてる: [![Gitter chat][gitter-image]][gitter-url]

Google Form上では、メンテナーもサポートしている

プルリクエストに集中して対応するため、GitHub issueは無効にしている [(#731)](https://github.com/vpulim/node-soap/pull/731)

## モジュール

### soap.createClient(url[, options], callback)

WSDL URLをもとに、SOAPクライアントを作成する。ローカルファイルパスにも対応。

- `url` (*string*): HTTP/HTTPS URLかローカルファイルパス
- `options` (*Object*):
  - `endpoint` (*string*): SOAPサービスがWSDLファイル中で定義したホストを上書きする
  - `envelopeKey` (*string*): カスタムエンベロープキーを設定する (**デフォルト**: `'soap'`)
  - `preserveWhitespace` (*boolean*): textとcdata中にある先頭と末尾の空白文字を維持する
  - `escapeXML` (*boolean*): SOAPメッセージ中のXML特殊文字(`&`、`>`、`<`など)をエスケープする (**デフォルト**: `true`)
  - `suppressStack` (*boolean*): エラーメッセージのスタックトレースの表示を一部省略する
  - `returnFault` (*boolean*): 不正なリクエストに対して`Invalid XML`SOAP faultを返す (**デフォルト**: `false`)
  - `forceSoap12Headers` (*boolean*): SOAP 1.2を有効にする
  - `httpClient` (*Object*): ビルトインのHTTPクライアントをオーバーライドする。`request(rurl, data, callback, exheaders, exoptions)`を埋め込む必要がある
  - `request` (*Object*): デフォルトのリクエストモジュール([Axios](https://axios-http.com/) `v0.40.0`)をオーバーライドする
  - `wsdl_headers` (*Object*): WSDLリクエストを送信するためのHTTPヘッダーと値を設定する
  - `wsdl_options` (*Object*): WSDLリクエスト上のリクエストモジュールのオプションを設定する。デフォルトのリクエストモジュールを使用している場合は、[Request Config | Axios Docs](https://axios-http.com/docs/req_config)を参照
  - `disableCache` (*boolean*): WSDLファイルとオプションオブジェクトのキャッシングを防ぐ
  - `overridePromiseSuffix` (*string*): PromiseメソッドのWSDL操作名のsuffixをオーバーライドする。WSDL操作名の終わりが`Async`である場合、このオプションを設定する必要がある。 (**デフォルト**: `Async`)
  - `normalizeNames` (*boolean*): WSDL操作名中の非識別子(`[^a-z&_0-9]`)を`_`に置換する。クライアントが使用しているWSDLに`soap:method`と`soap-method`のような2つの操作があると上書きされる。この場合は、代わりに`client['soap:method']()`のようなブラケット記法を使う必要がある。
  - `namespaceArrayElements` (*boolean*): 非標準の配列構造をサポートする。JSON配列`{list: [{elem: 1}, {elem: 2}]}`はXML`<list><elem>1</elem></list> <list><elem>2</elem></list>`に変換される。`false`の場合、XML`<list> <elem>1</elem> <elem>2</elem> </list>`に変換される。 (**デフォルト**: `false)
  - `stream` (*boolean): XML SOAPレスポンスをパースするストリームを使用する。(**デフォルト**: `false`)
  - `returnSaxStream` (*boolean*): SAXストリームを返し、XMLのパースの責任をエンドユーザーに移す
  - `parseResponseAttachments` (*boolean*): MTOMアタッチメントをもつmultipart/relatedレスポンスとして返す。SOAPクライアントの`lastResponseAttachments`プロパティ上のアタッチメントを参照する。 (**デフォルト**: `false`)
- `callback` (*Function*): 
  - `err` (*Error |*)
  - `result` (*Any*)
- Returns: `(Client)`

**使用例**
```js
  var soap = require('soap');
  var url = 'http://example.com/wsdl?wsdl';
  var args = {name: 'value'};

  soap.createClient(url, {}, function(err, client) {
      client.MyFunction(args, function(err, result) {
          console.log(result);
      });
  });
```

注意点: 0.10.Xより上のnodeを使用している場合、長いチャンクレスポンスの切り捨てを避けるためにSOAPヘッダーに`{connection: 'keep-alive'}`を指定する必要の可能性がある。

### soap.createClientAsync(url[, options])

WSDL URLをもとに、SOAPクライアントを作成する。ローカルファイルパスにも対応。

与えられたWSDLファイルをもとに、`Promise<Client>`を構築する。

- `url` (*string*): HTTP/HTTPS URLかローカルファイルパス
- `options` (*Object*): [soap.createClient(url[, options], callback)](#soapcreateclienturl-options-callback)の説明を参照
- Returns: `Promise<Client>`

**使用例**

```js
  var soap = require('soap');
  var url = 'http://example.com/wsdl?wsdl';
  var args = {name: 'value'};

  // then/catch
  soap.createClientAsync(url).then((client) => {
    return client.MyFunctionAsync(args);
  }).then((result) => {
    console.log(result);
  });

  // async/await
  var client = await soap.createClientAsync(url);
  var result = await client.MyFunctionAsync(args);
  console.log(result[0]);
```

注意点: 0.10.Xより上のnodeを使用している場合、長いチャンクレスポンスの切り捨てを避けるためにSOAPヘッダーに`{connection: 'keep-alive'}`を指定する必要の可能性がある。

### soap.listen(server, path, service, wsdl, callback)

パス上でlistenし、サービスを提供するSOAPサーバーを作成する

### soap.listen(server, options)

パス上でlistenし、サービスを提供するSOAPサーバーを作成する

- `server` (*Object*): [http](https://nodejs.org/api/http.html)サーバーか[Express](http://expressjs.com/)フレームワークベースのサーバー
- `path` (*string*)
- `options` (*Object*): *server*オプションと[WSDLオプション]を含めたオブジェクト
  - `path` (*string*)
  - `services` (*Object*)
  - `xml` (*string*)
  - `uri` (*string*)
  - `pfx` (*string | Buffer*): PFX形式、もしくはPKCS12形式のサーバーの秘密鍵や証明書、CA証明書 (秘密鍵、証明書、CA証明書は相互排他となる)
  - `key` (*string | Buffer*): PEM形式のサーバー秘密鍵 (鍵を配列に格納可能) (必須)
  - `passphrase` (*string*): PFX、秘密鍵のパスフレーズ
  - `cert` (*string*): PEM形式のサーバー証明書鍵 (証明書を配列に格納可能) (必須)
  - `ca` (*string[]  | Buffer*): PEM形式の信頼される証明書。省略された場合、VeriSignのようなよく知られたロートCAが用いられる
  - `crl` (*string | string[]*): PEMをCRL(Certificate Revocation List)にエンコードする 
  - `ciphers` (*string*): 使用する、もしくは除外する暗号を`:`で区切って記述する。
  - `enableChunkedEncoding` (*boolean*): レスポンス中のchunked transfer encodingを操作する。クライアントの一部(Windows10のMDM登録SOAPクライアントなど)は、transfer-encodingモードに厳密で、chunkedレスポンスを受け入れ不可となる。このオプションはchunked transfer encodingを無効にする (**デフォルト**: `true`)
- `services` (*Object*)
- `wsdl` (*string*): サービスを定義するXML
- `callback` (*Function*): サーバーが初期化されたあとに実行される関数
- Returns: `Server`

**使用例**

```js
  var myService = {
      MyService: {
          MyPort: {
              MyFunction: function(args) {
                  return {
                      name: args.name
                  };
              },

              // コールバックとして非同期関数を定義する方法
              MyAsyncFunction: function(args, callback) {
                  callback({
                      name: args.name
                  });
              },

              // Promiseを用いた非同期関数を定義する方法
              MyPromiseFunction: function(args) {
                  return new Promise((resolve) => {
                    resolve({
                      name: args.name
                    });
                  });
              },

              // 送られてきたheaderをどのように受け取るか
              HeadersAwareFunction: function(args, cb, headers) {
                  return {
                      name: headers.Token
                  };
              },

              // オリジナルのリクエスト(req)も調査可能
              reallyDetailedFunction: function(args, cb, headers, req) {
                  console.log('SOAP `reallyDetailedFunction` request from ' + req.connection.remoteAddress);
                  return {
                      name: headers.Token
                  };
              }
          }
      }
  };

  var xml = require('fs').readFileSync('myservice.wsdl', 'utf8');

  //httpサーバーの例
  var server = http.createServer(function(request,response) {
      response.end('404: Not Found: ' + request.url);
  });

  server.listen(8000);
  soap.listen(server, '/wsdl', myService, xml, function(){
    console.log('server initialized');
  });

  //expressサーバーの例
  var app = express();
  //ボディ部のパーサーミドルウェアをサポート (オプション)
  app.use(bodyParser.raw({type: function(){return true;}, limit: '5mb'}));
  app.listen(8001, function(){
      //注意点: /wsdlルートはsoapモジュールによってハンドルされ、
      //他全てのルートとミドルウェアは動作を続ける
      soap.listen(app, '/wsdl', myService, xml, function(){
        console.log('server initialized');
      });
  });
```

```js
var xml = require('fs').readFileSync('myservice.wsdl', 'utf8');

soap.listen(server, {
    // サーバーオプション
    path: '/wsdl',
    services: myService,
    xml: xml,

    // WSDLオプション
    attributesKey: 'theAttrs',
    valueKey: 'theVal',
    xmlKey: 'theXml'
});
```

### サーバーログ

`log`メソッドが定義されている場合は、データと共に'received'か'replied'を指定すると呼び出される。

```js
  server = soap.listen(...)
  server.log = function(type, data) {
    // 'received'か'replied'型
  };
```

### サーバーイベント

サーバーインスタンスは以下のイベントを発行する: 

- リクエスト - メッセージを受け取る度に発行される。コールバックのシグネチャーは`function(request, methodName)`。
- レスポンス - SOAPレスポンスを送る前に発行される。コールバックのシグネチャーは`function(response, methodName)`。
- ヘッダー - SOAPヘッダーが空ではないときに発行される。コールバックのシグネチャーは`function(headers, methodName)`。

`request`、`headers`、そして専用サービスメソッドの順に呼び出されます。

### 片道呼び出しでのサーバーレスポンス

片道での(もしくは非同期の)呼び出しは、WSDLで定義された出力がないオペレーションが呼び出された際に起こる。
サーバーはオペレーションの結果を無視して、クライアントにレスポンス(デフォルトではボディ部がなしのステータスコード200)を送る。

標準的な実装のSOAPに対して、適切なクライアントの機体に応えるようなレスポンスを設定できる。
サーバーオプションに`oneWay`を渡す必要がある。また、以下のキーを使う必要がある。

- `emptyBody`: trueに設定すると、空のボディを返し、そうでないとコンテンツはなくなる。(デフォルトはfalse)
- `responseCode`: デフォルトのステータスコード200を上書きするオプション (SAPの標準に準拠した202レスポンスなど)