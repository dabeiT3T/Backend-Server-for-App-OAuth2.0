# [移动应用微信登陆开发][weixinoauth]

## 准备工作

```
1、目前移动应用上微信登录只提供原生的登录方式，需要用户安装微信客户端才能配合使用。

2、对于Android应用，建议总是显示微信登录按钮，当用户手机没有安装微信客户端时，请引导用户下载安装微信客户端。

3、对于iOS应用，考虑到iOS应用商店审核指南中的相关规定，建议开发者接入微信登录时，先检测用户手机是否已安装微信客户端（使用sdk中isWXAppInstalled函数 ），对未安装的用户隐藏微信登录按钮，只提供其他登录方式（比如手机号注册登录、游客登录等）。
```

## 授权流程

### 时序图

![获取access_token时序图][access_token_sequence]

### 请求CODE

*移动端执行*

| 参数  | 是否必须 | 说明                                                         |
| ----- | -------- | ------------------------------------------------------------ |
| appid | 是       | 应用唯一标识，在微信开放平台提交应用审核通过后获得           |
| scope | 是       | 应用授权作用域，如获取用户个人信息则填写snsapi_userinfo（ [什么是授权域？](https://open.weixin.qq.com/cgi-bin/showdocument?action=doc&id=open1419317851&t=0.18835821719976198#0) ） |
| state | 否       | 用于保持请求和回调的状态，授权请求后原样带回给第三方。该参数可用于防止csrf攻击（跨站请求伪造攻击），建议第三方带上该参数，可设置为简单的随机数加session进行校验 |

#### 返回说明

| 返回值 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| code   | 用户换取access_token的code，仅在ErrCode为0时有效             |
| state  | 第三方程序发送时用来标识其请求的唯一性的标志，由第三方程序调用sendReq时传入，由微信终端回传，state字符串长度不能超过1K |

### 通过code获取 access_token

| 参数       | 是否必须 | 说明                                                    |
| ---------- | -------- | ------------------------------------------------------- |
| appid      | 是       | 应用唯一标识，在微信开放平台提交应用审核通过后获得      |
| secret     | 是       | 应用密钥AppSecret，在微信开放平台提交应用审核通过后获得 |
| code       | 是       | 填写第一步获取的code参数                                |
| grant_type | 是       | 填authorization_code                                    |

#### 返回说明

| 参数          | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| access_token  | 接口调用凭证                                                 |
| expires_in    | access_token接口调用凭证超时时间，单位（秒）                 |
| refresh_token | 用户刷新access_token                                         |
| openid        | 授权用户唯一标识                                             |
| scope         | 用户授权的作用域，使用逗号（,）分隔                          |
| unionid       | 当且仅当该移动应用已获得该用户的userinfo授权时，才会出现该字段 |

### 刷新 access_token 有效期

access_token是调用授权关系接口的调用凭证，由于access_token有效期（目前为2个小时）较短，当access_token超时后，可以使用refresh_token进行刷新

**注意：**

```
1、Appsecret 是应用接口使用密钥，泄漏后将可能导致应用数据泄漏、应用的用户数据泄漏等高风险后果；存储在客户端，极有可能被恶意窃取（如反编译获取Appsecret）；
2、access_token 为用户授权第三方应用发起接口调用的凭证（相当于用户登录态），存储在客户端，可能出现恶意获取access_token 后导致的用户数据泄漏、用户微信相关接口功能被恶意发起等行为；
3、refresh_token 为用户授权第三方应用的长效凭证，仅用于刷新access_token，但泄漏后相当于access_token 泄漏，风险同上。
```

**建议将Appsecret、用户数据（如access_token）放在App云端服务器，由云端中转接口调用请求。**

### 通过 access_token 调用接口

| 授权作用域（scope）       | 接口说明                                             |
| ------------------------- | ---------------------------------------------------- |
| snsapi_base               | 通过code换取access_token、refresh_token和已授权scope |
| /sns/oauth2/refresh_token |                                                      |
| /sns/auth                 | 检查access_token有效性                               |
| snsapi_userinfo           | 获取用户个人信息                                     |

其中snsapi_base属于基础接口，若应用已拥有其它scope权限，则默认拥有snsapi_base的权限。使用snsapi_base可以让移动端网页授权绕过跳转授权登录页请求用户授权的动作，直接跳转第三方网页带上授权临时票据（code），但会使得用户已授权作用域（scope）仅为snsapi_base，从而导致无法获取到需要用户授权才允许获得的数据和基础功能。

**注意：** `snsapi_base` 即[微信公众号静默授权][wechat_webpage_authorization]，用户无感知。



[weixinoauth]: https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=open1419317851&token=&lang=zh_CN
[access_token_sequence]: https://res.wx.qq.com/op_res/ZLIc-BdWcu_ixroOT0sBEtk0UwpTewqS6ujxbC2QOpbKIVp_DzleM_C9I-9GPDDh
[wechat_webpage_authorization]: https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/Wechat_webpage_authorization.html
