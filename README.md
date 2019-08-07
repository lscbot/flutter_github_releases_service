# flutter_github_releases_service

在flutter中每次更新版本，把打包后的apk放在GitHub releases上

此包的目的是检测当前用户使用的app版本，是否是在GitHub releases上的最新版本。

并提供下载，更新，自动安装最新版的apk

注意: 这只在Android上测试过



## install
```
dependencies:
  flutter_github_releases_service:
```

## 修改 'android\app\src\main\AndroidManifest.xml'
更多的配置可以看[flutter_downloader](https://pub.flutter-io.cn/packages/flutter_downloader)包的说明
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android" package="com.example.example">

    <!-- new -->
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.REQUEST_INSTALL_PACKAGES" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
    <!-- new -->
    
    <application android:name="io.flutter.app.FlutterApplication" android:label="example" android:icon="@mipmap/ic_launcher">
        <activity android:name=".MainActivity" android:launchMode="singleTop" android:theme="@style/LaunchTheme" android:configChanges="orientation|keyboardHidden|keyboard|screenSize|locale|layoutDirection|fontScale|screenLayout|density|uiMode" android:hardwareAccelerated="true" android:windowSoftInputMode="adjustResize">
            <meta-data android:name="io.flutter.app.android.SplashScreenUntilFirstFrame" android:value="true" />
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>

        <!-- new -->
        <provider android:name="vn.hunghd.flutterdownloader.DownloadedFileProvider" android:authorities="${applicationId}.flutter_downloader.provider" android:exported="false" android:grantUriPermissions="true">
            <meta-data android:name="android.support.FILE_PROVIDER_PATHS" android:resource="@xml/provider_paths"/>
        </provider>

        <provider android:name="androidx.work.impl.WorkManagerInitializer" android:authorities="${applicationId}.workmanager-init" android:enabled="false" android:exported="false" />

        <provider android:name="vn.hunghd.flutterdownloader.FlutterDownloaderInitializer" android:authorities="${applicationId}.flutter-downloader-init" android:exported="false">
            <meta-data android:name="vn.hunghd.flutterdownloader.MAX_CONCURRENT_TASKS" android:value="5" />
        </provider>
        <!-- new -->

    </application>
</manifest>
```


## Usage example:
```dart
import 'package:flutter_github_releases_service/dto/github_releases/github_releases.dto.dart';
import 'package:flutter_github_releases_service/flutter_github_releases_service.dart';

  GithubReleasesService githubReleasesService = GithubReleasesService(
    owner: 'januwA',
    repo: 'flutter_anime_app',
  );

  Future<GithubReleasesDto> get latest => githubReleasesService.latest;

  RaisedButton(
    onPressed: () async {
      print(
          'latestVersion: ' + await githubReleasesService.latestVersion);
      print('localVersion: ' + await githubReleasesService.localVersion);
      print(await githubReleasesService.isNeedUpdate);

      if (await githubReleasesService.isNeedUpdate) {
        var _latest = await latest;

        try {
          githubReleasesService.downloadApk(
            downloadUrl: _latest.assets.first.browserDownloadUrl,
            apkName: _latest.assets.first.name,
          );
        } catch (e) {
          print('安装失败: $e');
        }
      }
    },
    child: Text('Test'),
  )
```