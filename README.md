# carnation server API reference

## 1 OAuth2 アクセストークン取得

carnation API の呼び出しを行うためには、OAuth2 access token が必要です.
[RFC6749 The OAuth 2.0 Authorization Framework](https://tools.ietf.org/html/rfc6749)


carnation server では、カメラアプリからの呼び出しに用いる token を user token, STB viewer からのAPI呼び出しに用いる token を viewer token と呼んでいます.

user token は [RFC6749 Client Password](https://tools.ietf.org/html/rfc6749#section-2.3.1) の方式で取得します. 

viewer token は、[RFC6479 Client Credentials Grant](https://tools.ietf.org/html/rfc6749#section-4.4) の方式で取得します.


現状の実装では、Access Token は、24時間で無効になります.
```
   self.expires_at = now + 1 * (3600 * 24)  # 1 days for now
```
無効になった場合は、再取得もしくは refresh token が必要です.
なお refresh token を行うことができるのは、user token のみです.
viewer token の取得に際しては、パスワード入力などのユーザーインタラクションがないので、必要に応じてトークンを単に再取得すれば良いという考え方です.

取得したトークンは、全ての carnation server API を呼び出す際に、Request Header に次のように設定する必要があります. [RFC6750 The OAuth 2.0 Authorization Framework: Bearer Token Usage](https://tools.ietf.org/html/rfc6750)

```
Authorization:Bearer a233dc78b84983da434344235a65fc84
```

### 1.1 Authentication header 

user token, viewer token どちらもの取得のときも、[RFC2617] (https://tools.ietf.org/html/rfc2617) で規定されている Basic Authentication header を設定する必要があります.
RFC2617 で用いられている Authentication header に設定する userid, password はそれぞれ carnation server の内部的な値(というか呼び方) では次のものに相当します.


| rfc2617       | carnation     |
|---------------|---------------|
| userid        | appid         |
| password      | secret        |


### 1.2 get user token

| method        | end point     |
|---------------|---------------|
| PUT           | /token        |

Authentication header にセットする appid, secret はサービス運営者からアサインされるもので、アプリケーションのタイプやバージョンごとに異なるものを持つことができます（共通でも構わない）.
そのアプリケーションが carnation server にアクセスする権限を持つかどうか carnation server が判断するためのものです.
アプリケーション開発者が 個別のアプリケーションに搭載します.

**parameters**

新規取得

| name        |  value             |  mandatory? | default value  |
|-------------|--------------------|-------------|----------------|
| grant_type  |  "password"        |    yes     | N/A            |
| username    |  ユーザーの email  |    yes     | N/A            |
| password    |  ユーザーパスワード|    yes     | N/A            |

refresh_token

| name          |  value             |  mandatory? | default value  |
|---------------|--------------------|-------------|----------------|
| grant_type    |  "refresh_token"   |    yes      | N/A            |
| refresh_token |  refresh token     |    yes      | N/A            |

**output**

| name          |  value          | 
|---------------|-----------------|
| access_token  |  user token |
| refresh_token |  refresh token      | 
| token_type    |  "bearer"       |
| user_id       | user id |
| expires_in    | expire するまでの秒|
| scope         | スコープ文字列（現状使われてない)|

**example**

```
{
  "access_token":"7fb0b857382671892b77f5d4048589b0",
  "refresh_token":"9d9fba1cb49c124b817c1dfe4ed9307a",
  "token_type":"bearer",
  "user_id":4,
  "expires_in":86400,
  "scope":"read create delete"
}
```

### 1.3 get viewer token

| method        | end point     |
|---------------|---------------|
| PUT           | /token    |

Authentication header にセットする appid, secret は、それぞれのSTBで独自な値を持ちます.
STBの出荷時になんらかの手段で端末に搭載されます.


**parameters**

| name          |  value                |  mandatory?   | default value  |
|---------------|-----------------------|---------------|----------------|
| grant_type    |  "client_credential"  |    yes        | N/A            |

**output**

| name          |  value          | 
|---------------|-----------------|
| access_token  |  user token |
| token_type    |  "bearer"       |
| user_id       | view id |
| expires_in    | expire するまでの秒|
| scope         | スコープ文字列（現状使われてない)|

**example**
```
{
  "access_token":"8d69445035b989dfd50f467d5aab5afa",
  "token_type":"bearer",
  "viewer_id":1,
  "expires_in":86400,
  "scope":"read like"
}
```

## 2 user management

### 2.1 create new user


| method        | end point     | required token   |
|---------------|---------------|------------------|
| POST          | /api/v1/user  | signup |

ユーザーを作成します.
この API を呼び出すためには、admin もしくは signup の権限が必要です.
なお現状のこの APIの実装は簡易実装なので今後改良が必要です.


**parameters**

| name          |  value          |  mandatory?   | default value  |
|---------------|-----------------|---------------|----------------|
| email         |  メールアドレス |    yes        | N/A            |
| password      |  パスワード     |    yes        | N/A            |

**output**

| name          |  value          | 
|---------------|-----------------|
| id         |  user_id |
| email      |  user email      | 
| name      |  user name (メールアドレスから適当に生成される)     |
| password      |  パスワード     | 

**error**

| code    |  message                        | description |
|---------|---------------------------------|---------------------------|
| 400     |  access denied                  | 権限がない|
| 400     |  invalid parameter  email       | emailが指定されてない|
| 400     |  invalid parameter  password    | password が指定されてない|
| 400     |  sepcified email already exists | 同じ email のユーザーが既にある|

**example**

```
{
  "id": 7,
  "email": "testtest@chikaku.co.jp",
  "name": "testtest_chikaku.co.jp",
  "password": "abc"
}
```

### 2.2 change user attribute

| method        | end point     | required token |
|---------------|---------------|----------------|
| POST          | /api/v1/attributes  | user(of user_id)|

ユーザーの email, name, password を変更します
この API を呼び出すためには、admin もしくは user_id のユーザーであることが必要です.
パラメータで指定したもののみが指定した値に変更されます.

**parameters**

| name          |  value              |  mandatory?   | default value  |
|---------------|---------------------|---------------|----------------|
| user_id       |  user id            |   yes         | N/A            |
| email         |  user email address |    no         | N/A            |
| name          |  user name          |    no         | N/A            |
| password      |  user password      |    no         | N/A            |

**output**

| name          |  value          | 
|---------------|-----------------|
| id         |  user_id |
| email      |  user email      | 
| name      |  user name |
| password      |  user password |

**error**

| code    |  message                        | description |
|---------|---------------------------------|---------------------------|
| 400     |  access denied                  | 権限がない |
| 400     |  token invalid                  | access token 不正   |
| 400     |  no such user                   | user_id 不正 |

**example**

```
{
  "id": 4,
  "email": "test01@chikaku.com",
  "name": "harahara",
  "password": ""
}
```

### 2.3 delete user

| method        | end point     | required token |
|---------------|---------------|----------------|
| DELETE        | /api/v1/user  | admin |

ユーザーを削除します
この API を呼び出すためには、admin 権限が必要です.
user を削除しても S3 storage のファイルは削除されません.(未実装です)

**parameters**

| name          |  value          |  mandatory?   | default value  |
|---------------|-----------------|---------------|----------------|
| user_id       |  user id        |   yes         | N/A            |

**output**

| name      |  value          | 
|-----------|-----------------|
| id        |  user_id        |
| email     |  user email     | 
| name      |  user name      |

**error**

| code    |  message                        | description |
|---------|---------------------------------|---------------------------|
| 400     |  access denied                  | 権限がない |
| 400     |  user not found                 | user_id 不正 |

**example**

```
{
  "id": 4,
  "email": "test01@chikaku.com",
  "name": "harahara",
}
```

### 2.4 get user info

| method        | end point     | required token   |
|---------------|---------------|------------------|
| GET           | /api/v1/user  | user(of user_id) |

ユーザー情報を取得します

**parameters**

| name          |  value        | mandatory? | default value  |
|---------------|---------------|------------|----------------|
| user_id       |  user id      |  yes       | N/A            |

**output**

| name                |  value          | 
|---------------------|-----------------|
| id                  |  user_id        |
| email               |  user email     | 
| name                |  user name      |
| role                |  admin:1, default:2, signup:3, 通常:100|
| status              |  1 (固定)       |
| timezone            |  timezone(数値) |
| viewers             | このユーザーが所有している viewerの 一覧（配列） |
| groups              |  このユーザーが所有している group の一覧（配列） |
| belong_to_groups    |  このユーザーが所属しているグループの一覧（配列）|

**error**

| code    |  message                        | description |
|---------|---------------------------------|---------------------------|
| 400     |  access denied                  | token 不正、権限がない |
| 404     |  user not found                 | user_id 不正 |

**example**

```
{
  "id": 4,
  "email": "test01@chikaku.com",
  "name": "harahara",
  "role": 100,
  "status": 1,
  "timezone": 9,
  "viewers": [
    {
      "id": 1,
      "name": "test01viewer",
      "status": 1,
      "user_id": 4,
      "client_id": 5,
      "phone_number": null,
      "postal_code": null,
      "address": null,
      "valid_through": 1444636922,
      "created_at": 1413100922,
      "timezone": 9,
      "credentials": {
        "id": 5,
        "appid": "6052d5885f9c2a12c09ef90f815225d3",
        "secret": "f6af879a7db8bfbe183e08c1a68e9035",
        "created_at": 1413100922
      }
    }
  ],
  "groups": [
    {
      "id": 1,
      "name": "test01group",
      "description": null,
      "user_id": 4,
      "created_at": 1413100922
    }
  ],
  "belong_to_groups": [
    {
      "id": 1,
      "name": "test01group",
      "description": null,
      "user_id": 4,
      "created_at": 1413100922
    }
  ]
}
```

### 2.5 get user id by email

| method        | end point              | required token |
|---------------|------------------------|---------------|
| GET           | /api/v1/user_by_email  | user,viewer   |

指定された email を持つユーザーの id を検索します

**parameters**

| name          |  value        | mandatory? | default value  |
|---------------|---------------|------------|----------------|
| email         |  user email   |  yes       | N/A            |

**output**

| name                |  value          | 
|---------------------|-----------------|
| user_id             |  user id        |

**error**

| code    |  message                        | description |
|---------|---------------------------------|---------------------------|
| 404     |  user not found                 | 指定された email の userは無い |

**example**
```
{
  "user_id": 4
}
```

## 3 item management

**アイテムアップロードの流れ**

<a href="upload_sequence.svg">
<img src="upload_sequence.png" alt="upload_sequence"/>
</a>

1. カメラアプリより、/api/v1/initiate を呼んでアップロードを開始します
2. carnation server から database へアクセスして、新規アイテムを生成します
3. カメラアプリへ、item_id, item_url を返します
4. カメラアプリは、アイテム（画像・動画）のファイルを item_url を使って S3 へアップロードします
5. カメラアプリは アップロードの結果：HTTP PUT method のレスポンスを確認します
6. カメラアプリは carnation server にアップロード完了を通知します (/api/v1/activate)
7. carnation server は redis の queue に item_id を追加します
8. カメラアプリへ activate の結果を通知します.この時点ではまだ派生ファイルは生成されてません. item のstatus は、0(initiate) となります

**派生ファイル生成の流れ**

上記のアップロードの流れとはまったく独立に、非同期に動いている派生ファイル生成用のプロセスが resque というモジュールを使って動いています.
この resque process は、負荷状況に応じて複数のプロセスを複数のマシン上で起動しておくことが可能です。

<a href="create_derivatives.svg">
<img src="create_derivatives.png" alt="create_derivatives"/>
</a>

1. resque process は定期的に新規の item_id を redis queue へ問い合わせます. もし上記アイテムアップロードにより、新規 item_id が登録されている場合は、その item_id を返します
2. resque process は s3 storage よりアップロードされたオリジナルのファイルを取得します
3. thumbnail image, 動画スクリーンショットなど、派生ファイルを生成します
4. 生成した派生ファイルを S3 へアップロードします
5. item のステータスを 0(initiate) から 1(active) へ変更します

### 3.1 initiate item upload

| method        | end point              | required token |
|---------------|------------------------|----------------|
| POST          | /api/v1/item/initiate  | user           |

アイテムアップロードを開始します.

**parameters**

| name        |  value                     | mandatory? | default value  |
|-------------|----------------------------|------------|----------------|
| user_id     |  user id                   |  yes       | N/A            |
| item_id     |  item id                   |  no        | N/A            |
| file_hash   |  item id                   |  no        | ""             |
| file_info   |  text                      |  no        | ""             |
| timezone    |  timezone number e.g. 9    |  no        |user's timezone |
| shot_at     |  [epoch time](http://en.wikipedia.org/wiki/Unix_time) | no | initiate した日時|
| title       |  text                      |  no        | ""             |
| description |  text                      |  no        | ""             |
| extension   | ".jpg",".png",".mp4",".mov"| yes        | N/A            |

- fish_hash はファイルの sha1sum を16進表記した文字列です. Unix系OSの sha1sum コマンドの出力と同じです. 同一のファイルが既にアップロード済みかどうかを、file_hash を比較することで検出します.
```
$ echo -n "hello" | sha1sum
aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d
```
- file_info はアプリで自由に設定する文字列です たとえばアプリのローカルなファイルのパス情報などを入れることを想定しています
- timezone を指定しない場合は user の timezone が使われます
- 動画ファイルなど 撮影日時がファイルの exif情報として取得できない場合は、shot_at を指定して明示的に撮影日時を設定してください. shot_at を指定しなくて、かつファイルの exif 情報がない場合は、shot_at は initiate を呼び出した日時が設定されます.


**output**

| name                |  value          | 
|---------------------|-----------------|
| item_id             |  item id        |
| status              |  item status(0) |
| url                 |  url to put     |

- カメラアプリは取得した url を用いて HTTP PUT メソッドによりファイルのアップロードを行います.
- アップロードが完了したら、次項の /api/v1/item/activte を呼び出します.

- url は 28800秒(8時間）有効です.
- 期限が切れる前にアップロードを完了させる必要があります.
-もし期限が切れてしまった場合など、再度 url を取得する必要がある場合は、item_id を指定して initiate を呼び出します。


**error**

| code    |  message                        | description |
|---------|---------------------------------|---------------------------|
| 400     |  user_id invalid                | user_id 不正 |
| 400     |  invalid token                | access token 不正 |
| 400     |  access denied                  | 権限不足 |
| 400     |  item_id invalid                | item_id が指定されているが該当する itemがない|
| 400     |  extension required             | extension が指定されていない |
| 400     |  extension invalid              | extension の値が不正|
| 400     |  file_hash conflict             | 同じfile_hash を持つ item が既にある|

**example**

```
{
  "item_id": 118,
  "status": 0,
  "url": "https://carnationdev.s3-ap-northeast-1.amazonaws.com/00000004/00000118.jpg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAI2ZSXBHOXAWRFCQA%2F20150412%2Fap-northeast-1%2Fs3%2Faws4_request&X-Amz-Date=20150412T042537Z&X-Amz-Expires=28800&X-Amz-Signature=6e1e5abea772aa7f5421398bf2652117fac9297115c5190a8bc65b175bc31c83&X-Amz-SignedHeaders=Host"
}
```

### 3.2 notify upload completed

### 3.3 delete item

### 3.4 undelete item

## 4 get item

### 4.1 get single item

### 4.2 get items

## 5 events

### 5.1 get user events

### 5.2 tell the server item(s) are read

### 5.3 tell the server item(s) are unread

### 5.4 tell the server item(s) are retrieved

## 6 device registration

### 6.1 register device to receive push notifications

### 6.2 get the list of registered devices

### 6.3 delete device registration

### 6.4 send a message to a registered device

## 7 viewer API

### 7.1 retrieve users that can be accessed by viewer

### 7.2 post viewer likes item

### 7.3 get viewer info

## 8 file upload

### 8.1 get upload url for HTTP put
