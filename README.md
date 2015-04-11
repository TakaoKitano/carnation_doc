# carnation server API reference

## 1 OAuth2 アクセストークン取得

carnation API の呼び出しを行うためには、OAuth2 access token が必要です.
[RFC6749 The OAuth 2.0 Authorization Framework](https://tools.ietf.org/html/rfc6749)


carnation server では、カメラアプリからの呼び出しに用いる token を user token, STB viewer からのAPI呼び出しに用いる token を viewer token と呼んでいます.

user token は [Client Password](https://tools.ietf.org/html/rfc6749#section-2.3.1) の方式で取得します. 

viewer token は、[Client Credentials Grant](https://tools.ietf.org/html/rfc6749#section-4.4) の方式で取得します.

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
| grant_type    |  "refresh_token"   |    必須     | N/A            |
| refresh_token |  refresh token     |    必須     | N/A            |

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

### 1.2 get viewer token

| method        | end point     |
|---------------|---------------|
| PUT           | /token    |

Authentication header にセットする appid, secret は、それぞれのSTBで独自な値を持ちます.
STBの出荷時になんらかの手段で端末に搭載されます.


**parameters**

| name          |  value                |  mandatory?   | default value  |
|---------------|-----------------------|---------------|----------------|
| grant_type    |  "client_credential"  |    必須       | N/A            |

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

| method        | end point     |
|---------------|---------------|
| POST          | /api/v1/user  |

ユーザーを作成します.
この API を呼び出すためには、admin もしくは signup の権限が必要です。


**parameters**

| name          |  value          |  mandatory?   | default value  |
|---------------|-----------------|---------------|----------------|
| email         |  メールアドレス |    必須       | N/A            |
| password      |  パスワード     |    必須       | N/A            |

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

| method        | end point     |
|---------------|---------------|
| POST          | /api/v1/attributes  |

ユーザーの email, name, password を変更します
この API を呼び出すためには、admin もしくは user_id のユーザーであることが必要です.
パラメータで指定したもののみが指定した値に変更されます.

**parameters**

| name          |  value          |  mandatory?   | default value  |
|---------------|-----------------|---------------|----------------|
| user_id       |  user id        |   yes       | N/A            |
| email         |  user email address  |    no       | N/A            |
| name          |  user name      |    no       | N/A            |
| password      |  user password     |    no       | N/A            |

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

| method        | end point     |
|---------------|---------------|
| DELETE        | /api/v1/user  |

ユーザーを削除します
この API を呼び出すためには、admin 権限が必要です.

**parameters**

| name          |  value          |  mandatory?   | default value  |
|---------------|-----------------|---------------|----------------|
| user_id       |  user id        |   yes       | N/A            |

**output**

| name          |  value          | 
|---------------|-----------------|
| id         |  user_id |
| email      |  user email      | 
| name      |  user name |

**error**

| code    |  message                        | description |
|---------|---------------------------------|---------------------------|
| 400     |  access denied                  | 権限がない |
| 400     |  user not found                  | user_id 不正 |

**example**

```
{
  "id": 4,
  "email": "test01@chikaku.com",
  "name": "harahara",
}
```

### 2.4 get user info

### 2.5 get user id by email

## 3 item upload & delete

### 3.1 initiate item upload

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
