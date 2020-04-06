# photo_manager

[![pub package](https://img.shields.io/pub/v/photo_manager.svg) ![dev](https://img.shields.io/pub/v/photo_manager?include_prereleases)](https://pub.dartlang.org/packages/photo_manager)
[![GitHub](https://img.shields.io/github/license/Caijinglong/flutter_photo_manager.svg)](https://github.com/Caijinglong/flutter_photo_manager)
[![GitHub stars](https://img.shields.io/github/stars/Caijinglong/flutter_photo_manager.svg?style=social&label=Stars)](https://github.com/Caijinglong/flutter_photo_manager)

A flutter api for photo, you can get image/video from ios or android.

一个提供相册 api 的插件, android ios 可用,没有 ui,以便于自定义自己的界面, 你可以通过提供的 api 来制作图片相关的 ui 或插件

If you just need a picture selector, you can choose to use [photo](https://pub.dartlang.org/packages/photo) library , a multi image picker. All UI create by flutter.

- [photo_manager](#photomanager)
  - [install](#install)
    - [Add to pubspec](#add-to-pubspec)
    - [import in dart code](#import-in-dart-code)
  - [Usage](#usage)
    - [request permission](#request-permission)
    - [you get all of asset list (gallery)](#you-get-all-of-asset-list-gallery)
      - [FilterOption](#filteroption)
    - [Get asset list from `AssetPathEntity`](#get-asset-list-from-assetpathentity)
      - [paged](#paged)
      - [range](#range)
      - [Old version](#old-version)
    - [AssetEntity](#assetentity)
      - [location info of android Q](#location-info-of-android-q)
      - [Origin description](#origin-description)
    - [observer](#observer)
    - [Experimental](#experimental)
      - [Delete item](#delete-item)
      - [Insert new item](#insert-new-item)
      - [Copy asset](#copy-asset)
        - [Only for iOS](#only-for-ios)
        - [Only for Android](#only-for-android)
  - [iOS config](#ios-config)
    - [iOS plist config](#ios-plist-config)
    - [enabling localized system albums names](#enabling-localized-system-albums-names)
  - [android config](#android-config)
    - [about androidX](#about-androidx)
    - [Android Q privacy](#android-q-privacy)
    - [glide](#glide)
  - [common issues](#common-issues)
    - [ios build error](#ios-build-error)

## install

### Add to pubspec

the latest version is [![pub package](https://img.shields.io/pub/v/photo_manager.svg)](https://pub.dartlang.org/packages/photo_manager)

```yaml
dependencies:
  photo_manager: $latest_version
```

### import in dart code

```dart
import 'package:photo_manager/photo_manager.dart';
```

## Usage

### request permission

You must get the user's permission on android/ios.

```dart
var result = await PhotoManager.requestPermission();
if (result) {
    // success
} else {
    // fail
    /// if result is fail, you can call `PhotoManager.openSetting();`  to open android/ios applicaton's setting to get permission
}
```

### you get all of asset list (gallery)

```dart
List<AssetPathEntity> list = await PhotoManager.getAssetPathList();
```

| name         | description                        |
| ------------ | ---------------------------------- |
| hasAll       | Is there an album containing "all" |
| type         | image/video/all , default all.     |
| filterOption | See FilterOption.                  |

#### FilterOption

| name               | description                                                                                                                                                                               |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| needTitle          | The title attribute of the picture must be included in android (even if it is false), it is more performance-consuming in iOS, please consider whether you need it. The default is false. |
| sizeConstraint     | Constraints on resource size.                                                                                                                                                             |
| durationConstraint | Constraints of time, pictures will ignore this constraint.                                                                                                                                |
| dateTimeCond       | Includes date filtering and date sorting                                                                                                                                                  |

Example see [filter_option_page.dart](https://github.com/CaiJingLong/flutter_photo_manager/example/lib/page/filter_option_page.dart).

### Get asset list from `AssetPathEntity`

#### paged

```dart
// page: The page number of the page, starting at 0.
// perPage: The number of pages per page.
final assetList = await path.getAssetListPaged(page, perPage);
```

The old version, it is not recommended for continued use, because there may be performance issues on some phones. Now the internal implementation of this method is also paged, but the paged count is assetCount of AssetPathEntity.

#### range

```dart
final assetList = await path.getAssetListRange(start: 0, end: 88); // use start and end to get asset.
// Example: 0~10 will return 10 assets. Special case: If there are only 5, return 5
```

#### Old version

```dart
AssetPathEntity data = list[0]; // 1st album in the list, typically the "Recent" or "All" album
List<AssetEntity> imageList = await data.assetList;
```

### AssetEntity

```dart
AssetEntity entity = imageList[0];

File file = await entity.file; // image file

Uint8List originBytes = await entity.originBytes; // image/video original file content,

Uint8List thumbBytes = await entity.thumbData; // thumb data ,you can use Image.memory(thumbBytes); size is 64px*64px;

Uint8List thumbDataWithSize = await entity.thumbDataWithSize(width,height); //Just like thumbnails, you can specify your own size. unit is px; format is optional support jpg and png.

AssetType type = entity.type; // the type of asset enum of other,image,video

Duration duration = entity.videoDuration; //if type is not video, then return null.

Size size = entity.size

int width = entity.width;

int height = entity.height;

DateTime createDt = entity.createDateTime;

DateTime modifiedDt = entity.modifiedDateTime;

/// Gps info of asset. If latitude and longitude is 0, it means that no positioning information was obtained.
/// This information is not necessarily available, because the photo source is not necessarily the camera.
/// Even the camera, due to privacy issues, this property must not be available on androidQ and above.
double latitude = entity.latitude;
double longitude = entiry.longitude;

Latlng latlng = await entity.latlngAsync(); // In androidQ or higher, need use the method to get location info.

String mediaUrl = await entity.getMediaUrl(); /// It can be used in some video player plugin to preview, such as [flutter_ijkplayer](https://pub.dev/packages/flutter_ijkplayer)

String title = entity.title;
```

About title: if the title is null or empty string, need use the titleAsync to get it. See below for the definition of attributes.

```dart

  /// It is title `MediaStore.MediaColumns.DISPLAY_NAME` in MediaStore on android.
  ///
  /// It is `PHAssetResource.filename` on iOS.
  ///
  /// Nullable in iOS. If you must need it, See [FilterOption.needTitle] or use [titleAsync].
  String title;

  /// It is [title] in Android.
  ///
  /// It is [PHAsset valueForKey:@"filename"] in iOS.
  Future<String> get titleAsync => _plugin.getTitleAsync(this);
```

#### location info of android Q

Because of AndroidQ's privacy policy issues, it is necessary to locate permissions in order to obtain the original image, and to obtain location information by reading the Exif metadata of the data.

#### Origin description

The `originFile` and `originBytes` will return the original content.

Not guaranteed to be available in flutter.  
Because flutter's Image does not support heic.  
The video is also the original format, non-exported format, compatibility does not guarantee usability.

### observer

use `addChangeCallback` to regiser observe.

```dart
PhotoManager.addChangeCallback(changeNotify);
PhotoManager.startChangeNotify();
```

```dart
PhotoManager.removeChangeCallback(changeNotify);
PhotoManager.stopChangeNotify();
```

### Experimental

**Important**: The functions are not guaranteed to be fully usable, because it involves data modification, some APIs will cause irreversible deletion / movement of the data, so please use test equipment to make sure that there is no problem before using it.

#### Delete item

Hint: this will delete the asset from your device. For iOS, it's not just about removing from the album.

```dart
final List<String> result = await PhotoManager.editor.deleteWithIds([entity.id]); // The deleted id will be returned, if it fails, an empty array will be returned.
```

Tip: You need to call the corresponding `PathEntity`'s `refreshPathProperties` method to refresh the latest assetCount.

And [range](#range) way to get the latest data to ensure the accuracy of the current data. Such as [example](https://github.com/CaiJingLong/flutter_photo_manager/blob/0298d19464c05b231e2e97989f068ec3a72b0ab0/example/lib/model/photo_provider.dart#L104-L113).

#### Insert new item

```dart
final AssetEntity imageEntity = await PhotoManager.editor.saveImage(uint8list); // nullable

final AssetEntity imageEntity = await PhotoManager.editor.saveImageWithPath(path); // nullable

File videoFile = File("video path");
final AssetEntity videoEntity = await await PhotoManager.editor.saveVideo(videoFile); // nullable
```

#### Copy asset

Availability:

- iOS: some albums are smart albums, their content is automatically managed by the system and cannot be inserted manually.
- android:
  - Before api 28, the method will copy some column from origin row.
  - In api 29 or higher, There are some restrictions that cannot be guaranteed, See [document of relative_path](https://developer.android.com/reference/android/provider/MediaStore.MediaColumns#RELATIVE_PATH).

```dart

```

##### Only for iOS

Create folder:

```dart
PhotoManager.editor.iOS.createFolder(
  name,
  parent: parent, // It is a folder or Recent album.
);
```

Create album:

```dart
PhotoManager.editor.iOS.createAlbum(
  name,
  parent: parent, // It is a folder or Recent album.
);
```

Remove asset in album, the asset can't be delete in device, just remove of album.

```dart
PhotoManager.editor.iOS.removeInAlbum();  // remove single asset.
PhotoManager.editor.iOS.removeAssetsInAlbum(); // Batch remove asset in album.
```

Delete the path in device. Both folders and albums can be deleted, except for smart albums.

```dart
PhotoManager.editor.iOS.deletePath();
```

##### Only for Android

Move asset to another album

```dart
PhotoManager.editor.android.moveAssetToAnother(entity: assetEntity, target: pathEntity);
```

Remove all non-existing rows. For normal Android users, this problem doesn't happened.  
A row record exists in the Android MediaStore, but the corresponding file has been deleted. This kind of abnormal deletion usually comes from file manager, helper to clear cache or adb.  
This is a very resource-consuming operation. If the first one is not completed, the second one cannot be opened.

```dart
await PhotoManager.editor.android.removeAllNoExistsAsset();
```

## iOS config

### iOS plist config

Because the album is a privacy privilege, you need user permission to access it. You must to modify the `Info.plist` file in Runner project.

like next

```xml
    <key>NSPhotoLibraryUsageDescription</key>
    <string>App need your agree, can visit your album</string>
```

xcode like image
![in xcode](https://raw.githubusercontent.com/CaiJingLong/some_asset/master/flutter_photo2.png)

In ios11+, if you want to save or delete asset, you also need add `NSPhotoLibraryAddUsageDescription` to plist.

### enabling localized system albums names

By default iOS will retrieve system album names only in English whatever the device's language currently set.
To change this you need to open the ios project of your flutter app using xCode

![in xcode](https://raw.githubusercontent.com/CaiJingLong/some_asset/master/iosFlutterProjectEditinginXcode.png)

Select the project "Runner" and in the localizations table, click on the + icon

![in xcode](https://raw.githubusercontent.com/CaiJingLong/some_asset/master/iosFlutterAddLocalization.png)

Select the adequate language(s) you want to retrieve localized strings.
Validate the popup screen without any modification
Close xCode
Rebuild your flutter project
Now, the system albums should be displayed according to the device's language

## android config

### about androidX

Google recommends completing all support-to-AndroidX migrations in 2019. Documentation is also provided.

This library has been migrated in version 0.2.2, but it brings a problem. Sometimes your upstream library has not been migrated yet. At this time, you need to add an option to deal with this problem.

The complete migration method can be consulted [gitbook](https://caijinglong.gitbooks.io/migrate-flutter-to-androidx/content/).

### Android Q privacy

Now, the android part of the plugin uses api 29 to compile the plugin, so your android sdk environment must contain api 29 (androidQ).

AndroidQ has a new privacy policy, users can't access the original file.

If your compileSdkVersion and targetSdkVersion are both below 28, you can use `PhotoManager.forceOldApi` to force the old api to access the album. If you are not sure about this part, don't call this method.

### glide

Android native use glide to create image thumb bytes, version is 4.9.0.

If your other android library use the library, and version is not same, then you need edit your android project's build.gradle.

```gradle
rootProject.allprojects {

    subprojects {
        project.configurations.all {
            resolutionStrategy.eachDependency { details ->
                if (details.requested.group == 'com.github.bumptech.glide'
                        && details.requested.name.contains('glide')) {
                    details.useVersion "4.9.0"
                }
            }
        }
    }
}
```

## common issues

### ios build error

if your flutter print like the log. see [stackoverflow](https://stackoverflow.com/questions/27776497/include-of-non-modular-header-inside-framework-module)

```bash
Xcode's output:
↳
    === BUILD TARGET Runner OF PROJECT Runner WITH CONFIGURATION Debug ===
    The use of Swift 3 @objc inference in Swift 4 mode is deprecated. Please address deprecated @objc inference warnings, test your code with “Use of deprecated Swift 3 @objc inference” logging enabled, and then disable inference by changing the "Swift 3 @objc Inference" build setting to "Default" for the "Runner" target.
    === BUILD TARGET Runner OF PROJECT Runner WITH CONFIGURATION Debug ===
    While building module 'photo_manager' imported from /Users/cai/IdeaProjects/flutter/sxw_order/ios/Runner/GeneratedPluginRegistrant.m:9:
    In file included from <module-includes>:1:
    In file included from /Users/cai/IdeaProjects/flutter/sxw_order/build/ios/Debug-iphonesimulator/photo_manager/photo_manager.framework/Headers/photo_manager-umbrella.h:16:
    /Users/cai/IdeaProjects/flutter/sxw_order/build/ios/Debug-iphonesimulator/photo_manager/photo_manager.framework/Headers/MD5Utils.h:5:9: error: include of non-modular header inside framework module 'photo_manager.MD5Utils': '/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator11.2.sdk/usr/include/CommonCrypto/CommonDigest.h' [-Werror,-Wnon-modular-include-in-framework-module]
    #import <CommonCrypto/CommonDigest.h>
            ^
    1 error generated.
    /Users/cai/IdeaProjects/flutter/sxw_order/ios/Runner/GeneratedPluginRegistrant.m:9:9: fatal error: could not build module 'photo_manager'
    #import <photo_manager/ImageScannerPlugin.h>
     ~~~~~~~^
    2 errors generated.
```
