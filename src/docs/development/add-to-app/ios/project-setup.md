---
title: Integrate a Flutter module into your iOS project
title: 将 Flutter module 集成到 iOS 项目
short-title: Integrate Flutter
short-title: 集成 Flutter
description: Learn how to integrate a Flutter module into your existing iOS project.
description: 了解如何将 Flutter module 集成到你现有的 iOS 项目中。
---

Flutter can be incrementaly added into your existing iOS application as embedded
frameworks.

Flutter 可以以 framework 框架的形式添加到你的既有 iOS 应用中。

## System requirements

## 系统要求

Your development environment must meet the [macOS system requirements for Flutter][]
with [Xcode installed][]. Flutter supports iOS 8.0 and later.

你的开发环境必须满足 [Flutter 对 macOS 系统的版本要求][macOS system requirements for Flutter]
并 [已经安装 Xcode][Xcode installed]，Flutter 支持 iOS 8.0 及以上。

## Create a Flutter module

## 创建 Flutter module

To embed Flutter into your existing application, first create a Flutter module.

为了将 Flutter 集成到你的既有应用里，第一步要创建一个 Flutter module。

From the command line, run:

在命令行中执行：

```sh
cd some/path/
flutter create --template module my_flutter
```

A Flutter module project is created at `some/path/my_flutter/`. From that
directory, you can run the same `flutter` commands you would
in any other Flutter project, like `flutter run --debug` or `flutter build ios`.
You can also run the module in [Android Studio/IntelliJ][] or [VS Code][] with
the Flutter and Dart plugins.
This project contains a single-view example version of your module before it's
embedded in your existing application, which is useful for incrementally
testing the Flutter-only parts of your code.

Flutter module 会创建在 `some/path/my_flutter/` 目录。
在这个目录中，你可以像在其它 Flutter 项目中一样，执行 `flutter` 命令。
比如 `flutter run --debug` 或者 `flutter build ios`。
你也同样可以在 [Android Studio/IntelliJ][] 或者 [VS Code][] 中运行这个模块，
并附带 Flutter 和 Dart 插件。在集成到既有应用前，
这个项目在 Flutter module 中包含了一个单视图的示例代码，
对 Flutter 侧代码的测试会有帮助。

### Module organization

### 模块组织

The `my_flutter` module directory structure is similar to a normal Flutter
application:

在 `my_flutter` 模块，目录结构和普通 Flutter 应用类似：

```text
my_flutter/
├─.ios/
│ ├─Runner.xcworkspace
│ └─Flutter/podhelper.rb
├─lib/
│ └─main.dart
├─test/
└─pubspec.yaml
```

Add your Dart code to the `lib/` directory.

添加你的 Dart 代码到 `lib/` 目录。

Add Flutter dependencies to `my_flutter/pubspec.yaml`, including Flutter packages
and plugins.

添加 Flutter 依赖到 `my_flutter/pubspec.yaml`，
包括 Flutter packages 和 plugins。

The `.ios/` hidden subfolder contains an Xcode workspace where you can
run a standalone version of your module. It is a wrapper project to bootstrap your Flutter code,
and contains helper scripts to facilitate building frameworks or
embedding the module into your existing application with [CocoaPods][].

`.ios/` 隐藏文件夹包含了一个 Xcode workspace，用于单独运行你的 Flutter module。
它是一个独立启动 Flutter 代码的壳工程，并且包含了一个帮助脚本，
用于编译 framewroks 或者使用 [CocoaPods][] 将 Flutter module 集成到你的既有应用。

{{site.alert.note}}

Add custom iOS code to your existing application or a plugin, not to
the module in `.ios/`. Changes made in `.ios/` aren't embedded in your existing application.
Regenerate the directory by running `flutter clean` or `flutter pub get` in the
`my_flutter` directory.

