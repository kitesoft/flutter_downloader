# Flutter Downloader

[![pub package](https://img.shields.io/pub/v/flutter_downloader.svg)](https://pub.dartlang.org/packages/flutter_downloader)

A plugin for creating and managing download tasks. Supports iOS and Android. 

This plugin is based on [`WorkManager`][1] in Android and [`NSURLSessionDownloadTask`][2] in iOS to run download task in background mode.


## iOS integration

* Open Xcode. Enable background mode.

<img width="512" src="https://github.com/hnvn/flutter_downloader/blob/master/screenshot/enable_background_mode.png?raw=true"/>

* Add following code to your `AppDelegate` (this method will be called when an URL session finished its work while your app is not running):

````objectivec
- (void)application:(UIApplication *)application handleEventsForBackgroundURLSession:(NSString *)identifier completionHandler:(void (^)(void))completionHandler {
    completionHandler();
}
````

**Note:** If you want to download file with HTTP request, you need to disable Apple Transport Security (ATS) feature.
* Disable ATS for a specific domain only: (add following codes to the end of your `Info.plist` file)
````xml
<key>NSAppTransportSecurity</key>
<dict>
  <key>NSExceptionDomains</key>
  <dict>
    <key>www.yourserver.com</key>
    <dict>
      <!-- add this key to enable subdomains such as sub.yourserver.com -->
      <key>NSIncludesSubdomains</key>
      <true/>
      <!-- add this key to allow standard HTTP requests, thus negating the ATS -->
      <key>NSTemporaryExceptionAllowsInsecureHTTPLoads</key>
      <true/>
      <!-- add this key to specify the minimum TLS version to accept -->
      <key>NSTemporaryExceptionMinimumTLSVersion</key>
      <string>TLSv1.1</string>
    </dict>
  </dict>
</dict>
```` 

* Completely disable ATS: (add following codes to the end of your `Info.plist` file)

````xml
<key>NSAppTransportSecurity</key>  
<dict>  
    <key>NSAllowsArbitraryLoads</key><true/>  
</dict>
````

## Android integration

In order to handle click action on notification to open the downloaded file on Android, you need to add some additional configurations:

* add the following codes to your `AndroidManifest.xml` (inside `application` tag):

````xml
<provider
    android:name="vn.hunghd.flutterdownloader.DownloadedFileProvider"
    android:authorities="${applicationId}.flutter_downloader.provider"
    android:exported="false"
    android:grantUriPermissions="true">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/provider_paths"/>
</provider>
````

* you have to save your downloaded files in external storage (where the other applications have permission to read your files)

### Note:
The downloaded files are only able to be opened if your device has at least an application that can read these file types (mp3, pdf, etc) 

## Usage

````dart
import 'package:flutter_downloader/flutter_downloader.dart';
````

To create new download task:

````dart
final taskId = await FlutterDownloader.enqueue(
  url: 'your download link', 
  savedDir: 'the path of directory where you want to save downloaded files', 
  showNotification: true, // show download progress in status bar (for Android)
  clickToOpenDownloadedFile: true, // click on notification to open downloaded file (for Android)
);
````

To update download progress:

````dart
FlutterDownloader.registerCallback((id, status, progress) {
  // code to update your UI
});
````

To load the status of download tasks:

````dart
final tasks = await FlutterDownloader.loadTasks();
````

To cancel a task:

````dart
FlutterDownloader.cancel(taskId: taskId);
````

To cancel all tasks:

````dart
FlutterDownloader.cancelAll();
````

[1]: https://developer.android.com/topic/libraries/architecture/workmanager
[2]: https://developer.apple.com/documentation/foundation/nsurlsessiondownloadtask?language=objc