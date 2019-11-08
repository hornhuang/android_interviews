# 前言

----

- 本篇文章将对 `BroadcastReceiver` 开发中，可能用到的知识点，可能遇到的问题进行总结。
- 希望本文能帮助你揭开 `Android` 开发过程中的难题。

> 最后，希望大家阅读愉快！

# 文章目录

----

![BroadcastReceiver](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac176ef2b27?w=1186&h=1128&f=png&s=139626)

# 方便大家学习，我在 GitHub 上建立个 仓库

----

- 仓库内容与博客同步更新。由于我在 `稀土掘金` `简书` `CSDN` `博客园` 等站点，都有新内容发布。所以大家可以直接关注该仓库，以免错过精彩内容！

- 仓库地址：
[超级干货！精心归纳 `Android` 、`JVM` 、算法等，各位帅气的老铁支持一下！给个 Star ！](https://github.com/FishInWater-1999/android_interviews)


# 一、BroadcastReceiver

----

- `BroadcastReceiver`，顾名思义就是“广播接收者”的意思，它是Android四大基本组件之一。
- 这种组件本质上是一种全局的监听器，用于监听系统全局的广播消息。
- 它可以接收来自系统和应用的的广播。

## 1.1 什么是 BroadcastReceiver

![什么是 BroadcastReceiver](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac1720e2d52?w=525&h=185&f=png&s=12804)

- 是四大组件之一, 主要用于接收 `app` 发送的广播
- 内部通信实现机制:通过 `android` 系统的 `Binder` 机制.

## 1.2 广播分为两种

![广播分为两种](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac1774dc971?w=296&h=183&f=png&s=6410)

#### 1.2.1 无序广播

![无序广播](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac17759b96f?w=614&h=236&f=png&s=25709)

- 也叫标准广播，是一种完全异步执行的广播。
- 在广播发出之后，所有广播接收器几乎都会在同一时刻接收到这条广播消息，它们之间没有任何先后顺序，广播的效率较高。
- 优点: 完全异步, 逻辑上可被任何接受者收到广播,效率高
- 缺点: 接受者不能将处理结果交给下一个接受者, 且无法终止广播.

#### 1.2.2 有序广播

![有序广播](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac19ee293d4?w=1131&h=248&f=png&s=46926)

- 是一种同步执行的广播。
- 在广播发出之后，同一时刻只有一个广播接收器能够收到这条广播消息，当其逻辑执行完后该广播接收器才会继续传递。
- 调用 `SendOrderedBroadcast()` 方法来发送广播，同时也可调用 `abortBroadcast()` 方法拦截该广播。可通过 `<intent-filter>` 标签中设置 `android:property` 属性来设置优先级，未设置时按照注册的顺序接收广播。
- 有序广播接受器间可以互传数据。
- 当广播接收器收到广播后，当前广播也可以使用 `setResultData` 方法将数据传给下一个接收器。
- 使用 `getStringExtra` 函数获取广播的原始数据，通过 `getResultData` 方法取得上个广播接收器自己添加的数据，并可用 `abortBroadcast` 方法丢弃该广播，使该广播不再被别的接收器接收到。

![总结](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac19f368f18?w=526&h=229&f=png&s=16342)

- 总结
1. 按被接收者的优先级循序传播 `A > B > C` ,
2. 每个都有权终止广播, 下一个就得不到
3. 每一个都可进行修改操作, 下一个就得到上一个修改后的结果.

#### 1.2.3 最终广播者

![最终广播者](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac19f459e2a?w=1057&h=238&f=png&s=31353)

- `Context.sendOrderedBroadcast ( intent , receiverPermission , resultReceiver , scheduler , initialCode , initialData , initialExtras )` 时我们可以指定 `resultReceiver` 为最终广播接收者.
- 如果比他优先级高的接受者不终止广播, 那么他的 `onReceive` 会执行两次
- 第一次是正常的接收
- 第二次是最终的接收
- 如果优先级高的那个终止广播, 那么他还是会收到一次最终的广播

#### 1.2.4 常见的广播接收者运用场景

![广播接收者运用场景](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac19fd511fe?w=608&h=190&f=png&s=16215)

- 开机启动, `sd` 卡挂载, 低电量, 外拨电话, 锁屏等
- 比如根据产品经理要求, 设计播放音乐时, 锁屏是否决定暂停音乐.


## 1.3 BroadcastReceiver 的种类


#### 1.3.1 广播作为 Android 组件间的通信方式，如下使用场景：

> 对前一部分 “ 请描述一下 `BroadcastReceiver` ” 进行展开补充


![BroadcastReceiver 使用场景](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac1a415f2f9?w=1122&h=320&f=png&s=57892)

- `APP` 内部的消息通信。
- 不同 `APP` 之间的消息通信。

- `Android` 系统在特定情况下与 APP 之间的消息通信。

- 广播使用了观察者模式，基于消息的发布 / 订阅事件模型。广播将广播的发送者和接受者极大程度上解耦，使得系统能够方便集成，更易扩展。

- BroadcastReceiver 本质是一个全局监听器，用于监听系统全局的广播消息，方便实现系统中不同组件间的通信。

- 自定义广播接收器需要继承基类 `BroadcastReceiver` ，并实现抽象方法 `onReceive ( context, intent ) ` 。默认情况下，广播接收器也是运行在主线程，因此 `onReceiver()` 中不能执行太耗时的操作（ 不超过 `10s` ），否则将会产生 `ANR` 问题。`onReceiver()` 方法中涉及与其他组件之间的交互时，可以使用发送 `Notification` 、启动 `Service` 等方式，最好不要启动 `Activity`。


#### 1.3.2 系统广播

![系统广播](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac1a5ca8ba6?w=809&h=311&f=png&s=52133)

- `Android` 系统内置了多个系统广播，只要涉及手机的基本操作，基本上都会发出相应的系统广播，如开机启动、网络状态改变、拍照、屏幕关闭与开启、电量不足等。在系统内部当特定时间发生时，系统广播由系统自动发出。

- 常见系统广播 `Intent` 中的 `Action` 为如下值：

![常见系统广播 Intent 中的 Action](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac1c0ca664e)

1. 短信提醒：`android.provider.Telephony.SMS_RECEIVED`
2. 电量过低：`ACTION_BATIERY_LOW` 
3. 电量发生改变：`ACTION_BATTERY_CHANGED`
4. 连接电源：`ACTION_POWER_CO`                                                                                                                                                                                                                                                                                                      　　　　　　　　　　　　      

- 从 `Android 7.0` 开始，系统不会再发送广播 `ACTION_NEW_PICTURE` 和 `ACTION_NEW_VIDEO` ，对于广播 `CONNECTIVITY_ACTION` 必须在代码中使用 `registerReceiver` 方法注册接收器，在 `AndroidManifest` 文件中声明接收器不起作用。
- 从 `Android 8.0` 开始，对于大多数隐式广播，不能在 `AndroidManifest` 文件中声明接收器。



#### 1.3.3 局部广播

![局部广播](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac1c82a4352?w=869&h=254&f=png&s=42683)

- 局部广播的发送者和接受者都同属于一个 `APP` 
- 相比于全局广播具有以下优点：

1. 其他的 `APP` 不会受到局部广播，不用担心数据泄露的问题。
2. 其他 `APP` 不可能向当前的 `APP` 发送局部广播，不用担心有安全漏洞被其他 `APP` 利用。
3. 局部广播比通过系统传递的全局广播的传递效率更高。

- `Android v4` 包中提供了 `LocalBroadcastManager` 类，用于统一处理 APP 局部广播，使用方式与全局广播几乎相同，只是调用注册 / 取消注册广播接收器和发送广播偶读方法时，需要通过 `LocalBroadcastManager` 类的 `getInstance()` 方法获取的实例调用。




## 1.4 BroadcastReceiver 注册方式

![BroadcastReceiver 注册方式](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac1c852f913?w=947&h=192&f=png&s=18993)

#### 1.4.1 静态注册
在 `AndroidManifest.xml` 文件中配置。

```java
<receiver android:name=".MyReceiver" android:exported="true">
	<intent-filter>
		<!-- 指定该 BroadcastReceiver 所响应的 Intent 的 Action -->
        <action android:name="android.intent.action.INPUT_METHOD_CHANGED"
		<action android:name="android.intent.action.BOOT_COMPLETED" />
	</intent-filter>
</receiver>
```

- 两个重要属性需要关注：

![两个重要属性需要关注](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac1cdcd823e?w=1064&h=206&f=png&s=35669)

1. `android: exported`
其作用是设置此 `BroadcastReceiver` 能否接受其他 `APP` 发出的广播 ，当设为 `false` 时，只能接受同一应用的的组件或具有相同 `user ID` 的应用发送的消息。这个属性的默认值是由 `BroadcastReceiver` 中有无 `Intent-filter` 决定的，如果有 `Intent-filter` ，默认值为 `true` ，否则为 `false` 。
2. `android: permission`
如果设置此属性，具有相应权限的广播发送方发送的广播才能被此 `BroadcastReceiver` 所接受；如果没有设置，这个值赋予整个应用所申请的权限。


#### 1.4.2 动态注册
- 调用 `Context` 的 `registerReceiver ( BroadcastReceiver receiver , IntentFilter filter )` 方法指定。



## 1.5 在 Mainfest 和代码如何注册和使用 BroadcastReceiver ? ( 一个 action 是重点 )

![Mainfest 和代码如何注册和使用 BroadcastReceiver](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac1cd356db7?w=1279&h=309&f=png&s=63111)

####  1.5.1 使用文件注册 ( 静态广播 ) 

- 只要 `app` 还在运行,那么会一直收到广播消息

![使用文件注册 ( 静态广播 ) ](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac1e54b00fa?w=829&h=302&f=png&s=79466)

- 演示:

1. 一个 `app` 里: 自定义一个类继承 `BroadcastReceiver` 然后要求重写 `onReveiver` 方法

```java
public class MyBroadCastReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        Log.d("MyBroadCastReceiver", "收到信息,内容是 :　" + intent.getStringExtra("info") + "");
    }
}
```

2. 清单文件注册,并设置 `Action` , 就那么简单完成接收准备工作

```java
<receiver android:name=".MyBroadCastReceiver">
    <intent-filter>
        <action android:name="myBroadcast.action.call"/>
    </intent-filter>
</receiver>
```

#### 1.5.2 代码注册 ( 动态广播 ) 

- 当注册的 `Activity` 或者 `Service` 销毁了那么就会接收不到广播.

![代码注册 ( 动态广播 ) ](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac1e9ee86eb?w=766&h=593&f=png&s=177398)

- 演示:

1. 在和广播接受者相同的 `app` 里的 `MainActivity` 添加一个注册按钮 , 用来注册广播接收者
2. 设置意图过滤,添加 `Action`

```java
//onCreate创建广播接收者对象
mReceiver = new MyBroadCastReceiver();              

//注册按钮
public void click(View view) {
    IntentFilter intentFilter = new IntentFilter();
    intentFilter.addAction("myBroadcast.action.call");
    registerReceiver(mReceiver, intentFilter);
}
```

3. 销毁的时候取消注册

```java
@Override
protected void onDestroy() {
    unregisterReceiver(mReceiver);
    super.onDestroy();
}
```

#### 1.5.3 在另一个 app , 定义一个按钮, 设置意图, 意图添加消息内容, 意图设置 action( ... ) 要匹配 , 然后发送广播即可.

![代码注册 ( 动态广播 )](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac1f196e0ab?w=827&h=279&f=png&s=74563)

```java
public void click(View view) {
    Intent intent = new Intent();
    intent.putExtra("info", "消息内容");
    intent.setAction("myBroadcast.action.call");
    sendBroadcast(intent);
}
```

- 运行两个 `app` 之后:

1. 静态注册的方法: 另一 `app` 直接发广播就收到了
2. 动态注册的方法: 自己的 `app` 先代码注册,然后另一个 `app` 直接发广播即可.-


## 1.6 BroadcastReceiver 的实现原理是什么？

- `Android` 中的广播使用了设计模式中的观察者模式：基于消息的发布 / 订阅事件模型。

![BroadcastReceiver 的实现原理](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac1f5c43488?w=920&h=200&f=png&s=24786)

- 模型中主要有 `3` 个角色：
1. 消息订阅者（ 广播接收者 ）
2. 消息发布者（ 广播发布者 ）
3. 消息中心（ `AMS`，即 `Activity Manager Service` ）

#### 1.6.1 原理：

![原理](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac20fae2862?w=854&h=241&f=png&s=35333)

- 广播接收者通过 `Binder` 机制在 `AMS`（ `Activity Manager Service` ） 注册；

- 广播发送者通过 `Binder` 机制向 `AMS` 发送广播；

- `AMS` 根据广播发送者要求，在已注册列表中，寻找合适的 `BroadcastReceiver` （ 寻找依据：`IntentFilter / Permission` ）；

- `AMS` 将广播发送到 `BroadcastReceiver` 相应的消息循环队列中；

- 广播接收者通过消息循环拿到此广播，并回调 `onReceive()` 方法。

- 需要注意的是：广播的发送和接受是异步的，发送者不会关心有无接收者或者何时收到。

  


## 1.7 本地广播

![本地广播](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac1f9173961?w=924&h=291&f=png&s=52161)

- 本地广播机制使得发出的广播只能够在应用程序的内部进行传递，并且广播接收器也只能接受来自本应用程序发出的广播，则安全性得到了提高。
- 本地广播主要是使用了一个 `LocalBroadcastManager` 来对广播进行管理，并提供了发送广播和注册广播接收器的方法。
- 开发者只要实现自己的 `BroadcastReceiver` 子类，并重写 `onReceive ( Context context, Intetn intent )` 方法即可。
- 当其他组件通过 `sendBroadcast()` 、`sendStickyBroadcast()` 、`sendOrderBroadcast()` 方法发送广播消息时，如果该 `BroadcastReceiver` 也对该消息“感兴趣”，`BroadcastReceiver` 的 `onReceive ( Context context, Intetn intent )` 方法将会被触发。
 
- 使用步骤：

1. 调用 LocalBroadcastManager.getInstance() 获得实例
2. 调用 registerReceiver() 方法注册广播
3. 调用 sendBroadcast() 方法发送广播
4. 调用 unregisterReceiver() 方法取消注册

#### 1.7.1 注意事项：

![注意事项](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac21baeb536?w=1167&h=199&f=png&s=35622)

1. 本地广播无法通过静态注册方式来接受，相比起系统全局广播更加高效。
2. 在广播中启动 `Activity` 时，需要为 `Intent` 加入 `FLAG_ACTIVITY_NEW_TASK` 标记，否则会报错，因为需要一个栈来存放新打开的 `Activity` 。
3. 广播中弹出 `Alertdialog` 时，需要设置对话框的类型为 `TYPE_SYSTEM_ALERT` ，否则无法弹出。
4. 不要在 `onReceiver()` 方法中添加过多的逻辑或者进行任何的耗时操作，因为在广播接收器中是不允许开启线程的，当 `onReceiver()` 方法运行了较长时间而没有结束时，程序就会报错。


## 1.8 Sticky Broadcast 粘性广播

![Sticky Broadcast 粘性广播](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac21cbb2dde?w=1050&h=244&f=png&s=50108)

- 如果发送者发送了某个广播，而接收者在这个广播发送后才注册自己的 Receiver ，这时接收者便无法接收到刚才的广播
- 为此 `Android` 引入了 `StickyBroadcast` ，在广播发送结束后会保存刚刚发送的广播（ `Intent` ），这样当接收者注册完 `Receiver` 后就可以继续使用刚才的广播。
- 如果在接收者注册完成前发送了多条相同 `Action` 的粘性广播，注册完成后只会收到一条该 `Action` 的广播，并且消息内容是最后一次广播内容。

- 系统网络状态的改变发送的广播就是粘性广播。
1. 粘性广播通过 `Context` 的 `sendStickyBroadcast ( Intent )`  接口发送，需要添加权限
2. `uses-permission android:name=”android.permission.BROADCAST_STICKY”` 
3. 也可以通过 `Context` 的 `removeStickyBroadcast ( Intent intent )` 接口移除缓存的粘性广播

## 1.9 LocalBroadcastManager 详解

#### 1.9.1 特点：

![LocalBroadcastManager 特点](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac21fcac4e7?w=890&h=228&f=png&s=25298)

1. 使用它发送的广播将只在自身APP内传播，因此你不必担心泄漏隐私数据；

2. 其他 `APP` 无法对你的 `APP` 发送该广播，因为你的APP根本就不可能接收到非自身应用发送的该广播，因此你不必担心有安全漏洞可以利用；

3. 比系统的全局广播更加高效。

#### 1.9.2 源码分析 ：

![LocalBroadcastManager 源码分析](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac246ecc38a?w=1191&h=283&f=png&s=48803)

1. `LocalBroadcastManager` 内部协作主要是靠这两个 `Map` 集合：`MReceivers` 和 `MActions` ，当然还有一个 List 集合 `MPendingBroadcasts` ，这个主要就是存储待接收的广播对象。

2. `LocalBroadcastManager` 高效的原因主要是因为它内部是通过 `Handler` 实现的，它的 `sendBroadcast()` 方法含义并非和我们平时所用的一样，它的 `sendBroadcast()` 方法其实是通过 `handler` 发送一个 `Message` 实现的；

3. 既然它内部是通过 `Handler` 来实现广播的发送的，那么相比于系统广播通过 `Binder` 实现那肯定是更高效了，同时使用 `Handler` 来实现，别的应用无法向我们的应用发送该广播，而我们应用内发送的广播也不会离开我们的应用；


#### 1.9.3 BroadcastReceiver 安全问题

![BroadcastReceiver 安全问题](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac23392df5c?w=933&h=221&f=png&s=39603)

- `BroadcastReceiver` 设计的初衷是从全局考虑可以方便应用程序和系统、应用程序之间、应用程序内的通信，所以对单个应用程序而言`BroadcastReceiver` 是存在安全性问题的 ( 恶意程序脚本不断的去发送你所接收的广播 ) 。为了解决这个问题 `LocalBroadcastManager` 就应运而生了。

- `LocalBroadcastManager` 是 `Android Support` 包提供了一个工具，用于在同一个应用内的不同组件间发送 `Broadcast`。`LocalBroadcastManager` 也称为局部通知管理器，这种通知的好处是安全性高，效率也高，适合局部通信，可以用来代替 `Handler` 更新 `UI` 


#### 1.9.4 广播的安全性

- `Android` 系统中的广播可以跨进程直接通信，会产生以下两个问题：
1. 其他 `APP` 可以接收到当前 `APP` 发送的广播，导致数据外泄。
2. 其他 `APP` 可以向当前 `APP` 放广播消息，导致 `APP` 被非法控制。

![广播的安全性](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac23987ede0?w=748&h=250&f=png&s=35016)

- 发送广播
1. 发送广播时，增加相应的 `permission` ，用于权限验证。
2. 在 `Android 4.0` 及以上系统中发送广播时，可以使用 `setPackage()` 方法设置接受广播的包名。
3. 使用局部广播。

- 接受广播
1. 注册广播接收器时，增加相应的 `permission` ，用于权限验证。
2. 注册广播接收器时，设置 `android:exported` 的值为false。

- 使用局部广播。
1. 发送广播时，如果增加了 `permission`
2. 那接受广播的 `APP` 必须申请相应权限，这样才能收到对应的广播，反之亦然。


#### 1.9.5 使用 BroadcastReceiver 的好处

![BroadcastReceiver 的好处](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac24de70f1f?w=723&h=227&f=png&s=20845)

1. 因广播数据在本应用范围内传播，你不用担心隐私数据泄露的问题。

2. 不用担心别的应用伪造广播，造成安全隐患。

3. 相比在系统内发送全局广播，它更高效。



## 1.10 如何让自己的广播只让指定的 app 接收?

![让自己的广播只让指定的 app 接收](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac260e1e9ea?w=927&h=411&f=png&s=124993)


- 在发送广播的 `app` 端,自定义定义权限, 那么想要接收的另外 `app` 端必须声明权限才能收到.

1. 权限, 保护层级是普通正常.
2. 用户权限

```java
<permission android:name="broad.ok.receiver" android:protectionLevel="normal"/>
<uses-permission android:name="broad.ok.receiver" />
```

3. 发送广播的时候加上权限字符串

```java
public void click(View view) {
    Intent intent = new Intent();
    intent.putExtra("info", "消息内容");
    intent.setAction("myBroadcast.action.call");
    sendBroadcast(intent, "broad.ok.receiver");
    //sendOrderedBroadcast(intent,"broad.ok.receiver");
}
```

4. 其他app接收者想好获取广播,必须声明在清单文件权限

```java
<uses-permission android:name="broad.ok.receiver"/>
```




## 1.11 广播的优先级对无序广播生效吗?

![广播的优先级对无序广播生效](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac263ed4951?w=467&h=142&f=png&s=7956)

- 优先级对无序也生效.



## 1.12 动态注册的广播优先级谁高?

![动态注册的广播优先级谁高](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac27a8a55c2?w=375&h=183&f=png&s=8377)

- 谁先注册,谁就高



## 1.13 如何判断当前的 BrodcastReceiver 接收到的是有序还是无序的广播?

![判断当前的 BrodcastReceiver 接收到的是有序还是无序](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac271038fa4?w=1020&h=199&f=png&s=37088)

- 在 `onReceiver` 方法里,直接调用判断方法得返回值

```java
public void onReceive(Context context, Intent intent) {
    Log.d("MyBroadCastReceiver", "收到信息,内容是 :　" + intent.getStringExtra("info") + "");
    boolean isOrderBroadcast = isOrderedBroadcast();
}
```


## 1.14 BroadcastReceiver 不能执行耗时操作

![BroadcastReceiver 不能执行耗时操作](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac2786df844?w=1017&h=206&f=png&s=27935)

- 一方面
1. `BroadcastReceiver` 一般处于主线程。 
2.  耗时操作会导致 `ANR`
 
- 另一方面
3. `BroadcastReceiver` 启动时间较短。 
4. 如果一个进程里面只存在一个 `BroadcastReceiver` 组件。并且在其中开启子线程执行耗时任务。 
5. 系统会认为该进程是优先级最低的空进程。很容易将其杀死。





# 二、ContentProvider

----

- `ContentProvider` 应用程序间非常通用的共享数据的一种方式，也是 `Android` 官方推荐的方式。
- `Android` 中许多系统应用都使用该方式实现数据共享，比如通讯录、短信等。


![ContentProvider](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac286aa6253?w=1045&h=863&f=png&s=112353)

## 2.1 Android 为什么要设计 ContentProvider 这个组件？

![为什么要设计 ContentProvider](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac28ee68828?w=1078&h=258&f=png&s=55161)

- 很多做 `Android` 开发的人都不怎么使用它，觉得直接读取数据库会更简单方便。
- 那么 `Android` 搞一个内容提供者在数据和应用之间，只是为了装高大上，故弄玄虚？我认为其设计用意在于：

1. 封装。对数据进行封装，提供统一的接口，使用者完全不必关心这些数据是在 `DB` ，`XML` 、`Preferences` 或者网络请求来的。当项目需求要改变数据来源时，使用我们的地方完全不需要修改。
2. 提供一种跨进程数据共享的方式。
3. 应用程序间的数据共享还有另外的一个重要话题，就是数据更新通知机制了。因为数据是在多个应用程序中共享的，当其中一个应用程序改变了这些共享数据的时候，它有责任通知其它应用程序，让它们知道共享数据被修改了，这样它们就可以作相应的处理。


## 2.2 如何访问自定义 ContentProvider

![如何访问自定义 ContentProvider](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac2a001a1bb)

- `ContentResolver` 接口的 `notifyChange` 函数来通知那些注册了监控特定 URI的ContentObserver 对象，使得它们可以相应地执行一些处理。
- ContentObserver 可以通过 registerContentObserver 进行注册。
- 通过 `ContentProvider` 的 `Uri` 访问开放的数据。

1. `ContenResolver` 对象通过 `Context` 提供的方法 `getContenResolver()` 来获得。
2. `ContenResolver` 提供了以下方法来操作：`insert` `delete` `update` `query` 这些方法分别会调用 `ContenProvider` 中与之对应的方法并得到返回的结果。

## 2.3 通过 ContentResolver 获取 ContentProvider 内容的基本步骤

![ContentResolver 获取 ContentProvider 内容的基本步骤](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac2a27690e1?w=924&h=234&f=png&s=24581)

1. 得到 `ContentResolver` 类对象：`ContentResolver cr = getContentResolver ( )`。
2. 定义要查询的字段 `String` 数组。
3. 使用 `cr.query()` ; 返回一个 `Cursor` 对象。
4. 使用 `while` 循环得到 `Cursor` 里面的内容。


## 2.4 ContentProvider 是如何实现数据共享的：

![ContentProvider 是如何实现数据共享的](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac2b28e4802?w=1189&h=332&f=png&s=89191)

- 在 `Android` 中如果想将自己应用的数据 ( 一般多为数据库中的数据 ) 提供给第三发应用, 那么我们只能通过 `ContentProvider` 来实现了。 `ContentProvider` 是应用程序之间共享数据的接口。
- 使用的时候首先自定义 一个类继承 `ContentProvider` , 然后覆写 `query` 、`insert` 、`update` 、`delete` 等 方法。
- 因为其是四大组件之一因此必须在 `AndroidManifest` 文件中进行注册。 
- 把自己的数据通过 `uri` 的形式共享出去 `android` 系统下 不同程序 数据默认是不能共享访问 需要去实现一个类去继承 `ContentProvider`。

```java
public class PersonContentProvider extends ContentProvider{

   public boolean onCreate(){ }
   query(Url, String[], String, String[], String);
   insert(Uri,ContentValues);
   update(Uri,ContentValues,String[]);
   delete(Uri,String,String[]);
   
} 
```


## 2.5 为什么要用 ContentProvider ?它和 sql 的实现上有什么差别?

![为什么要用 ContentProvider ?它和 sql 的实现上有什么差别](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac2b46f5dc8?w=1074&h=207&f=png&s=30641)

- `ContentProvider` 屏蔽了数据存储的细节 , 内部实现对用户完全透明 , 用户只需要关心操作数据的 `uri` 就可以了, `ContentProvider` 可以实现不同 `app`之间 共享。
- `Sql` 也有增删改查的方法, 但是 `sql` 只能查询本应用下的数据库。
- 而 `ContentProvider` 还可以去增删改查本地文件. `xml` 文件的读取等。

## 2.6 Uri 介绍

![Uri 介绍](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac2bcf3fe20?w=1389&h=305&f=png&s=74762)


- 为系统的每一个资源给其一个名字，比方说通话记录。
1. 每一个 `ContentProvider` 都拥有一个公共的 `URI` ，这个 `URI` 用于表示这个 `ContentProvider` 所提供的数据。 
2. `Android` 所提供的 `ContentProvider` 都存放在 `android.provider` 包中。 

- 将其分为 `A，B，C，D` 4个部分：
- `A`：标准前缀，用来说明一个 `Content Provider` 控制这些数据，无法改变的；`"content://"`；
- `B`：`URI` 的标识，用于唯一标识这个 `ContentProvider` ，外部调用者可以根据这个标识来找到它。它定义了是哪个 `ContentProvider` 提供这些数据。对于第三方应用程序，为了保证 `URI` 标识的唯一性，它必须是一个完整的、小写的类名。这个标识在元素的 `authorities` 属性中说明：一般是定义该 `ContentProvider` 的包类的名称；
- `C`：路径（ `path` ），通俗的讲就是你要操作的数据库中表的名字，或者你也可以自己定义，记得在使用的时候保持一致就可以了；`"content://com.bing.provider.myprovider/tablename"`。
- `D`：如果URI中包含表示需要获取的记录的 `ID`；则就返回该id对应的数据，如果没有 `ID`，就表示返回全部； `"content://com.bing.provider.myprovider/tablename/#"` `#` 表示数据 `id` 。


## 2.7 如何访问 asserts 资源目录下的数据库?

![访问 asserts 资源目录下的数据库](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac2c9a5fb7c?w=829&h=184&f=png&s=16001)

- 把数据库 `db` 复制到 `/data/data/packagename/databases/` 目录下, 然后直接就能访问了。
 

## 2.8 多个进程同时调用一个 ContentProvider 的 query 获取数据，ContentPrvoider 是如何反应的呢？

![调用一个 ContentProvider 的 query 获取数据，ContentPrvoider 是如何反应的](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac3174d25e4?w=909&h=229&f=png&s=33902)

- 一个 `ContentProvider` 可以接受来自另外一个进程的数据请求。
- 尽管 `ContentResolver` 与 `ContentProvider` 类隐藏了实现细节，但是 `ContentProvider` 所提供的 `query()`，`insert()`，`delete()`，`update()` 都是在 `ContentProvider` 进程的线程池中被调用执行的，而不是进程的主线程中。
- 这个线程池是有 `Binder` 创建和维护的，其实使用的就是每个应用进程中的 `Binder` 线程池。

## 2.9 Android 设计 ContentProvider 的目的是什么呢？

![设计 ContentProvider 的目的](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac3fe6318d1?w=1163&h=254&f=png&s=48465)

- 隐藏数据的实现方式，对外提供统一的数据访问接口；
- 更好的数据访问权限管理。`ContentProvider` 可以对开发的数据进行权限设置，不同的 `URI` 可以对应不同的权限，只有符合权限要求的组件才能访问到 `ContentProvider` 的具体操作。
- `ContentProvider` 封装了跨进程共享的逻辑，我们只需要 `Uri` 即可访问数据。由系统来管理 `ContentProvider` 的创建、生命周期及访问的线程分配，简化我们在应用间共享数据（ 进程间通信 ）的方式。我们只管通过 `ContentResolver` 访问 `ContentProvider` 所提示的数据接口，而不需要担心它所在进程是启动还是未启动。

## 2.10 运行在主线程的 ContentProvider 为什么不会影响主线程的UI操作?

![ContentProvider 为什么不会影响主线程的UI操作](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac40b1e20e9?w=1288&h=243&f=png&s=43118)

- `ContentProvider` 的 `onCreate()` 是运行在 `UI` 线程的，而 `query()` ，`insert()` ，`delete()` ，`update()` 是运行在线程池中的工作线程的
- 所以调用这向个方法并不会阻塞 `ContentProvider` 所在进程的主线程，但可能会阻塞调用者所在的进程的 `UI` 线程！
- 所以，调用 `ContentProvider` 的操作仍然要放在子线程中去做。
- 虽然直接的 `CRUD` 的操作是在工作线程的，但系统会让你的调用线程等待这个异步的操作完成，你才可以继续线程之前的工作。


## 2.11 外提供数据共享，那么如何限制对方的使用呢？

![如何限制对方的使用](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac4128ba42d?w=1157&h=384&f=png&s=116346)

- `android:exported` 属性非常重要。这个属性用于指示该服务是否能够被其他应用程序组件调用或跟它交互。
- 如果设置为 `true`，则能够被调用或交互，否则不能。
- 设置为 `false` 时，只有同一个应用程序的组件或带有相同用户 `ID` 的应用程序才能启动或绑定该服务。

- 对于需要开放的组件应设置合理的权限，如果只需要对同一个签名的其它应用开放 `ContentProvider` ，则可以设置 `signature` 级别的权限。
- 大家可以参考一下系统自带应用的代码，自定义了 `signature` 级别的 `permission` ：


```java
<permission android:name="com.android.gallery3d.filtershow.permission.READ"
            android:protectionLevel="signature" />

<permission android:name="com.android.gallery3d.filtershow.permission.WRITE"
            android:protectionLevel="signature" />

<provider
    android:name="com.android.gallery3d.filtershow.provider.SharedImageProvider"
    android:authorities="com.android.gallery3d.filtershow.provider.SharedImageProvider"
    android:grantUriPermissions="true"
    android:readPermission="com.android.gallery3d.filtershow.permission.READ"
    android:writePermission="com.android.gallery3d.filtershow.permission.WRITE" />
```

#### 2.11.1 如果我们只需要开放部份的 `URI` 给其他的应用访问呢？

![如果我们只需要开放部份的  URI 给其他的应用访问呢](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac40c1f2e69?w=1081&h=452&f=png&s=144506)

- 可以参考 `Provider` 的 `URI` 权限设置，只允许访问部份 `URI` ，可以参考原生 `ContactsProvider2` 的相关代码（ 注意 `path-permission` 这个选项 ）：

```java
<provider android:name="ContactsProvider2"
    android:authorities="contacts;com.android.contacts"
    android:label="@string/provider_label"
    android:multiprocess="false"
    android:exported="true"
    android:grantUriPermissions="true"
    android:readPermission="android.permission.READ_CONTACTS"
    android:writePermission="android.permission.WRITE_CONTACTS">
    <path-permission
            android:pathPrefix="/search_suggest_query"
            android:readPermission="android.permission.GLOBAL_SEARCH" />
    <path-permission
            android:pathPrefix="/search_suggest_shortcut"
            android:readPermission="android.permission.GLOBAL_SEARCH" />
    <path-permission
            android:pathPattern="/contacts/.*/photo"
            android:readPermission="android.permission.GLOBAL_SEARCH" />
    <grant-uri-permission android:pathPattern=".*" />
</provider>
```

## 2.12 ContentProvider 接口方法运行在哪个线程中呢？

![ContentProvider 接口方法运行在哪个线程中](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac41d929db8?w=1116&h=199&f=png&s=29872)

- `ContentProvider` 可以在 `AndroidManifest.xml` 中配置一个叫做 `android:multiprocess` 的属性，默认值是 false ，表示 ContentProvider 是单例的
- 无论哪个客户端应用的访问都将是同一个 `ContentProvider` 对象，如果设为 `true` ，系统会为每一个访问该 `ContentProvider` 的进程创建一个实例。

#### 2.12.1 这点还是比较好理解的，那如果我要问每个 ContentProvider 的操作是在哪个线程中运行的呢?（ 其实我们关心的是 UI 线程和工作线程 ）

![每个 ContentProvider 的操作是在哪个线程中运行的](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac42a37490f?w=1493&h=231&f=png&s=57634)

- 比如我们在UI线程调用getContentResolver().query查询数据，而当数据量很大时（或者需要进行较长时间的计算）会不会阻塞UI线程呢？

- 要分两种情况回答这个问题：

1. `ContentProvider` 和调用者在同一个进程，`ContentProvider` 的方法（ `query/insert/update/delete` 等 ）和调用者在同一线程中；
2. `ContentProvider` 和调用者在不同的进程，`ContentProvider` 的方法会运行在它自身所在进程的一个 Binder 线程中。
但是，注意这两种方式在 `ContentProvider` 的方法没有执行完成前都会 `blocked` 调用者。所以你应该知道这个上面这个问题的答案了吧。
3. 也可以看看 `CursorLoader` 这个类的源码，看 `Google` 自己是怎么使用 `getContentResolver().query` 的。

## 2.13 ContentProvider 是如何在不同应用程序之间传输数据的？

![ContentProvider 是如何在不同应用程序之间传输数据](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac434314bde?w=1135&h=198&f=png&s=34663)

- 一个应用进程有 `16` 个 `Binder` 线程去和远程服务进行交互，而每个线程可占用的缓存空间是 `128KB` 这样，超过会报异常。
- `ContentResolver` 虽然是通过 `Binder` 进程间通信机制打通了应用程序之间共享数据的通道，但 `ContentProvider` 组件在不同应用程序之间传输数据是基于匿名共享内存机制来实现的。

# 总结

----

1. 本文应该是一篇非常全面的 `BroadcastReceiver`  知识总结了，如果还有什么可以补充的知识点，欢迎大家在评论区指出。
2. 前前后后投入了大量时间来完成。希望大家通过本次阅读都能有所收获。
2. **`重点`**：关于 `Android` 的四大组件，到现在为止我才总结完 `Activity` 、`Service` 、`BroadcastRecevier` 等，有关 事件分发、滑动冲突、新能优化等重要模块，我也将进行详尽的总结，欢迎大家关注 [_yuanhao 的 掘金](https://juejin.im/user/5d00b2ee6fb9a07ef5622eed) ，方便及时接收更新

# 码字不易，你的点赞是我总结的最大动力！

----

- 由于我在「稀土掘金」「简书」「`CSDN`」「博客园」等站点，都有新内容发布。所以大家可以直接关注我的 `GitHub` 仓库，以免错过精彩内容！

- 仓库地址：
[超级干货！精心归纳 `Android` 、`JVM` 、算法等，各位帅气的老铁支持一下！给个 Star ！](https://github.com/FishInWater-1999/android_interviews)

- 一万多字长文，加上精美思维导图，**记得点赞哦**，欢迎关注 **[_yuanhao 的 掘金](https://juejin.im/user/5d00b2ee6fb9a07ef5622eed)** ，我们下篇文章见！

![Android](https://user-gold-cdn.xitu.io/2019/11/8/16e48ac440026e54?w=2000&h=1215&f=jpeg&s=190057)
