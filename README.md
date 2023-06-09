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
  - [PasswordDigestを用いたサーバーセキュリティーの例](#passworddigestを用いたサーバーセキュリティーの例)
  - [サーバー接続認証](#サーバー接続認証)
- [SOAPヘッダー](#soapヘッダー)
  - [受信SOAPヘッダー](#受信soapヘッダー)
  - [送信SOAPヘッダー](#送信soapヘッダー)
- [クライアント](#クライアント)
  - [Client.describe()](#clientdescribe)
  - [Client.setSecurity(security)](#clientsetsecuritysecurity)
  - [Client.method(args, callback, options)](#clientmethodargs-callback-options)
  - [Client.methodAsync(args, options)](#clientmethodasyncargs-options)
  - [Client.service.port.method(args, callback\[,options\[,extraHeaders\]\])](#clientserviceportmethodargs-callbackoptionsextraheaders)
  - [名前空間prefixのオーバーライド](#名前空間prefixのオーバーライド)
  - [Client.lastRequest](#clientlastrequest)
  - [Client.setEndpoint(url)](#clientsetendpointurl)
  - [クライアントイベント](#クライアントイベント)
- [WSDL](#wsdl)
- [WSDL.constructor(wsdl, baseURL, options)](#wsdlconstructorwsdl-baseurl-options)
  - [wsdl.xmlToObject(xml)](#wsdlxmltoobjectxml)
  - [wsdl.objectToXML(object, typeName, namespacePrefix, namespaceURI, ...)](#wsdlobjecttoxmlobject-typename-namespaceprefix-namespaceuri-)
- [セキュリティ](#セキュリティ)
  - [BasicAuthSecurity](#basicauthsecurity)
  - [BearerSecurity](#bearersecurity)
  - [ClientSSLSecurity](#clientsslsecurity)
  - [ClientSSLSecurityPFX](#clientsslsecuritypfx)
  - [WSSecurity](#wssecurity)

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

### PasswordDigestを用いたサーバーセキュリティーの例

`server.authenticate`が定義されてない場合、認証は行われない。

非同期認証 :

```js
  server = soap.listen(...)
  server.authenticate = function(security, callback) {
    var created, nonce, password, user, token;
    token = security.UsernameToken, user = token.Username,
            password = token.Password, nonce = token.Nonce, created = token.Created;

    myDatabase.getUser(user, function (err, dbUser) {
      if (err || !dbUser) {
        callback(false);
        return;
      }

      callback(password === soap.passwordDigest(nonce, created, dbUser.password));
    });
  };
```

同期認証 : 

```js
  server = soap.listen(...)
  server.authenticate = function(security) {
    var created, nonce, password, user, token;
    token = security.UsernameToken, user = token.Username,
            password = token.Password, nonce = token.Nonce, created = token.Created;
    return user === 'user' && password === soap.passwordDigest(nonce, created, 'password');
  };
```

### サーバー接続認証

`server.authorizeConnection`はSOAPサービスのメソッドより先に呼び出される。
`server.authorizeConnection`が定義され`false`を返す場合、着信した接続は終了する。

```js
  server = soap.listen(...)
  server.authorizeConnection = function(req) {
    return true; // or false
  };
```

## SOAPヘッダー

### 受信SOAPヘッダー

3つめの引数を与えることによって、サービスメソッドはSOAPヘッダーとみなすことができる。

```js
  {
      HeadersAwareFunction: function(args, cb, headers) {
          return {
              name: headers.Token
          };
      }
  }
```

また、'headers'イベントを登録することも可能である。
このイベントは、SOAPヘッダーが空でないときに、サービスメソッドが呼び出される前にトリガーされる。

```js
  server = soap.listen(...)
  server.on('headers', function(headers, methodName) {
    // サービスメソッドが渡される前であれば、
    // ヘッダーの値を変えることは可能である。
    // また、SOAP Faultを投げることも可能である。
  });
```

最初のパラメーターはヘッダーオブジェクトで、2つめのパラメーターは呼び出されるSOAPメソッドの名前となる。
(この場合、メソッドによってヘッダーの扱いを変える必要がある)

### 送信SOAPヘッダー

クライアントとサーバーは送信されるものに付与されるSOAPヘッダーを定義することができる。
ヘッダーを管理するのに、以下のようなメソッドがある。

**addSoapHeader(soapHeader[, name, namespace, xmlns])**

SOAPヘッダーにsoap:Headerノードを追加する。

**パラーメーター**

- `soapHeader` (*Object({rootName: {name: 'value'}})*): 厳密なxml-stringか関数 (サーバー限定)

サーバー限定ではあるが、`soapHeader`はリクエスト中の情報から動的にヘッダーを生成することができる関数となることができる。
この関数は、受信したリクエストごとに以下の引数で呼び出される。

- `methodName`: リクエストメソッドの名前
- `args`: リクエストの引数
- `headers`: リクエストのヘッダー
- `req`: オリジナルのリクエストオブジェクト

関数の返り値は、リクエストに対するレスポンスの送信ヘッダーの代わりとして、オブジェクト({rootName: {name: 'value'}})か厳密なxml-stringでなければならない。

例:

```js
  server = soap.listen(...);
  server.addSoapHeader(function(methodName, args, headers, req) {
    console.log('Adding headers for method', methodName);
    return {
      MyHeader1: args.SomeValueFromArgs,
      MyHeader2: headers.SomeRequestHeader
    };
    // または "<MyHeader1>値</MyHeader1>" と指定できる
  });
```

**返り値**

ヘッダーが挿入されるインデックス

**第一引数がオブジェクトのときオプションのパラメーターは :**

- `name`: 未知のパラメーター(空の文字列でもよい)
- `namespace`: xml名前空間のプレフィックス
- `xmlns`: URI

**changeSoapHeader(index, soapHeader[, name, namespacem xmlns])**

すでにあるSOAPヘッダーを変更する

**パラメーター**

- `index`: ヘッダーのインデックスを新しく与えられた値で置き換える
- `soapHeader`: オブジェクト({rootName: {name: 'value'}})や厳密なxml-string、関数 (サーバー限定)

`soapHeader`に関数を渡す方法は`addSoapHeader`を参照

**getSoapHeaders()**

全ての定義済みヘッダーを返す

**clearSoapHeaders()**

全ての定義済みヘッダーを除外する

## クライアント

`Client`のインスタンスは`soap.createClient`のコールバックに渡され、SOAPサービスでメソッドを実行するのに使用される。

### Client.describe()

サービスやポート、メソッドをJavaScriptのオブジェクトとして登録する

```js
  client.describe() // returns
    {
      MyService: {
        MyPort: {
          MyFunction: {
            input: {
              name: 'string'
            }
          }
        }
      }
    }
```

### Client.setSecurity(security)

使用するセキュリティプロトコルを指定する

[セキュリティ](#セキュリティ)の使用例を参照

### Client.method(args, callback, options)

SOAPサービスのメソッドを呼び出す

- `args` (*Object*): SOAPボディ部内にXMLドキュメントを生成する引数
- `callback` (*Function*)
- `options` (*Object*): WSDLリクエストにあるリクエストモジュールのオプションを設定する。
  デフォルトのリクエストモジュールを使用している場合は、[Request Config | Axios Docs](https://axios-http.com/docs/req_config)を参照。
  以下のような`node-soap`で追加されたオプション:
  - `forever` (*boolean*): keep-aliveの接続とプールを有効にする
  - `attachments` (*Array*): アタッチメントオブジェクトの配列。リクエストをMTOM(*headers['Content-Type']='multipart/related; type="application/xop+xml"; start=...'*)に変換する。
  ```js
  [{
      mimetype: content mimetype,
      contentId: part id,
      name: file name,
      body: binary data
   },
  ...
  ]
  ```
  - `forceMTOM` (*boolean*): アタッチメントがない場合でも、MOTMとしてリクエストを送信する
  - `forceGzip` (*boolean*): gzipへのエンコードを強制する (**デフォルト**: `false`)

使用例
```js
  client.MyFunction({name: 'value'}, function(err, result, rawResponse, soapHeader, rawRequest) {
      // resultはJavaScriptオブジェクト
      // rawResponseはxmlレスポンス
      // soapHeaderはJavaScriptオブジェクトとしてのレスポンスSOAPヘッダー
      // rawRequestは無加工のxmlリクエスト
  })
```

### Client.methodAsync(args, options)

SOAPサービスのメソッドを呼び出す

- `args` (*Object*): SOAPボディ部内にXMLドキュメントを生成する引数
- `options` (*Object*): [Client.method(args, callback, options)](#clientmethodargs-callback-options)を参照

使用例
```js
  client.MyFunctionAsync({name: 'value'}).then((result) => {
    // resultは、resultやrawResponse、soapheader、rawRequestを含んだJavaScriptオブジェクト
    // resultはJavaScriptオブジェクト
    // rawResponseはxmlレスポンス
    // soapHeaderはJavaScriptオブジェクトとしてのレスポンスSOAPヘッダー
    // rawRequestは無加工のxmlリクエスト
  })
```

`args`のJSON例

上の`{name: 'value'}`を引数として使っている使用例では、生成されるSOAPメッセージは以下のようになる:

```json
<?xml version="1.0" encoding="utf-8"?>
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
   <soapenv:Body>
      <Request xmlns="http://www.example.com/v1">
          <name>value</name>
      </Request>
   </soapenv:Body>
</soapenv:Envelope>
```

SOAPメッセージ中の"Request"要素は、WSDLを由来とする。`args`要素中に名前空間prefixがないときは、デフォルト値が使用される。それ以外の場合は、必要に応じて要素の名前に名前空間prefixを加える必要がある。

現在、JSONに引数を渡すとき、XMLの仕様では許容されているにもかかわらず要素に子要素とテキスト値の両方を渡すことはできない。

`args`のXML例

XMLを構成するためのJSON`args`に含まれる個々の要素の代わりに、整形されたXMLを渡すこともできる。その場合、XML宣言(`<?xml version="1.0" encoding="UTF-8"?>`)やドキュメント型宣言(`<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Frameset//EN" "http://www.w3.org/TR/html4/frameset.dtd">`)を含めてはいけない。

```js
 var args = { _xml: "<ns1:MyRootElement xmlns:ns1="http://www.example.com/v1/ns1">
                        <ChildElement>elementvalue</ChildElement>
                     </ns1:MyRootElement>"
            };
```

この場合、名前空間とprefixを指定する必要がある。上記の"`args`のJSON例"の用にWSDLから生成される要素は使用されず、"Request"要素が自動的に生成される。

### Client.service.port.method(args, callback[,options[,extraHeaders]])

指定のサービスとポートを使用したメソッドを呼び出す

- `args` (*Object*): SOAPボディ部中にあるXMLドキュメントを生成する引数
- `callback` (*Function*)
- `options` (*Object*): [Client.method(args, callback, options)](#clientmethodargs-callback-options)を参照
- `extraHeaders` (*Object*): WSDLリクエストのHTTPヘッダーの集合

使用例

```js
  client.MyService.MyPort.MyFunction({name: 'value'}, function(err, result) {
      // resultはJavaScriptオブジェクト
  })
```

**オプション(任意)**

- リクエストモジュールが受け付ける任意のオプションを受け付ける。[ここ](https://github.com/request/request)を参照
- 以下の例では、リクエストのタイムアウトを5秒に設定している:
```js
  client.MyService.MyPort.MyFunction({name: 'value'}, function(err, result) {
      // resultはJavaScriptオブジェクト
  }, {timeout: 5000})
```
- リクエストが渡されてからの経過時間も計測可能である
```js
  client.MyService.MyPort.MyFunction({name: 'value'}, function(err, result) {
      // client.lastElapsedTime - 最後のリクエストのミリ秒での経過時間
  }, {time: true})
```
- [Fiddler](https://www.telerik.com/fiddler)や[Betwixt](https://github.com/kdzwinel/betwixt)のようなデバッグプロキシ経由でもSOAPリクエストを渡すことができる
```js
  client.MyService.MyPort.MyFunction({name: 'value'}, function(err, result) {
      // client.lastElapsedTime - 最後のリクエストのミリ秒での経過時間
  }, {proxy: 'http://localhost:8888'})
```
- 呼び出す前にxmlを変更することもできる
```js
  client.MyService.MyPort.MyFunction({name: 'value'}, function(err, result) {
      // client.lastElapsedTime - 最後のリクエストのミリ秒での経過時間
  }, {postProcess: function(_xml) {
    return _xml.replace('text', 'newtext');
  }})
```

**追加のヘッダー(任意)**

オブジェクトプロパティーはリクエストの追加のHTTPヘッダーを定義する

- カスタムUser-Agentを追加
```js
client.addHttpHeader('User-Agent', `CustomUserAgent`);
```

**callback-lastパターンを使用した代替メソッドの呼び出し**

メソッド呼び出しのシグネチャーをnodeの標準的なcallback-lastパターンに合わせ、メソッド呼び出しのPromise化するイベントを可能にするために、以下のメソッドシグネチャーをサポートしている:
```js
client.MyService.MyPort.MyFunction({name: 'value'}, options, function (err, result) {
  // resultはJavaScriptオブジェクト
})

client.MyService.MyPort.MyFunction({name: 'value'}, options, extraHeaders, function (err, result) {
  // resultはJavaScriptオブジェクト
})
```

### 名前空間prefixのオーバーライド

`node-soap`は名前空間について問題を解決している最中である。リクエストボディ中のある要素に誤った名前空間prefixが与えられていた場合、オブジェクト中の名前にprefixを加えることができる。

すなわち:

```js
  client.MyService.MyPort.MyFunction({'ns1:name': 'value'}, function(err, result) {
      // 名前空間がどうあるべきかに関係なく、リクエストボディには`<ns:name>`が含まれる
  }, {timeout: 5000})
```

- パラメーターの名前空間prefixを取り除く場合
```js
  client.MyService.MyPort.MyFunction({':name': 'value'}, function(err, result) {
      // 名前空間がどうあるべきかに関係なく、リクエストボディには`<name>`が含まれる
  }, {timeout: 5000})
```

### Client.lastRequest

クライアントに記録された最後のSOAPリクエストを含むプロパティー

### Client.setEndpoint(url)

SOAPサービスのエンドポイントアドレスを書き換える

### クライアントイベント

クライアントインスタンスは以下のイベントを発する。

**リクエスト**

リクエストが送信される前に発せられる。イベントハンドラーのシグネチャーは`(xml, eid)`となる。

- xml : ヘッダーを含むSOAPリクエスト(エンベロープ)全体
- eid : exchange id

**メッセージ**

リクエストが送信される前に発せられるが、ボディのみイベントハンドラーに渡される。
SOAPヘッダーの/storeをログに残したくない場合に有用である。イベントハンドラーのシグネチャーは`(message, eid)`となる。

- message : SOAPのボディコンテンツ
- eid : exchange id

**SOAPエラー**

間違ったレスポンスを受け取ったときに発せられる。エラーのログをグローバルに取得したいときに有用である。
イベントハンドラーのシグネチャーは`(error, eid)`となる。

- error : レスポンスも含むエラーオブジェクト
- eid : exchange id

**レスポンス**

レスポンスを受け取った後に発せられる。成功・エラー問わず全てのレスポンスに対して発せられる。
イベントハンドラーのシグネチャーは`(body, response, eid)`となる。

- body : SOAPレスポンスのボディ
- response : `IncomigMessage`レスポンスオブジェクトの全体
- eid : exchange id

'exchange'はリクエストとレスポンスの組み合わせである。全てのイベントで、イベントハンドラーはexchange idを受け取る。
exchange idはリクエストとレスポンスで同じものであり、レスポンスを受け取った時に組み合わせとなるリクエストがわかるものとなっている。

exchange idは、デフォルトではnode-uuidを使用して生成されるが、クライアント呼び出しの中で自分で設定することもできる。

使用例:
```js
  client.MyService.MyPort.MyFunction(args , function(err, result) {

  }, {exchangeId: myExchangeId})
```

## WSDL

WSDLインスタンスはSOAP呼び出しなしでメッセージをまとめたいときにも直接インスタンス化することができる。
WSDLがサービス(Windows Communication Foundation SOAP web servicesなど)のバインディングを含んでいないときに使用できる。

## WSDL.constructor(wsdl, baseURL, options)

WSDLコンテンツかWSDLへのURLからWSDLインスタンスを構築する。

**パラメーター**

- wsdl: WSDLかWSDLへのURL
- baseURL: SOAP APIのベースURL
- options: オプション(詳細はソースを確認)、デフォルトは`{}`

### wsdl.xmlToObject(xml)

xmlからオブジェクトへ変換する

**パラメーター**

- xml: 変換するSOAPレスポンス(xml)

**返り値**

xmlのオブジェクトをキーとして含むオブジェクト

### wsdl.objectToXML(object, typeName, namespacePrefix, namespaceURI, ...)

オブジェクトをxmlへ変換する

**パラメーター**

- object: 変換するオブジェクト
- typeName: オブジェクトの型(wsdlごと)
- namespacePrefix: 名前空間のprefix
- namespaceURI: 名前空間のURI

**返り値**

オブジェクトのXML表現

使用例:
```js
// 実際に使用する際の概要
import { AxiosInstance } from 'axios';
import { WSDL } from 'soap';
import { IProspectType } from './types';

// WSDLの文字列
const WSDL_CONTENT = "...";

const httpClient: AxiosInstance = /* ... インスタンス化 ... */;
const url = 'http://example.org/SoapService.svc';

const wsdl = new WSDL(WSDL_CONTENT, baseURL, {});

async function sampleGetCall(): IProspectType | undefined {
    const res = await httpClient.get(`${baseURL}/GetProspect`);

    const object = wsdl.xmlToObject(res.data);

    if (!object.ProspectType) {
      // レスポンスに期待された型がないとき
      return undefined;
    }
    // オプションで、いくつかのフィールドのアンラップとデフォルトの設定を行う
    // オブジェクトが期待されるプロトタイプと合致することを保証する
    // 最後に結果をキャストして返す
    return object.ProspectType as IProspectType;
}

async function samplePostCall(prospect: IProspectType) {
  // objectToXML(object, typeName, namespacePrefix, namespaceURI, ...)
  const objectBody = wsdl.objectToXML(obj, 'ProspectType', '', '');
  const data = `<?xml version="1.0" ?>${body}`;

  const res = await httpClient.post(`${baseURL}/ProcessProspect`, data);
  // Optionally, deserialize request and return response status.
}
```

## セキュリティ

`node-soap`はデフォルトでいくつかセキュリティプロトコルが備わっていて、追加も簡単に行える。
インターフェースはシンプルなものとなっている。
各プロトコルは以下のオプションメソッドを定義している。

- `addOptions(options)`: 最終的に直接`request`に渡されるoptions argsを受け取るメソッド
- `addHeaders(headers)`: 新しいヘッダーに加えるためのHTTPヘッダーと引数を受け取るメソッド
- `toXML()`: SOAPヘッダーに加えられたXMLを返すメソッド。`postProcess`が定義されている場合は実行されない。
- `postProcess(xml, envelopeKey)`: リクエストXMLとエンベロープキーを組み合わせたものを受け取り、処理されたXMLを返すメソッド。
  `options.postProcess`の前に実行される。

### BasicAuthSecurity

```js
client.setSecurity(new soap.BasicAuthSecurity('username', 'password'));
```

### BearerSecurity

```js
client.setSecurity(new soap.BearerSecurity('token'));
```

### ClientSSLSecurity

メモ: プロトコルの動作に問題がある場合、デフォルトのリクエストオプションとして以下のオプションがコンストラクターに渡されていると考えられる。

- `rejectUnauthorized: false`
- `strictSSL: false`
- `secureOptions: constants.SSL_OP_NO_TLSv1_2` (nodeのバージョンが10.0以上の時に必要であると思われる)

`forever: true`オプションを使えば、TLSセッションの再利用ができる。

```js
client.setSecurity(new soap.ClientSSLSecurity(
                '/path/to/key',
                'path/to/cert',
                '/path/to/ca-cert',  /* もしくはバッファーの配列: [fs.readFileSync('/path/to/ca-cert/1', 'utf8'),
                'fs.readFileSync('/path/to/ca-cert/2', 'utf8')], */
                {   /* デフォルトリクエストオプションは以下の通り */
                    // strictSSL: true,
                    // rejectUnauthorized: false,
                    // hostname: 'some-hostname'
                    // secureOptions: constants.SSL_OP_NO_TLSv1_2,
                    // forever: true,
                },
      ));
```

### ClientSSLSecurityPFX

メモ: プロトコルの動作に問題がある場合、デフォルトのリクエストオプションとして以下のオプションがコンストラクターに渡されていると考えられる。

- `rejectUnauthorized: false`
- `strictSSL: false`
- `secureOptions: constants.SSL_OP_NO_TLSv1_2` (nodeのバージョンが10.0以上の時に必要であると思われる)

`forever: true`オプションを使えば、TLSセッションの再利用ができる。

```js
client.setSecurity(new soap.ClientSSLSecurityPFX(
                '/path/to/pfx/cert', // もしくはバッファー: [fs.readFileSync('/path/to/pfx/cert', 'utf8'),
                'path/to/optional/passphrase',
                {   /* デフォルトリクエストオプションは以下の通り */
                    // strictSSL: true,
                    // rejectUnauthorized: false,
                    // hostname: 'some-hostname'
                    // secureOptions: constants.SSL_OP_NO_TLSv1_2,
                    // forever: true,
                },
      ));
```

### WSSecurity

`WSSecurity`はWS-Securityを埋め込む。ユーザーネームトークンとPasswordText/PasswordDigestに対応。

```js
  var options = {
    hasNonce: true,
    actor: 'actor'
  };
  var wsSecurity = new soap.WSSecurity('username', 'password', options)
  client.setSecurity(wsSecurity);
```

`options`オブジェクトはオプションで、以下のプロパティーを含めることができる。

- `passwordType`: PasswordTextかPasswordDigest (デフォルトは`'PasswordText'`)
- `hasTimeStamp`: タイムスタンプ要素を追加する (デフォルトは`true`)
- `hasTokenCreated`: Created要素を追加する (デフォルトは`true`)
- `hasNonce`: Nonce要素を追加する (デフォルトは`false`)
- `mustUnderstand`: セキュリティタグにmustUnderstand=1属性を追加する (デフォルトは`false`)
- `actor`: 設定すると、セキュリティタグにActor要素と値を追加する (デフォルトは`''`)