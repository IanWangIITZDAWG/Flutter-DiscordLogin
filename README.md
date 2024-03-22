# Discord Login for Flutter App

## 步驟

### 1. 創建 Discord Application

[Discord Developer](https://discord.com/developers/applications)
先自行創建 Discord Application。因為不需要甚麼特別資訊，很簡單，這邊就不介紹創建步驟了。

### 2. 在 Flutter 中使用 WebView

在 Flutter 中沒有 Discord 登入套件可用，所以需要使用 WebView_flutter 從瀏覽器中拿資料回 App。*此篇實作 webview_flutter 版本需要在 4 以上。

### 3. Discord 登入流程

1. 引導至 `https://discord.com/api/oauth2/authorize?client_id=[client_id]&redirect_uri=[redirect_uri]&response_type=code&scope=email+identify+guilds` 讓使用者登入授權。
2. 登入授權成功後，會自動跳轉至你設定的 `[redirect_uri]`，並在 url 後面揭露 authCode，提取 authCode。
3. 提取 authCode 後，呼叫 DiscordApi，`https://discord.com/api/oauth2/token`，body: `{'client_id': [client_id], 'client_secret': [client_secret], 'grant_type': 'authorization_code', 'redirect_uri': [redirect_uri], 'code': authCode`，提取回傳中的 token。
4. 提取 token 後，呼叫 DiscordApi，`https://discord.com/api/users/@me`，headers: `{ 'Authorization': 'Bearer $token' }`，回傳使用者資訊。
5. 用 DiscordApi 回傳的使用者資訊登入你的產品。

### 4. 設定 pubspec.yaml 依賴

在 `pubspec.yaml` 新增依賴 `webview_flutter: ^4.2.0`。

### 5. 新增 Discord Login 頁面

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:test/application/user/bloc.dart';
import 'package:test/infrastructure/login.dart';
import 'package:test/library/global/path.dart';
import 'package:test/types/user/user.dart';
import 'package:webview_flutter/webview_flutter.dart';
import 'package:http/http.dart' as http;

class DiscordLoginPage extends StatefulWidget {
 @override
 _DiscordLoginPageState createState() => _DiscordLoginPageState();
}

class _DiscordLoginPageState extends State<DiscordLoginPage> {
 late WebViewController _webViewController;
 @override
 void initState() {
   _webViewController = WebViewController()
     //允許使用JS
     ..setJavaScriptMode(JavaScriptMode.unrestricted)
     //監聽URL
     ..setNavigationDelegate(NavigationDelegate(onPageStarted: (url) async {
       if (Uri.parse(url).queryParameters.containsKey('code')) {
         var authCode = Uri.parse(url).queryParameters['code'];
         final accessToken = await getAccessToken(authCode!);
         final userInfo = await getUserInfo(accessToken);
         //拿到Discord user資料後，call登入api
         print(userInfo);
       }
     }))
     //載入Widget時載入uri
     ..loadRequest(Uri.parse('https://discord.com/api/oauth2/authorize?client_id=859351116579209226&redirect_uri=https://sports.zbdigital.net/Live&response_type=code&scope=email+identify+guilds'));
   super.initState();
 }

 Future<String> getAccessToken(String authCode) async {
   final uri = Uri.parse('https://discord.com/api/oauth2/token');
   final body = {
     'client_id': 'xxxxxxxx',
     'client_secret': 'xxxxxxxx',
     'grant_type': 'authorization_code',
     'redirect_uri': 'https://3345678',
     'code': authCode,
   };
   final response = await http.post(uri, body: body);
   final accessToken = jsonDecode(response.body)['access_token'];
   return accessToken;
 }

 Future<Map<String, dynamic>> getUserInfo(String accessToken) async {
   final uri = Uri.https(
     'discord.com',
     '/api/users/@me',
   );
   final headers = {
     'Authorization': 'Bearer $accessToken',
   };
   final response = await http.get(uri, headers: headers);
   final userInfo = jsonDecode(response.body);
   return userInfo;
 }

 @override
 Widget build(BuildContext context) {
   return Scaffold(
     appBar: AppBar(
       title: const Text('Discord 登入'),
     ),
     body: WebViewWidget(controller: _webViewController),
   );
 }
}
```

### 6. iOS 設定

iOS 應該沒有額外設定需要做，如果有出錯在自行看狀況修正。
