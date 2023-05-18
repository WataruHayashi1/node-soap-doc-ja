# node-soap-doc-ja <!-- omit in toc -->

> [Node.js用SOAP通信モジュールnode-soapののREADME](https://github.com/vpulim/node-soap/blob/master/Readme.md)の日本語訳です

> node.js用SOAPクライアント・サーバー

このモジュールを使用することで、WebサービスへのSOAP通信やSOAPサービスを提供するサーバーの構築が可能になる。


- [機能](#機能)
- [インストール](#インストール)
- [サポート](#サポート)
- [モジュール](#モジュール)
  - [soap.createClient(url\[, options\], callback)](#soapcreateclienturl-options-callback)

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

WSDL urlからSOAPクライアントを作成する。ローカルファイルパスにも対応。

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