
Table of Contents
=================

  * [carnation server API reference](#carnation-server-api-reference)
    * [OAuth2 アクセストークン取得](#oauth2-%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9%E3%83%88%E3%83%BC%E3%82%AF%E3%83%B3%E5%8F%96%E5%BE%97)
      * [Authentication header](#authentication-header)
        * [user token 取得の際の appid, secret](#user-token-%E5%8F%96%E5%BE%97%E3%81%AE%E9%9A%9B%E3%81%AE-appid-secret)
          * [viewer token 取得の際の appid, secret](#viewer-token-%E5%8F%96%E5%BE%97%E3%81%AE%E9%9A%9B%E3%81%AE-appid-secret)
      * [get user token](#get-user-token)
        * [parameters](#parameters)
      * [get viewer token](#get-viewer-token)
        * [parameters](#parameters-1)
    * [user management](#user-management)
      * [create new user](#create-new-user)
      * [change user attribute](#change-user-attribute)
      * [delete user](#delete-user)
      * [get user info](#get-user-info)
      * [get user id by email](#get-user-id-by-email)
    * [upload](#upload)
      * [initiate item upload](#initiate-item-upload)
      * [notify upload completed](#notify-upload-completed)
      * [delete item](#delete-item)
      * [undelete item](#undelete-item)
    * [get item](#get-item)
      * [get single item](#get-single-item)
      * [get items](#get-items)
    * [events](#events)
      * [get user events](#get-user-events)
      * [tell the server item(s) are read](#tell-the-server-items-are-read)
      * [tell the server item(s) are unread](#tell-the-server-items-are-unread)
      * [tell the server item(s) are retrieved](#tell-the-server-items-are-retrieved)
    * [device](#device)
      * [register device to receive push notifications](#register-device-to-receive-push-notifications)
      * [get the list of registered devices](#get-the-list-of-registered-devices)
      * [delete device registration](#delete-device-registration)
      * [send a message to a registered device](#send-a-message-to-a-registered-device)
    * [viewer API](#viewer-api)
      * [retrieve users that can be accessed by viewer](#retrieve-users-that-can-be-accessed-by-viewer)
      * [post viewer likes item](#post-viewer-likes-item)
      * [get viewer info](#get-viewer-info)
    * [file upload](#file-upload)
      * [get upload url for HTTP put](#get-upload-url-for-http-put)

# carnation server API reference

## OAuth2 アクセストークン取得

carnation API の呼び出しを行うためには、OAuth2 access token が必要です.
[RFC6749 The OAuth 2.0 Authorization Framework](https://tools.ietf.org/html/rfc6749)


carnation server では、カメラアプリからの呼び出しに用いる token を user token, STB viewer からのAPI呼び出しに用いる token を viewer token と呼んでいます.

user token は [Client Password](https://tools.ietf.org/html/rfc6749#section-2.3.1) の方式で取得します. 

viewer token は、[Client Credentials Grant](https://tools.ietf.org/html/rfc6749#section-4.4) の方式で取得します.

### Authentication header 

user token, viewer token どちらもの取得のときも、[RFC2617] (https://tools.ietf.org/html/rfc2617) で規定されている Basic Authentication header を設定する必要があります.
RFC2617 で用いられている Authentication header に設定する userid, password はそれぞれ carnation server の内部的な値では次のものに相当します.


| rfc2617       | carnation     |
|---------------|---------------|
| userid        | appid         |
| password      | secret        |

#### user token 取得の際の appid, secret

appid, secret はサービス運営者からアサインされるもので、アプリケーションのタイプやバージョンごとに異なるものを持つことができます（共通でも構わない）.
そのアプリケーションが carnation server にアクセスする権限を持つかどうか carnation server が判断するためのものです.
アプリケーション開発者が 個別のアプリケーションに搭載します.

#### viewer token 取得の際の appid, secret

appid, secret は、それぞれのSTBで独自な値を持ちます.
STBの出荷時になんらかの手段で端末に搭載されます.

### get user token

| method        | end point     |
|---------------|---------------|
| PUT           | /token    |

#### parameters 

| name        |  value             |  mandatory? | default value  |
|-------------|--------------------|-------------|----------------|
| grant_type  |  "password"        |    必須     | N/A            |
| username    |  ユーザーの email  |    必須     | N/A            |
| password    |  ユーザーパスワード|    必須     | N/A            |

### get viewer token

| method        | end point     |
|---------------|---------------|
| PUT           | /token    |

#### parameters 

| name          |  value                |  mandatory?   | default value  |
|---------------|-----------------------|---------------|----------------|
| grant_type    |  "client_credential"  |    必須       | N/A            |



## user management

### create new user

### change user attribute

### delete user

### get user info

### get user id by email

## upload

### initiate item upload

### notify upload completed

### delete item

### undelete item

## get item

### get single item

### get items

## events

### get user events

### tell the server item(s) are read

### tell the server item(s) are unread

### tell the server item(s) are retrieved

## device

### register device to receive push notifications

### get the list of registered devices

### delete device registration

### send a message to a registered device

## viewer API

### retrieve users that can be accessed by viewer

### post viewer likes item

### get viewer info

## file upload

### get upload url for HTTP put