iOS 代码要添加到你的既有应用或者 Flutter plugin中，
而不是 Flutter module 的 `.ios/` 目录下。
`.ios/` 下的改变不会集成到你的既有应用。
在 `my_flutter` 执行 `flutter clean`
或者 `flutter pub get` 会重新生成这个目录。

{{site.alert.end}}

## Embed the Flutter module in your existing application

## 在你的既有应用中集成 Flutter module

There are two ways to embed Flutter in your existing application.

这里有两种方式可以将 Flutter 集成到你的既有应用中。

1. Use the CocoaPods dependency manager and installed Flutter SDK. (Recommended.)

   使用 CocoaPods 依赖管理和已安装的 Flutter SDK 。（推荐）

2. Create frameworks for the Flutter engine, your compiled Dart code, and all
   Flutter plugins. Manually embed the frameworks, and update your existing
   application's build settings in Xcode.
   
   把 Flutter engine 、你的 dart 代码和所有 Flutter plugin 编译成 framework 。
   用 Xcode 手动集成到你的应用中，并更新编译设置。

{{site.alert.note}}

Your app will not run on a simulator in Release mode because Flutter does
not yet support output x86 ahead-of-time (AOT) binaries for your Dart code. You can run
in Debug mode on a simulator or a real device, and Release on a real device.

你的应用将不能在模拟器上运行 Release 模式，
因为 Flutter 还不支持将 Dart 代码编译成 x86 ahead-of-time (AOT) 模式的二进制文件。
你可以在模拟机和真机上运行 Debug 模式，在真机上运行 Release 模式。

{{site.alert.end}}

Using Flutter will [increase your app size][].

使用 Flutter 会 [增加应用体积][increase your app size] 。

### Option A - Embed with CocoaPods and the Flutter SDK

### 选项 A - 使用 CocoaPods 和 Flutter SDK 集成

This method requires every developer working on your project to have a locally installed
version of the Flutter SDK. Simply build your application in Xcode to automatically run the script to
embed your Dart and plugin code. This allows rapid iteration with the most up-to-date
version of your Flutter module without running additional commands outside of Xcode.

这个方法需要你的项目的所有开发者，都在本地安装 Flutter SDK。
只需要在 Xcode 中编译应用，就可以自动运行脚本来集成 dart 代码和 plugin。
这个方法允许你使用 Flutter module 中的最新代码快速迭代开发，
而无需在 Xcode 以外执行额外的命令。

The following example assumes that your existing application and the Flutter module are in sibling directories.
If you have a different directory structure, you may need to adjust the relative paths.

下面的示例假设你的既有应用和 Flutter module 在相邻目录。如果你有不同的目录结构，需要适配到对应的路径。

```text
some/path/
├── my_flutter/
│   └── .ios/
│       └── Flutter/
│         └── podhelper.rb
└── MyApp/
    └── Podfile
```

If your existing application (`MyApp`) doesn't already have a Podfile, follow the
[CocoaPods getting started guide][] to add a `Podfile` to your project.

如果你的应用（`MyApp`）还没有 Podfile，根据 [CocoaPods getting started guide][] 来在项目中添加 `Podfile`。

1. Add the following lines to your `Podfile`:

   在 `Podfile` 中添加下面代码：

    <?code-excerpt "MyApp/Podfile" title?>
    ```ruby
      flutter_application_path = '../my_flutter'
      load File.join(flutter_application_path, '.ios', 'Flutter', 'podhelper.rb')
    ```

2. For each [Podfile target][] that needs to
embed Flutter, call `install_all_flutter_pods(flutter_application_path)`.

   每个需要集成 Flutter 的 [Podfile target][]，
   执行 `install_all_flutter_pods(flutter_application_path)`：

    <?code-excerpt "MyApp/Podfile" title?>
    ```ruby
      target 'MyApp' do
        install_all_flutter_pods(flutter_application_path)
      end
    ```

3. Run `pod install`.

   运行 `pod install`。

{{site.alert.note}}

