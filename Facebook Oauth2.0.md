# [Facebook Oauth2.0][facebook_login]

## 总结

官方文档是这么说：

1. 登录 `Facebook` 创建应用，添加 `Bundle ID`、启用单点登录。

2. 登陆成功例如在安卓中，`LoginResult` 参数将拥有新的 `AccessToken` 及最新授权或拒绝的权限。

3. 当用户授权了脸书登录，`app` 将获取用户 `access token`。（文档上的意思是）当用户使用了 `SDK` 上的功能，令牌将自动更新；并在最后一次使用后 `60天` 过期。如果不使用 `SDK` 那么你需要使用代码手动来刷新令牌。当用户的 `access token` 过期了，用户应该重新回到登录授权的流程。

4. 由于脸书的社交背景，所以提供更多的接口/权限可以调用。默认可以获取的信息有：

   - id
   - first_name
   - last_name
   - middle_name
   - name
   - name_format
   - picture
   - short_name

   可以通过增加参数的形式请求用户的 `email`，但即使请求了 `email` 权限，也不表示一定能获得电子邮箱。例如，用户使用手机号而不是电子邮箱注册 Facebook 帐户时，那他的电子邮箱字段可能就是空的。

5. 官方给出的安全清单：

   - 切勿在客户端或可反编译的代码中添加应用密钥。
   - 使用应用密钥为所有服务器到服务器的图谱 API 调用签名。
   - 在客户端上使用唯一的短期口令。
   - 不要盲目相信您的应用使用的访问口令实际由您的应用生成。
   - 在使用“登录”对话框时使用状态参数。
   - 在可能的情况下使用我们的官方 SDK。
   - 锁定 Facebook 应用设置，减少应用的攻击面。
   - 使用 HTTPS。

6. 脸书的接口名字叫 [图谱][graph_api]。

7. `图谱` 的结构：

   *节点* 是一个独立的对象，是一个独一无二的 `ID`，可以用来请求信息。

   ```http
   GET https://graph.facebook.com/object-id?access_token=your-access-token
   ```

   其中的 `access_token` 即登陆时获取的用户令牌。例如 `object-id` 为一个节点。

   *字段* 指明了节点请求中具体返回的字段。

   ```http
   GET https://graph.facebook.com/your-facebook-user-id?fields=name&access_token=your-access-token
   ```

   这里只请求用户的 `name` 字段。

   *连线* （官方的翻译）是节点相关的集合。例如一个页面节点上的照片、一个照片节点的评论。查询一个连线需要同时指定节点和连线。

   ```http
   GET https://graph.facebook.com/your-facebook-user-id/photos?access_token=your-access-token
   ```

8. 脸书对于接口调用会有[频率限制][rate_limiting]，例如 `应用级频率限制` 为每小时不得超过 `app` 授权的脸书用户数的200倍。他是所有用户调用的累加，而不单指一个用户。

9. 脸书官方提供了 [PHP SDK][facebook_php_sdk]，其它可在图谱上找到。

10. 测试用户 `access token` 有效性接口 [debug_token][debug_token]。

    ```http
    GET /v4.0/debug_token?input_token={input-token}
    ```

    | Name                   | Description                                                  | Type     |
    | ---------------------- | ------------------------------------------------------------ | -------- |
    | data                   | Data wrapper around the result.                              | object   |
    | app_id                 | The ID of the application this access token is for.          | string   |
    | application            | Name of the application this access token is for.            | string   |
    | expires_at             | Timestamp when this access token expires.                    | unixtime |
    | data_access_expires_at | Timestamp when app's access to user data expires.            | unixtime |
    | is_valid               | Whether the access token is still valid or not.              | bool     |
    | issued_at              | Timestamp when this access token was issued.                 | unixtime |
    | metadata               | General metadata associated with the access token. Can contain data like 'sso', 'auth_type', 'auth_nonce' | object   |
    | profile_id             | For impersonated access tokens, the ID of the page this token contains. | string   |
    | scopes                 | List of permissions that the user has granted for the app in this access token. | string[] |
    | user_id                | The ID of the user this access token is for.                 | string   |

11. 获取本人或其他 Facebook 用户信息的流程是相同的，都只需要使用用户的 Facebook 编号调用接口<del> [User][facebook_user]</del>。可以参考 `编号7`。



[facebook_login]: https://developers.facebook.com/docs/facebook-login/overview
[debug_token]: https://developers.facebook.com/docs/graph-api/reference/v4.0/debug_token
[facebook_user]: https://developers.facebook.com/docs/graph-api/reference/user
[rate_limiting]: https://developers.facebook.com/docs/graph-api/overview/rate-limiting
[facebook_php_sdk]: https://github.com/facebook/php-graph-sdk
[graph_api]: https://developers.facebook.com/docs/graph-api/using-graph-api/using-with-sdks

