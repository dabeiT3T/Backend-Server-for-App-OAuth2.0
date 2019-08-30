# Google Sign-In for Android

## 总结

 [官方文档][googleoauth]是这么说：

0. `Android` 工程配置见文档。

1. 如果 `app` 需要使用 `后端服务器验证` 和 `调用谷歌的接口`，必须先获取为该服务器创建的 `OAuth 2.0 client ID`：

   - 在 API Console 中打开 [Credentials page][credential_page]。
   - `Web application type client ID` 就是后端服务器的 `OAuth 2.0 client ID`。

   > **Note:** If you haven't recently created a new Android client, you might not have a **Web application** type client ID. You can create one by opening the [Credentials page](https://console.developers.google.com/apis/credentials) and clicking **New credentials > OAuth client ID**.
   >
   > 
   >
   > 如果没有生成过，需要在该页面点击 **New credentials > OAuth client ID** 

   当你创建 `GoogleSignInOption` 对象时，把 `client ID` 传参给 `requestIDtOKEN` 或 `requestServerAuthCode`  方法。

2. 由于都是一个爹，谷歌提供 `GoogleSignIn.silentSignIn` 方法，来检测用户是否已经使用谷歌登录，并使用 `GoogleSignIn.getLastSignedInAccount` 方法获取信息。这个可能会省去例如微信第三方登录在后台杀死的情况下再次登录的一系列验证操作。

3. 和 `QQ` 一样，用户信息直接能够在手机中获得存放在 `GoogleSignInAccount` 对象中。

   ```java
   GoogleSignInAccount acct = GoogleSignIn.getLastSignedInAccount(getActivity());
   if (acct != null) {
   	String personName = acct.getDisplayName();
   	String personGivenName = acct.getGivenName();
   	String personFamilyName = acct.getFamilyName();
   	String personEmail = acct.getEmail();
   	String personId = acct.getId();
   	Uri personPhoto = acct.getPhotoUrl();
   }
   ```

   > **Note:** A Google account's email address can change, so don't use it to identify a user. Instead, use the account's ID, which you can get on the client with `GoogleSignInAccount.getId`, and on the backend from the `sub` claim of the ID token.
   >
   > 
   >
   > 谷歌账号的 `email address` 是可变的，不要用其标识用户，而要用用户 `ID`。

**正文开始：**

4. 登陆时调用 `requestIdToken` 将 `标题1` 中生成的 `client ID` 作为实参传入。

   ```java
   // Request only the user's ID token, which can be used to identify the
   // user securely to your backend. This will contain the user's basic
   // profile (name, profile picture URL, etc) so you should not need to
   // make an additional call to personalize your application.
   GoogleSignInOptions gso = new GoogleSignInOptions.Builder(GoogleSignInOptions.DEFAULT_SIGN_IN)
           .requestIdToken(getString(R.string.server_client_id))
           .requestEmail()
           .build();
   ```

5. `app` 将获取的 `idToken` 及 `用户信息` 发送至服务器。

6. 服务器需要确保：

   - 使用谷歌的公钥验证 `idToken` 签名；
   - `aud` 应该匹配服务器的 `client ID` 以防其他恶意 `app` 伪装用户；
   - `iss` 的值为 `accounts.google.com` 或 `https://accounts.google.com`；
   - `exp` 为过期时间；
   - 如果使用 `G Suite domain` 需要验证是否正确；

7. 不过谷歌封装了方法（其它语言可以在官方文档中查看）：

   ```bash
   composer require google/apiclient
   ```

   ```php
   require_once 'vendor/autoload.php';
   
   // Get $id_token via HTTPS POST.
   
   $client = new Google_Client(['client_id' => $CLIENT_ID]);  // Specify the CLIENT_ID of the app that accesses the backend
   $payload = $client->verifyIdToken($id_token);
   if ($payload) {
       $userid = $payload['sub'];
       // If request specified a G Suite domain:
       //$domain = $payload['hd'];
   } else {
       // Invalid ID token
   }
   ```

   该方法自动校验 `JWT signature`、`aud`、`exp`、`iss`。

8. 也可以使用微信方式，向谷歌服务器发送 `GET` 或 `POST`验证请求。

   ```http
   GET https://oauth2.googleapis.com/tokeninfo?id_token=XYZ123
   ```

   成功返回 `json` 数据，但还是要检查 `aud` 是否匹配 `client Id`。使用 `G Suite domain` 将会多一个 `hd` 字段。

   ```json
   {
       // These six fields are included in all Google ID Tokens.
       "iss": "https://accounts.google.com",
       "sub": "110169484474386276334",
       "azp": "1008719970978-hb24n2dstb40o45d4feuo2ukqmcc6381.apps.googleusercontent.com",
       "aud": "1008719970978-hb24n2dstb40o45d4feuo2ukqmcc6381.apps.googleusercontent.com",
       "iat": "1433978353",
       "exp": "1433981953",
   
       // These seven fields are only included when the user has granted the "profile" and
       // "email" OAuth scopes to the application.
       "email": "testuser@gmail.com",
       "email_verified": "true",
       "name" : "Test User",
       "picture": "https://lh4.googleusercontent.com/-kYgzyAWpZzJ/ABCDEFGHI/AAAJKLMNOP/tIXL9Ir44LE/s99-c/photo.jpg",
       "given_name": "Test",
       "family_name": "User",
       "locale": "en"
   }
   ```

9. 保存新用户并开启会话。

10. 谷歌提供 [Cross Account protection][cross_account_protection] 将会在用户受到账号安全问题时发送警报。



[googleoauth]: https://developers.google.com/identity/sign-in/android/start
[credential_page]: https://console.developers.google.com/apis/credentials
[cross_account_protection]: https://developers.google.com/identity/risc