When you change the Flutter plugin dependencies in `my_flutter/pubspec.yaml`,
run `flutter pub get` in your Flutter module directory to refresh the list
of plugins read by the `podhelper.rb` script. Then, run `pod install` again from
in your application at`some/path/MyApp`.

当你在 `my_flutter/pubspec.yaml` 改变了 Flutter plugin 依赖，
需要在 Flutter module 目录运行 `flutter pub get`，
来更新会被`podhelper.rb` 脚本用到的 plugin 列表，
然后再次在你的应用目录 `some/path/MyApp` 运行 `pod install`.

{{site.alert.end}}

The `podhelper.rb` script embeds your plugins, `Flutter.framework`, and
`App.framework` into your project.

`podhelper.rb` 脚本会把你的 plugins，
`Flutter.framework`，和 `App.framework` 集成到你的项目中。

Your app's Debug and Release build configurations will embed the Debug or
Release [build modes of Flutter][], respectively. Add a Profile build configuration
to your app to test in profile mode.

你应用的 Debug 和 Release 编译配置，将会集成相对应的 
Debug 或 Release 的 [编译产物][build modes of Flutter]。
可以增加一个 Profile 编译配置用于在 profile 模式下测试应用。

{{site.alert.tip}}

`Flutter.framework` is the bundle for the Flutter engine, and `App.framework` is
the compiled Dart code for this project.

`Flutter.framework` 是 Flutter engine 的框架，
`App.framework` 是你的 Dart 代码的编译产物。

{{site.alert.end}}

Open `MyApp.xcworkspace` in Xcode. You can now build the project using `⌘B`.

在 Xcode 中打开 `MyApp.xcworkspace` ，你现在可以使用 `⌘B` 编译项目了。

### Option B - Embed frameworks in Xcode

### 选项 B - 在 Xcode 中集成 frameworks

Alternatively, you can generate the necessary frameworks and embed them in your application
by manually editing your existing Xcode project. You may do this if members of your
team can't locally install Flutter SDK and CocoaPods, or if you don't want to use CocoaPods
as a dependency manager in your existing applications. You must run `flutter build ios-framework`
every time you make code changes in your Flutter module.

除了上面的方法，你也可以创建必备的 frameworks，手动修改既有 Xcode 项目，将他们集成进去。
当你组内其它成员们不能在本地安装 Flutter SDK 和 CocoaPods，
或者你不想使用 CocoaPods 作为既有应用的依赖管理时，这种方法会比较合适。
但是每当你在 Flutter module 中改变了代码，
都必须运行 `flutter build ios-framework`。

If you're using the previous [Embed with CocoaPods and Flutter tools](#option-a---embed-with-cocoapods-and-the-flutter-sdk)
method, you can skip these instructions.

