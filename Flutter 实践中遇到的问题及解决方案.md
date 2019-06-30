# Flutter 实践中遇到的问题及解决方案
### Flutter 使用setData无法更新数据
Flutter 页面文件 嵌套 FlutterList列表文件，
setState 后，Flutter列表 部分数据不更新。

原因：未知
**解决方法**：
在Flutter列表页文件，build方法中通过widget获取值：
例如：
```
_loading = widget._showLoading;
```

### Flutter 在 release 环境下启动失败
Flutter 在 release 环境下启动失败，报如下异常：
```
fatal flutter/shell/platform/android/library_loader.cc
```
或者
```
/system/lib/libc.so (abort+57) [armeabi-v7a]
```

原因：没有添加 Flutter 的混淆规则。
**解决方案**：
在项目的  proguard-rules.pro 文件下添加
```
-keep class io.flutter.app.** { *; }
-keep class io.flutter.plugin.**  { *; }
-keep class io.flutter.util.**  { *; }
-keep class io.flutter.view.**  { *; }
-keep class io.flutter.**  { *; }
-keep class io.flutter.plugins.**  { *; }
```

