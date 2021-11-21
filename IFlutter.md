IFlutter
====

## install

### .pub-cache change

```
export PUB_CACHE=/TOOLS/SDK/flutter_linux/.pub-cache
```



### adroid-studio

安装插件flutter

### 问题处理

#### java.lang.NoClassDefFoundError: javax/xml/bind/annotation/XmlSchema

Installing **Android SDK Command-line tools** from **Android SDK Manager** did the job for me.

1. Open **Tools** > **SDK Manager**
2. From the left choose, **Appearance & Behavior** > **System Settings** > **Android SDK**
3. Select **SDK Tools** from the top menu
4. Check **Android SDK Command-line tools** and click 'apply'.
5. `./flutter doctor --android-licenses`



## 开发

![preview](IFlutter.assets/v2-3bdce3e32383a841f1e5c6efd930672b_r-16362842381512.jpg)



### 布局

#### widget

1. StatelessWidget

   它没要需要管理的内部状态，是无状态的。另外一种是可变状态的.

   无状态：不会被改变的Widget，例如纯展示页面，数据也不会改变

2. StatefulWidget

   它有需要管理的内部状态，使用`setState`来管理状态改变。 Widget是有状态的还是无状态的，取决于他们依赖于状态的变化：

   有状态：交互或者数据改变导致Widget改变，例如改变文案

#### container

用来设置背景、设置大小、设置边距(padding)的布局。

#### center

#### Align

`Align`组件可以调整子组件的位置，并且可以根据子组件的宽高来确定自身的宽高

```dart
Container(
  height: 200.0,
  width: 200.0,
  color: Colors.blue[50],
  child: Align(
	  alignment: Alignment.topRight,
	  child: Image(
		width: 100,
		height: 100,
		image: AssetImage('Test.jpg'),
	  )),
);
```



#### column

#### flex



#### row

#### WallLayout



### 动画

#### ScaleTransition

### 状态管理

#### createState()




#### setState()

更新UI

```dart
 void _handleTap() {
    if(!mounted)return;
    setState(() {
      _active = !_active;
    });
  }
```

mounted表明 State 当前是否正确绑定在View树中,mounted = false 时，调用setState()会报错。所以任何时候调用setState()都应检测mounted。

Whether this [State](https://api.flutter.dev/flutter/widgets/State-class.html) object is currently in a tree.

After creating a [State](https://api.flutter.dev/flutter/widgets/State-class.html) object and before calling [initState](https://api.flutter.dev/flutter/widgets/State/initState.html), the framework "mounts" the [State](https://api.flutter.dev/flutter/widgets/State-class.html) object by associating it with a [BuildContext](https://api.flutter.dev/flutter/widgets/BuildContext-class.html). The [State](https://api.flutter.dev/flutter/widgets/State-class.html) object remains mounted until the framework calls [dispose](https://api.flutter.dev/flutter/widgets/State/dispose.html), after which time the framework will never ask the [State](https://api.flutter.dev/flutter/widgets/State-class.html) object to [build](https://api.flutter.dev/flutter/widgets/State/build.html) again.

It is an error to call [setState](https://api.flutter.dev/flutter/widgets/State/setState.html) unless [mounted](https://api.flutter.dev/flutter/widgets/State/mounted.html) is true.

### 路由管理

### 异步开发

#### Future

await



async