如果你使用前面的
[使用 CocoaPods 和 Flutter SDK 集成](#option-a---embed-with-cocoapods-and-the-flutter-sdk)，
你可以跳过本步骤。

The following example assumes that you want to generate the frameworks to `some/path/MyApp/Flutter/`.

下面的示例假设你想在 `some/path/MyApp/Flutter/` 目录下创建 frameworks：

```sh
flutter build ios-framework --output=some/path/MyApp/Flutter/
```

```text
some/path/MyApp/
└── Flutter/
    ├── Debug/
    │   ├── Flutter.framework
    │   ├── App.framework
    │   ├── FlutterPluginRegistrant.framework
    │   └── example_plugin.framework (each plugin with iOS platform code is a separate framework)
      ├── Profile/
      │   ├── Flutter.framework
      │   ├── App.framework
      │   ├── FlutterPluginRegistrant.framework
      │   └── example_plugin.framework
      └── Release/
          ├── Flutter.framework
          ├── App.framework
          ├── FlutterPluginRegistrant.framework
          └── example_plugin.framework
```

{{site.alert.tip}}

With Xcode 11 installed, you can generate [XCFrameworks][] instead of universal frameworks by adding
the flags `--xcframework --no-universal`.

在 Xcode 11 中， 你可以添加 `--xcframework --no-universal` 参数来生成 [XCFrameworks][]，而不是通用 framework。

{{site.alert.end}}

Embed the generated frameworks into your existing application in Xcode. For example, you can
drag the frameworks from `some/path/MyApp/Flutter/Release/` in Finder
into your targets's build settings > General > Frameworks, Libraries, and Embedded Content. Then, select
"Embed & Sign" from the drop-down list.

在 Xcode 中将生成的 frameworks 集成到你的既有应用中。
例如，你可以在 `some/path/MyApp/Flutter/Release/` 
目录拖拽 frameworks 到 你的应用 target 编译设置的 
General > Frameworks, Libraries, and Embedded Content 下，
然后在 Embed 下拉列表中选择 "Embed & Sign"。

{% include app-figure.md image="development/add-to-app/ios/project-setup/embed-xcode.png" alt="Embed frameworks in Xcode" %}

In the target's build settings, add `$(PROJECT_DIR)/Flutter/Release/` to your Framework Search Paths (`FRAMEWORK_SEARCH_PATHS`).

在 target 的编译设置中的 Framework Search Paths (`FRAMEWORK_SEARCH_PATHS`) 增加 `$(PROJECT_DIR)/Flutter/Release/`。

{% include app-figure.md image="development/add-to-app/ios/project-setup/framework-search-paths.png" alt="Update Framework Search Paths in Xcode" %}

There are multiple ways to embed frameworks into an Xcode project—use the method that is best for your project.

在 Xcode 项目中即成 frameworks 有很多方法 —— 选择最适合你的项目的。

You should now be able to build the project in Xcode using `⌘B`.

你现在可以在 Xcode中使用 `⌘B` 编译项目。

{{site.alert.tip}}

To embed the Debug version of the Flutter frameworks in your Debug build configuration
and the Release version in your Release configuration, in your `MyApp.xcodeproj/project.pbxproj`, try
replacing `path = Flutter/Release/example.framework;`
with `path = "Flutter/$(CONFIGURATION)/example.framework";` for all added frameworks. (Note the added `"`.)

如果你想在 Debug 编译配置下使用 Debug 版本的 Flutter frameworks，
在 Release 编译配置下使用 Release 版本的 Flutter frameworks，
在 `MyApp.xcodeproj/project.pbxproj` 文件中，
尝试在所有 Flutter 相关 frameworks 上使用
`path = "Flutter/$(CONFIGURATION)/example.framework";`
替换 `path = Flutter/Release/example.framework;` 
（注意添加引号 `"`）。

You must also add `$(PROJECT_DIR)/Flutter/$(CONFIGURATION)` to your Framework Search Paths build setting.

你也必须在 Framework Search Paths 编译设置中使用 `$(PROJECT_DIR)/Flutter/$(CONFIGURATION)`。

{{site.alert.end}}

## Development

## 开发

You can now [add a Flutter screen][] to your existing application.

你现在可以 [添加一个 Flutter 页面][add a Flutter screen] 到你的既有应用中。


[macOS system requirements for Flutter]: /docs/get-started/install/macos#system-requirements
[Xcode installed]: /docs/get-started/install/macos#install-xcode
[Android Studio/IntelliJ]: /docs/development/tools/android-studio
[VS Code]: /docs/development/tools/vs-code
[CocoaPods]: https://cocoapods.org/
[CocoaPods getting started guide]: https://guides.cocoapods.org/using/using-cocoapods.html
[Podfile target]: https://guides.cocoapods.org/syntax/podfile.html#target
[increase your app size]: /docs/resources/faq#how-big-is-the-flutter-engine
[build modes of Flutter]: /docs/testing/build-modes
[XCFrameworks]: https://developer.apple.com/documentation/xcode_release_notes/xcode_11_release_notes
[add a Flutter screen]: /docs/development/add-to-app/ios/add-flutter-screen