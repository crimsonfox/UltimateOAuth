UltimateOAuth
=============
*__非常に高機能な__ PHPのTwitterライブラリです。*

[English](https://github.com/Certainist/UltimateOAuth/blob/master/README.md)

@Version: 5.3.0  
@Author : CertaiN  
@License: BSD 2-Clause  
@GitHub : http://github.com/certainist  


## \[特長\]

- PHP **5.2.0** 以降で動作。
- **cURL** や他のライブラリに依存しない。
- UNIXとWindowsを両方サポート。
- **同期リクエスト・非同期リクエスト** を両方サポート。
- **疑似xAuth認証** をサポート。
- **アカウント作成** をサポート。
- **API制限回避** を簡単に実現可能。

## \[クラス・メソッド一覧\]

### UltimateOAuth

```php
$uo = new UltimateOAuth(
    $consumer_key = "", $consumer_secret = "", $access_token = "", $access_token_secret = ""
);

$uo->get                   ($endpoint,                  $params = array()                       );
$uo->post                  ($endpoint,                  $params = array(), $wait_response = true);
$uo->postMultipart         ($endpoint,                  $params = array(), $wait_response = true);
$uo->OAuthRequest          ($endpoint, $method = "GET", $params = array(), $wait_response = true);
$uo->OAuthRequestMultipart ($endpoint,                  $params = array(), $wait_response = true);

$uo->directGetToken ($username, $password);

$uo->getAuthorizeURL    ($force_login = false);
$uo->getAuthenticateURL ($force_login = false);
```

### UltimateOAuthMulti

```php
$uom = new UltimateOAuthMulti;

$uom->enqueue (&$uo, $method, $arg1, $arg2, $arg3, ...);
$uom->execute ($wait_processes = true, $use_cwd = false);
```

### UltimateOAuthRotate

**UltimateOAuthRotate** クラスは **__call()** メソッドを用いることにより、 **UltimateOAuth** クラスのメソッドのうち  
`get()`, `post()`, `postMultipart()`, `OAuthRequest()`, `OAuthRequestMultipart()`  
には対応しています。

```php
$uor = new UltimateOAuthRotate;

$uor->__call       ($name, $arguments);

$uor->register     ($name, $consumer_key, $consumer_secret);
$uor->login        ($username, $password, $return_array = false, $successively = false);
$uor->setCurrent   ($name);
$uor->getInstance  ($name);
$uor->getInstances ($type);
```

------------------------------------------------------------------

0. マルチリクエスト設定
-----------------------

**UltimateOAuthConfig** の定数を編集してください。

- 無料レンタルサーバーなどで `proc_open()` 関数が禁止されている場合
  
  > - `USE_PROC_OPEN`: **FALSE**
  > - `FULL_URL_TO_THIS_FILE`: このファイル自身への **絶対URL** (絶対パスでは無い)
  
- `proc_open()` 関数が問題なく使える場合
  
  > - `USE_PROC_OPEN`: **TRUE** (デフォルト)
  

1. OAuth認証
-----------------------

ユーザーに認証させるためには、次のステップを踏んでください。

**prepare.php** (ログインページに遷移させるスクリプト)

```php
<?php

// ライブラリ読み込み
require_once('UltimateOAuth.php');

// セッション開始
session_start();

// UltimateOAuthオブジェクトを新規作成してセッションに保存
$_SESSION['uo'] = new UltimateOAuth('YOUR_CONSUMER_KEY', 'YOUR_CONSUMER_SECRET');
$uo = $_SESSION['uo'];

// リクエストトークンを取得
$res = $uo->post('oauth/request_token');
if (isset($res->errors)) {
    die(sprintf('Error[%d]: %s',
        $res->errors[0]->code,
        $res->errors[0]->message
    ));
}

// Authenticateで認証するなら
$url = $uo->getAuthenticateURL();
// Authorizeで認証するなら
// $url = $uo->getAuthorizeURL();

// Twitterのログインページに遷移
header('Location: '.$url);
exit();
```

ユーザーがログインを完了すると、あらかじめ指定した **コールバックURL** に **oauth_verifier** を伴って遷移します。  
このパラメータは [Twitter Developers](https://dev.twitter.com/apps) で事前に編集しておかなければなりません。

**callback.php** (ログイン後に遷移されるスクリプト)

```php
<?php

// ライブラリ読み込み
require_once('UltimateOAuth.php');

// セッション開始
session_start();

// セッションタイムアウトチェック
if (!isset($_SESSION['uo'])) {
    die('Error[-1]: Session timeout.');
}
$uo = $_SESSION['uo'];

// oauth_verifierパラメータが存在するかチェック
if (!isset($_GET['oauth_verifier'])) {
    die('Error[-1]: No oauth_verifier');
}

// アクセストークン取得
$res = $uo->post('oauth/access_token', array(
    'oauth_verifier' => $_GET['oauth_verifier']
));
if (isset($res->errors)) {
    die(sprintf('Error[%d]: %s',
        $res->errors[0]->code,
        $res->errors[0]->message
    ));
}

// アプリケーションのメインページに遷移
header('Location: main.php');
exit();
```


**main.php** (メインページ)

```php
<?php

// ライブラリ読み込み
require_once('UltimateOAuth.php');

// セッション開始
session_start();

// セッションタイムアウトチェック
if (!isset($_SESSION['uo'])) {
    die('Error[-1]: Session timeout.');
}
$uo = $_SESSION['uo'];

// ツイートしてみましょう
$uo->post('statuses/update', 'status=テストツイート！');
```



**備考1:**  
アクセストークンとアクセストークンシークレットを既に所持している場合は、  
以下のようにするだけでログイン済みのように出来ます。

```php
<?php

require_once('UltimateOAuth.php');

$uo = new UltimateOAuth($consumer_key, $consumer_secret, $access_token, $access_token_secreet);
```

**備考2:**  
認証済みのUltimateOAuthオブジェクトを文字列として保存・復元したい場合は、  
`serialize()` 関数と `unserialize()` 関数をご利用ください。


------------------------------------------------------------------

2-1. UltimateOAuthクラス詳細
----------------------------------

### UltimateOAuth::__construct()

UltimateOAuthインスタンスを生成します。

```php
$uo = new UltimateOAuth;
$uo = new UltimateOAuth($consumer_key, $consumer_secret);
$uo = new UltimateOAuth($consumer_key, $consumer_secret, $access_token, $access_token_secret);
```

#### 引数

- *(string)* *__\[$consumer\_key\]__*  
  省略された場合は公式アプリからランダムなものが選択されます。
- *(string)* *__\[$consumer\_secret\]__*  
  省略された場合は公式アプリからランダムなものが選択されます。
- *(string)* *__\[$access\_token\]__*  
  後で認証を行わない場合はここで指定が必要です。
- *(string)* *__\[$access\_token\]__*  
  後で認証を行わない場合はここで指定が必要です。

=========================================

### UltimateOAuth::OAuthRequest()

リクエストにはこのメソッドが主体として使われます。

```php
$uo->OAuthRequest($endpoint, $method = "GET", $params = array(), $wait_response = true);
```

#### 引数

- *(string)* *__$endpoint__*  
  [APIドキュメント](https://dev.twitter.com/docs/api/1.1)をご覧ください。  
  例:  
  `statuses/update`, `1.1/statuses/update`, `https://api.twitter.com/1.1/statuses/update.json`
  
- *(string)* *__\[$method\]__*  
  `POST` または `GET` 。 大文字小文字を区別しません。 デフォルトは `GET` です。
  
- *(mixed)* *__\[$params\]__*  
  **クエリ文字列** または **連想配列** 。  
  例:
  ```php
  $params = 'status=TestTweet';
  $params = array('status' => 'TestTweet');
  ```
  ファイルをアップロードするには:
  ```php
  $params = '@image=' . $filename;
  $params = array('@image' => $filename);
  $params = array('image' => base64_encode(file_get_contents($filename)));
  ```
  
  **注意:**  
  `statuses/update_with_media` にはこのメソッドを用いることは出来ません。  
  代わりに **UltimateOAuth::postMultipart()** をご使用ください。
  
- *(boolean)* *__\[$wait\_resposne\]__*  
  デフォルトは `TRUE` です。  
  詳細は下記。

#### 返り値

- リクエストが成功した場合、デコードされたJSONデータが返されます。  
  多くの場合は **stdClass** として返されます。  
  一部のエンドポイントは **配列** として返されます。  
  例: `statuses/home_timeline`, `users/lookup`  
  一部のエンドポイントは **クエリ文字列** を取得しますが、自動的に **stdClass** に変換されます。  
  例: `oauth/request_token`, `oauth/access_token`
  
- 失敗時には **エラーオブジェクト** が返されます。  
  下記のような構造を持っています。
  
  > - *(integer)* `$response->errors[0]->code`  
  >   **HTTPステータスコード** 。 エラーコードでは無い。  
  >   既に存在しているエラーコードはHTTPステータスコードで上書きされます。  
  >   ローカルでエラーが発生した場合、多くの場合は `-1` となります。
  >   
  > - *(string)* `$response->errors[0]->message`  
  >   エラーメッセージ。
  
- `$wait_response` が `FALSE` のとき、 レスポンスを待たず直ちに `NULL` を返します。


=========================================


### UltimateOAuth::get()<br />UltimateOAuth::post()

**UltimateOAuth::OAuthRequest()** のラッパーメソッド。

```php
$uo->get($endpoint, $params);
$uo->post($endpoint, $params, $wait_response);
```


=========================================


### UltimateOAuth::postMultipart()<br />UltimateOAuth::OAuthRequestMultipart()

主に `statuses/update_with_media` に対して用いられます。  
**postMultipart()** と **OAuthRequestMultipart()** は完全に同一のものです。

```php
$uo->postMultipart($endpoint, $params, $wait_response);
$uo->OAuthRequestMultipart($endpoint, $params, $wait_response);
```


#### 引数

- *(string)* *__$endpoint__*  
  例: `statuses/update_with_media`
  
- *(string)* *__\[$params\]__*  
  例:
  ```php 
  $params = '@media[]=' . $filename;
  $params = array('@media[]' => $filename);
  $params = array('media[]' => file_get_contents($filename));
  ```
  
- *(boolean)* *__\[$wait\_response\]__*  
  **UltimateOAuth::OAuthRequest()** のものと同一です。  
  デフォルトは `TRUE` です。
  
  
#### 返り値

- **UltimateOAuth::OAuthRequest()** のものと同一です。


=========================================

  
### UltimateOAuth::directGetToken()

あたかもOAuth認証を **xAuth** 認証のように行えるメソッドです。  
これは **疑似xAuth** 認証と呼ばれます。

```php
$uo->directGetToken($username, $password);
```

#### 引数
 
- *(string)* *__$username__*  
  *スクリーンネーム* またはEメールアドレス。
  
- *(string)* *__$password__*  
  パスワード。
  
#### 返り値

- `oauth/access_token` を用いたときの **UltimateOAuth::OAuthRequest()** のものと同一です。


=========================================


### UltimateOAuth::getAuthenticateURL()<br />UltimateOAuth::getAuthorizeURL()

```php
<?php
$uo->getAuthenticateURL($force_login);
$uo->getAuthorizeURL($force_login);
```

#### 引数

- *(boolean)* *__$force\_login__*  
  既にログイン済みのユーザーを再認証させるかどうか。  
  デフォルトは **FALSE** です。
  
#### 返り値

- URL文字列。

#### 備考: *Authenticate* と *Authorize* の違いは？

|                | Authenticate  |  Authorize   |
| -------------: |:---------------:| :-----------:|
| 新規ユーザー       | Twitterに遷移 | Twitterに遷移 |
| ログイン済みユーザー   | Twitterに遷移、但しアプリケーション設定で<br /> **__Allow this application to be used to Sign in with Twitter__**, <br />にチェックを入れた場合は、設定したコールバックにすぐ遷移する  |  Twitterに遷移  |


2-2. UltimateOAuthMultiクラス詳細
--------------------------------------

複数のリクエストを **並列処理** で高速に行えます。


=========================================


### UltimateOAuthMulti::__construct()

UltimateOAuthMultiインスタンスを生成します。

```php
$uom = new UltimateOAuthMulti;
```


=========================================


### UltimateOAuthMulti::enqueue()

新しいジョブを追加します。

```php
$uom->enqueue($uo, $method, $arg1, $arg2, ...);
```


=========================================


#### 引数

- *(UltimateOAuth)* *__&$uo__*  
  **UltimateOAuth** オブジェクト。 **参照渡し** 。  
  
- *(string)* *__$method__*  
  メソッド名。 HTTPメソッドではなく **クラスメソッド** を意味します。  
  例: `post`
  
- *(mixed)* *__\[$arg1\]__*, *__\[$arg2\]__*, *__\[...\]__*  
  例: `'statuses/update', 'status=TestTweet'`

#### 注意

バイナリデータを引数に含むことは出来ません。  
その場合は **`@`** プレフィックスをつけてファイルへのパスを指定するようにしてください。  
例: `'status=test&@media[]=test.jpg'`
  
=========================================
  
  
### UltimateOAuthMulti::execute()

全てのジョブを実行します。  
実行後ジョブはすべて消去されます。

```php
$uom->execute($wait_processes, $use_cwd);
```

#### 引数

- *(boolean)* *__\[$wait\_processes\]__*  
  同期リクエストを行うかどうか。 デフォルトは `TRUE` です。
  
- *(boolean)* *__\[$use\_cwd\]__*  
  (ファイルパス指定の起点に)カレントワーキングディレクトリを使うか、このライブラリが存在するディレクトリを使うか。  
  デフォルトは `FALSE` **(UltimateOAuth.phpのディレクトリ)** です。  
  `USE_PROC_OPEN == False` のときにこのオプションは `TRUE` に出来ません。
  
#### 返り値

- 実行結果の **配列** 。


------------------------------------------------------------------


2-3. UltimateOAuthRotateクラス詳細
---------------------------------------

簡単に **API規制回避** を行えます。  
さらに下記のような、 非常に役に立つ **隠しエンドポイント** を利用できるようになります。

- `GET activity/about_me`  
  自分に関するアクティビティを取得。
- `GET activity/by_friends`  
  フレンドに関するアクティビティを取得。
- `GET statuses/:id/activity/summary`  
  指定されたツイートに関するアクティビティを取得。
- `GET conversation/show/:id`  
  指定されたツイートを含む会話を取得。
- `POST friendships/accept`  
  指定されたフォローリクエストを受理する。
- `POST friendships/deny`  
  指定されたフォローリクエストを拒否する。
- `POST friendships/accept_all`  
  全てのフォローリクエストを受理する。
- `POST account/generate`  
  アカウントを作成する。

=========================================
  
### UltimateOAuthRotate::__construct()

UltimateOAuthRotateインスタンスを生成します。

```php
$uor = new UltimateOAuthRotate;
```


=========================================

### UltimateOAuthRotate::register()

あなた自身がTwitterに登録したアプリケーションを登録します。

```php
$uor->register($name, $consumer_key, $consumer_secret);
```

#### 引数

- *(string)* *__$name__*  
  公式アプリの名前と重複しなければどんな名前でもOKです。識別用として使われます。  
  例: `my_app_01`
  
- *(string)* *__$consumer\_key__*

- *(string)* *__$consumer\_secret__*

#### 返り値

- 結果を `TRUE` か `FALSE` で返します。


=========================================


### UltimateOAuthRotate::login()

登録されたアプリケーション全てでログインします。  
このメソッドは内部的に **UltimateOAuthMulti** クラスを利用しています。

```php
$uor->login($username, $password, $return_array, $successively);
```

#### 引数と返り値
 
- *(string)* *__$username__*  
  *スクリーンネーム* またはEメールアドレス。
  
- *(string)* *__$password__*  
  パスワード。
  
- *(string)* *__\[$return\_array\]__*  
レスポンスを **配列** として返すか、全て成功したかどうかを **ブール値** で返すか。  
デフォルトは `FALSE` **(ブール値で返す)** です。
  
- *(boolean)* *__\[$successively\]__*  
逐次ログイン処理を行うか、UltimateOAuthMultiクラスを用いて並列処理で行うか。  
デフォルトは `FALSE` **(UltimateOAuthMultiクラスを利用)** です。

=========================================

### UltimateOAuthRotate::setCurrent()

**POST** リクエストで使われるアプリケーションを選択。  
GETリクエストは無関係。

```php
$uor->setCurrent($name);
```

#### 引数

- *(string)* *__$name__*  
  例: `my_app_01`

#### 返り値

- 結果を `TRUE` か `FALSE` で返します。


=========================================


### UltimateOAuthRotate::getInstance($name)

指定されたUltimateOAuthインスタンスの **クローン** を取得します。

```php
$uor->getInstance($name);
```

#### 引数

- *(string)* *__$name__*  
  例: `my_app_01`
  
#### 返り値

- **UltimateOAuth** インスタンスか `FALSE` を返します。


=========================================


### UltimateOAuthRotate::getInstances()

全てのUltimateOAuthインスタンスの **クローン** を取得します。

```php
$uor->getInstances($type);
```

#### 引数

- *(integer)* *__$type__*  
  __0__ - 全てのインスタンスを返す **(デフォルト)**  
  __1__ - 公式キーのみのインスタンスを返す  
  __2__ - オリジナルキーのみのインスタンスを返す  
  __3__ - アカウント作成用インスタンスを返す
  
#### 返り値

- UltimateOAuthオブジェクトの **配列** を返します。


=========================================


### UltimateOAuthRotate::__call()

**UltimateOAuth** のメソッドをマジックメソッド `__call()` を用いてコールします。

例:
```php
$uor->get('statuses/home_timeline');
```

------------------------------------------------------------------

3. その他の例
------------------

### アカウント作成

#### 重要

このライブラリはレスポンスヘッダーに含まれるコンテンツのうち、  
`x-twitter-new-account-oauth-access-token` と `x-twitter-new-account-oauth-secret`  
をプロパティとして返り値のプロパティに適用します。  
これには `$response->access_token` と `$response->access_token_secret` でアクセスすることが出来ます。

#### サンプル

```php
<?php

// このライブラリを読み込む
require_once('UltimateOAuth.php');

// UltimateOAuthRotateからアカウント作成が可能なインスタンスを取得
$uor = new UltimateOAuthRotate;
$base = $uor->getInstance('Twitter for Android Sign-Up');

// 自分自身をまず認証させる
$base->directGetToken('Your screen_name', 'Your password');

// アカウントを作成する
$random_string = substr(md5(mt_rand()), 0, 15);
$res = $base->post('account/generate', array(
    'name'        => 'HAHAHAHA',
    'screen_name' => $random_string,
    'password'    => 'test1234',
    'email'       => $random_string . '@examples.com',
));

// エラーチェック
if (isset($res->errors)) {
    die(sprintf('Error[%d]: %s',
        $res->errors[0]->code,
        $res->errors[0]->message
    ));
}

// テストツイート
$tmp = $uor->getInstance('Twitter for Android');
$uo = new UltimateOAuth($tmp->consumer_key, $tmp->consumer_secret, $res->access_token, $res->access_token_secret);
$uo->post('statuses/update', 'status=HAHAHAHA!!!!');
```

================================

### カーソルを追跡する

いくつかのエンドポイントでは全ての結果を得るためにカーソルを追跡する必要があります。

#### サンプル

全フォロワーのうち250人のスクリーンネームを抽選します。

```php
<?php

// このライブラリを読み込む
require_once('UltimateOAuth.php');

// インスタンス生成
$uo = new UltimateOAuth('CK', 'CS', 'AT', 'AS');

// 初期化
$cursor = '-1';
$result = array();
$screen_names = array();

// カーソルを追跡しながらフォロワーの全user_idを取得
do {
    $res = $uo->get('followers/ids', 'stringify_ids=1&cursor=' . $cursor);
    if (isset($res->errors)) {
        die(sprintf('Error[%d]: %s',
            $res->errors[0]->code,
            $res->errors[0]->message
        ));
    }
    $result += array_flip($res->ids);
    $cursor = $res->next_cursor_str;
} while ($cursor);

// 250個以内で抽選
$count = count($result);
$result = $count <= 250 ? array_keys($result) : array_rand($result, 250);

// それらのuser_idに対応するscreen_nameを取得
for ($i = 0; $i < 250; $i += 100) {
    $res = $uo->get('users/lookup', 'user_id=' . implode(',', array_slice($result, $i, 100)));
    if (isset($res->errors)) {
        die(sprintf('Error[%d]: %s',
            $res->errors[0]->code,
            $res->errors[0]->message
        ));
    }
    foreach ($res as $user) {
        $screen_names[] = $user->screen_name;
    }
}

// 出力
print_r($screen_names);
```

===============================

### URLやメンションをエンティティ情報をもとにリンクする

こちらのライブラリをお使いください。  
[TwitterText](https://github.com/Certainist/TwitterText)

