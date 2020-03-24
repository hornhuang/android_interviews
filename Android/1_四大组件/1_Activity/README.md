![Android](https://img-blog.csdnimg.cn/20191029235433160.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzc3NzQ5,size_16,color_FFFFFF,t_70)

# 前言
学 `Android` 有一段时间了，一直都只顾着学新的东西，最近发现很多平常用的少的东西竟让都忘了，趁着这两天，打算把有关 `Activity` 的内容以问题的形式梳理出来，也供大家查缺补漏。

> 本文中，我将一改往日写博客的习惯，全文用 XMind 将所有知识点以思维导图的形式呈现，欢迎大家食用～～



# 文章目录
----

![文章目录](https://img-blog.csdnimg.cn/20191030000453730.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzc3NzQ5,size_16,color_FFFFFF,t_70)


# 神图
------

- 在开始之前，先让我们看看 `Android` 的 `activity` 到底都有哪些东西？
- 借一张网上很火的图带你了解 `Activity`

![一张图带你了解 Activity](https://img-blog.csdnimg.cn/20191024182225606.png)



# 1. 生命周期
----

- 先贴一张闻名遐迩的图
- 我们生命周期先看看具体有哪些方法回调，在逐一攻破：

![生命周期](https://img-blog.csdnimg.cn/20191024182604463.png)

## 1.1 Dialog 弹出时

![dialog弹出时](https://img-blog.csdnimg.cn/20191024194442984.png)

- 如果是单纯是创建的 `dialog` ，`Activity` 并不会执行生命周期的方法
- 但是如果是跳转到一个不是全屏的 `Activity` 的话, 当然就是按照正常的生命周期来执行了
- 即 `onPasue()` -> `onStop()`

## 1.2 横竖屏切换时

![横竖屏切换时](https://img-blog.csdnimg.cn/20191024194737383.png)

- 不设置 `Activity` 的 `android:configChanges` 时，切屏会重新调用各个生命周期，切横屏时会执行一次，切竖屏时会执行两次
- 设置 `Activity` 的 `android:configChanges="orientation"` 时，切屏还是会重新调用各个生命周期，切横、竖屏时只会执行一次
- 设置 `Activity` 的 `android:configChanges="orientation|keyboardHidden"` 时，切屏不会重新调用各个生命周期，只会执行 `onConfigurationChanged` 方法
- 注意：还有一点，非常重要，一个 `Android` 的变更细节！当 `API >12` 时，需要加入 `screenSize` 属性，否则屏幕切换时即使你设置了 `orientation` 系统也会重建 `Activity` ！

[横竖屏切换生命周期的执行](https://developer.android.com/guide/topics/resources/runtime-changes.html)


## 1.3 不同场景下 Activity 生命周期的变化过程

![不同场景下Activity生命周期的变化过程](https://img-blog.csdnimg.cn/20191024195628863.png)

- 启动 `Activity` ： `onCreate()` ---> `onStart()` ---> `onResume()` ，`Activity` 进入运行状态。
- 锁定屏与解锁屏幕：只会调用 `onPause()` ，而不会调用 `onStop` 方法，开屏后则调用 `onResume()`

![已启动的 Activity 生命周期的变化](https://img-blog.csdnimg.cn/20191024195708285.png)

- `Activity` 退居后台： 当前 `Activity` 转到新的 `Activity` 界面或按 `Home` 键回到主屏： `onPause()` ---> `onStop()` ，进入停滞状态。
- `Activity` 返回前台： `onRestart()` ---> `onStart()` ---> `onResume()` ，再次回到运行状态。
- `Activity` 退居后台： 且系统内存不足， 系统会杀死这个后台状态的 `Activity` ，若再次回到这个 `Activity` ,则会走 `onCreate()` --> `onStart()` ---> `onResume()`

## 1.4 将一个 Activity 设置成窗口的样式

![设置 Activity 成窗口样式](https://img-blog.csdnimg.cn/20191024195851984.png)

只需要给我们的 `Activity` 配置如下属性即可。
`android:theme="@android:style/Theme.Dialog"`


## 1.5 退出已调用多个 Activity 的 Application

- 通常情况用户退出一个 `Activity` 只需按返回键,我们写代码想退出 `activity` 直接调用 `finish()` 方法就行。

![退出调用多个 Activity 的 Application](https://img-blog.csdnimg.cn/20191024200608571.png)

- 发送特定广播:
1. 在需要结束应用时, 发送一个特定的广播,每个 `Activity` 收到广播后,关闭 即可。
2. 给某个 `activity` 注册接受接受广播的意图 `registerReceiver(receiver, filter)`
3. 如果过接受到的是 关闭 `activity` 的广播 `activity finish()` 掉

- 递归退出
1. 就调用 `finish()` 方法 把当前的 `Activity` 退出
2. 在打开新的 `Activity` 时使用 `startActivityForResult` , 然后自己加标志, 在 `onActivityResult` 中处理, 递归关闭。

- 其实 
1. 也可以通过 `intent` 的 `flag` 来实现 `intent.setFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP)` 激活一个新的 `activity`。 
2. 此时如果该任务栈中已经有该 `Activity` , 那么系统会把这个 `Activity` 上面的所有 `Activity` 干掉。
3. 其实相当于给 `Activity` 配置的启动模式为 `SingleTop` 。

- 记录打开的 `Activity` 
1. 每打开一个 `Activity` , 就记录下来。
2. 在需要退出时 , 关闭每一个 `Activity`

## 1.6 锁定屏与解锁屏幕，Activity 如何执行生命周期

![锁定屏与解锁屏幕，Activity如何执行生命周期](https://img-blog.csdnimg.cn/20191024200813704.png)

- 只会调用 `onPause()` ，而不会调用 `onStop` 方法，开屏后则调用 `onResume()` 。


## 1.7 修改 Activity 进入和退出动画

![修改 Activity 进入和退出动画](https://img-blog.csdnimg.cn/20191024201005845.png)

- 可以通过两种方式 ， 一是通过定义 `Activity` 的主题 ，二是通过覆写 `Activity` 的 `overridePendingTransition` 方法。
- 通过设置主题样式在 `styles.xml` 中编辑代码 ,  添加 `themes.xml` 文件：在 `AndroidManifest.xml` 中给指定的 `Activity` 指定 `theme`。
- 覆写 `overridePendingTransition` 方法：`overridePendingTransition(R.anim.fade, R.anim.hold)`; 


## 1.8 Activity 的四种状态
![四种状态](https://img-blog.csdnimg.cn/20191024201502443.png)

- `runnig` ：用户可以点击，`activity` 处于栈顶状态。
- `paused` ：`activity` 失去焦点的时候，被一个非全屏的 `activity` 占据或者被一个透明的 `activity` 覆盖，这个状态的 `activity` 并没有销毁，它所有的状态信息和成员变量仍然存在，只是不能够被点击。(内存紧张的情况，这个 `activity` 有可能被回收)

![关闭](https://img-blog.csdnimg.cn/20191024201536153.png)

- `stopped` ：这个 `activity` 被另外一个 `activity` 完全覆盖，但是这个 `activity` 的所有状态信息和成员变量仍然存在(除了内存紧张)
- `killed` ：这个 `activity` 已经被销毁，其所有的状态信息和成员变量已经不存在了。


## 1.9 如何处理异常退出

![如何处理异常退出](https://img-blog.csdnimg.cn/20191024201824767.png)

- `Activity` 异常退出的时候 --> `onPause()` --> `onSaveInstanceState()` --> `onStop()` --> `onDestory()`
- 需要注意的是 `onSaveInstanceState()` 方法与 `onPause` 并没有严格的先后关系，有可能在 `onPause` 之前，也有可能在其后面调用，但会在 `onStop()` 方法之前调用
- 异常退出后又重新启动该 `Activity`  --> `onCreate()` --> `onStart()` --> `onRestoreInstanceState()` --> `onResume()`

![异常退出后又重新启动](https://img-blog.csdnimg.cn/20191024201956931.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzc3NzQ5,size_16,color_FFFFFF,t_70)

- 搞懂这个生命周期的执行后就可以回答了，首先要知道面试官的意思：是要重新启动并恢复这个 `Activity` 还是说直接退出整个 `app` 
- 如果要恢复则要在 `onSaveInstanceState()` 中进行保存数据并在 `onRestoreInstanceState()` 中进行恢复
- 如果是要退出 `app` 的话就要捕获全局的异常信息，并退出 `app` 
- 当然个人建议是使用 `UncaughtExceotionHandler` 来捕获全局异常进行退出 `app` 的操作，这样会减少之前崩溃所造成的后遗症！

## 1.10 什么是 onNewIntent

![onNewIntent](https://img-blog.csdnimg.cn/20191024202053854.png)

- 如果 `IntentActivity` 处于任务栈的顶端，也就是说之前打开过的 `Activity` ，现在处于 `onPause` 、 `onStop` 状态的话，其他应用再发送 `Intent` 的话

- 执行顺序为：`onNewIntent`，`onRestart`，`onStart`，`onResume`。






# 2. 启动模式

----




## 2.1 启动模式
 ![启动模式](https://img-blog.csdnimg.cn/20191024202940255.png)

- `Activity` 一共有四种 `launchMode` ：`standard` 、`singleTop` 、`singleTask` 、`singleInstance` 。

![Standard](https://img-blog.csdnimg.cn/2019102420313232.png)

- `Standard` 模式（默认模式）

1. 说明： 每次启动一个 `Activity` 都会又一次创建一个新的实例入栈，无论这个实例是否存在。

2. 生命周期：每次被创建的实例 `Activity` 的生命周期符合典型情况，它的 `onCreate` 、`onStart` 、`onResume` 都会被调用。

3. 举例：此时 `Activity` 栈中以此有 `A` 、`B` 、`C` 三个 `Activity` ，此时C处于栈顶，启动模式为 `Standard` 模式。若在 `C Activity` 中加入点击事件，须要跳转到还有一个同类型的 `C Activity` 。结果是还有一个 `C Activity` 进入栈中，成为栈顶。


![SingleTop](https://img-blog.csdnimg.cn/20191024203443944.png)

- `SingleTop` 模式（栈顶复用模式）

1. 说明：分两种处理情况：须要创建的 `Activity` 已经处于栈顶时，此时会直接复用栈顶的 `Activity` 。不会再创建新的 `Activity` ；若须要创建的 `Activity` 不处于栈顶，此时会又一次创建一个新的 `Activity` 入栈，同 `Standard` 模式一样。

2. 生命周期：若情况一中栈顶的 `Activity` 被直接复用时，它的 `onCreate` 、`onStart` 不会被系统调用，由于它并没有发生改变。可是一个新的方法 `onNewIntent` 会被回调（ `Activity` 被正常创建时不会回调此方法）。

3. 举例：此时 `Activity` 栈中以此有 `A` 、`B` 、`C` 三个 `Activity` ，此时 `C` 处于栈顶，启动模式为 `SingleTop` 模式。情况一：在 `C Activity` 中加入点击事件，须要跳转到还有一个同类型的 `C Activity` 。结果是直接复用栈顶的 `C Activity`。情况二：在 `C Activity` 中加入点击事件，须要跳转到还有一个 `A Activity`。结果是创建一个新的 `Activity` 入栈。成为栈顶。

![SingleTask](https://img-blog.csdnimg.cn/20191024203705405.png)


- `SingleTask` 模式（栈内复用模式）

1. 说明：若须要创建的 `Activity` 已经处于栈中时，此时不会创建新的 `Activity` ，而是将存在栈中的 `Activity` 上面的其他 `Activity` 所有销毁，使它成为栈顶。

2. 如果是在别的应用程序中启动它，则会新建一个 `task` ，并在该task中启动这个 `Activity` ，`singleTask` 允许别的 `Activity` 与其在一个 `task` 中共存，也就是说，如果我在这个 `singleTask` 的实例中再打开新的 `Activity` ，这个新的 `Activity` 还是会在 `singleTask` 的实例的 `task` 中。

3. 生命周期：同 `SingleTop` 模式中的情况一同样。仅仅会又一次回调 `Activity` 中的 `onNewIntent` 方法

4. 举例：此时 `Activity` 栈中以此有 `A` 、`B` 、`C` 三个 `Activity` 。此时 `C` 处于栈顶，启动模式为 `SingleTask` 模式。情况一：在 `C Activity` 中加入点击事件，须要跳转到还有一个同类型的 `C Activity` 。结果是直接用栈顶的 `C Activity` 。情况二：在 `C Activity` 中加入点击事件，须要跳转到还有一个 `A Activity` 。结果是将 `A Activity` 上面的 `B` 、`C` 所有销毁，使 `A Activity` 成为栈顶。


![ingleInstance](https://img-blog.csdnimg.cn/20191024204027755.png)


- `SingleInstance` 模式（单实例模式）

1. 说明： `SingleInstance` 比较特殊，是全局单例模式，是一种加强的 `SingleTask` 模式。它除了具有它所有特性外，还加强了一点：只有一个实例，并且这个实例独立运行在一个 `task` 中，这个 `task` 只有这个实例，不允许有别的 `Activity` 存在。

2. 这个经常使用于系统中的应用，比如 `Launch` 、锁屏键的应用等等，整个系统中仅仅有一个！所以在我们的应用中一般不会用到。了解就可以。

3. 举例：比方 `A Activity` 是该模式，启动 `A` 后。系统会为它创建一个单独的任务栈，由于栈内复用的特性。兴许的请求均不会创建新的 `Activity` ，除非这个独特的任务栈被系统销毁。









## 2.2 启动模式的使用方式

![启动模式的使用方式](https://img-blog.csdnimg.cn/20191024204248613.png)

- 在 `Manifest.xml` 中指定 `Activity` 启动模式

1. 一种静态的指定方法
2. 在 `Manifest.xml` 文件里声明 `Activity` 的同一时候指定它的启动模式
3. 这样在代码中跳转时会依照指定的模式来创建 `Activity` 。

- 启动 `Activity` 时。在 `Intent` 中指定启动模式去创建 `Activity`

1. 一种动态的启动模式
2. 在 `new` 一个 `Intent` 后
3. 通过 `Intent` 的 `addFlags` 方法去动态指定一个启动模式。


- 注意：以上两种方式都能够为 `Activity` 指定启动模式，可是二者还是有差别的。

1. 优先级：动态指定方式即另外一种比第一种优先级要高，若两者同一时候存在，以另外一种方式为准。

2. 限定范围：第一种方式无法为 `Activity` 直接指定 `FLAG_ACTIVITY_CLEAR_TOP` 标识，另外一种方式无法为 `Activity` 指定 `singleInstance` 模式。



## 2.3 启动模式的实际应用场景

> 这四种模式中的 `Standard` 模式是最普通的一种，没有什么特别注意。而 `SingleInstance` 模式是整个系统的单例模式，在我们的应用中一般不会应用到。所以，这里就具体解说  `SingleTop` 和 `SingleTask` 模式的运用场景：

![启动模式的实际应用场景](https://img-blog.csdnimg.cn/20191024204743619.png)


- `SingleTask` 模式的运用场景

1. 最常见的应用场景就是保持我们应用开启后仅仅有一个 `Activity` 的实例。
2. 最典型的样例就是应用中展示的主页（ `Home` 页）。
3. 假设用户在主页跳转到其他页面，运行多次操作后想返回到主页，假设不使用 `SingleTask` 模式，在点击返回的过程中会多次看到主页，这明显就是设计不合理了。

- `SingleTop` 模式的运用场景

1. 假设你在当前的 `Activity` 中又要启动同类型的 `Activity` 
2. 此时建议将此类型 `Activity` 的启动模式指定为 `SingleTop` ，能够降低Activity的创建，节省内存！

- 注意：复用 `Activity` 时的生命周期回调

1. 这里还须要考虑一个 `Activity` 跳转时携带页面參数的问题。
2. 由于当一个 `Activity` 设置了 `SingleTop` 或者 `SingleTask` 模式后，跳转此 `Activity` 出现复用原有 `Activity` 的情况时，此 `Activity` 的 `onCreate` 方法将不会再次运行。`onCreate` 方法仅仅会在第一次创建 `Activity` 时被运行。
3. 而一般 `onCreate` 方法中会进行该页面的数据初始化、`UI` 初始化，假设页面的展示数据无关页面跳转传递的參数，则不必操心此问题
4. 若页面展示的数据就是通过 `getInten()` 方法来获取，那么问题就会出现：`getInten()` 获取的一直都是老数据，根本无法接收跳转时传送的新数据！

- 以下，通过一个样例来具体解释：

![图片描述](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcubXVrZXdhbmcuY29tLzVhYzljMzE0MDAwMTNjZDIwNDc0MDYwNC5wbmc?x-oss-process=image/format,png)

- 以上代码中的 `CourseDetailActivity` 在配置文件里设置了启动模式是 `SingleTop` 模式，依据上面启动模式的介绍可得知，当 `CourseDetailActivity` 处于栈顶时。
- 再次跳转页面到 `CourseDetailActivity` 时会直接复用原有的 `Activity` ，并且此页面须要展示的数据是从 `getIntent()` 方法得来，可是 `initData()` 方法不会再次被调用，此时页面就无法显示新的数据。

- 当然这样的情况系统早就为我们想过了，这时我们须要另外一个回调 `onNewIntent（Intent intent）`方法。此方法会传入最新的 `intent` ，这样我们就能够解决上述问题。这里建议的方法是又一次去 `setIntent` 。然后又一次去初始化数据和 `UI` 。代码例如以下所看到的：

![图片描述](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcubXVrZXdhbmcuY29tLzVhYzljMzliMDAwMWE1ZTkwMzc3MDI0Mi5wbmc?x-oss-process=image/format,png)

- 这样，在一个页面中能够反复跳转并显示不同的内容。



## 2.4 快速启动一个 Activity

![快速启动一个 Activity](https://img-blog.csdnimg.cn/20191024204848993.png)

- 这个问题其实也是比较简单的，就是不要在 `Activity` 的 `onCreate` 方法中执行过多繁重的操作，并且在 `onPasue` 方法中同样不能做过多的耗时操作。


## 2.5 启动流程

- 注意！这里并不是要回答 `Activity` 的生命周期！

- [3 分钟看懂 `Activity` 启动流程](https://juejin.im/entry/58f5b68e61ff4b005807ab47)



## 2.6 Activity 的 Flags

![Activity 的 Flags](https://img-blog.csdnimg.cn/20191024205623940.png)

- 标记位既能够设定Activity的启动模式，如同上面介绍的，在动态指定启动模式，比方 `FLAG_ACTIVITY_NEW_TASK` 和 `FLAG_ACTIVITY_SINGLE_TOP` 等。它还能够影响 `Activity` 的运行状态 ，比方 `FLAG_ACTIVITY_CLEAN_TOP` 和 `FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS` 等。

- 以下介绍几个基本的标记位，切勿死记，理解几个就可以，须要时再查官方文档。

![几个基本的标记位](https://img-blog.csdnimg.cn/20191024205814295.png)

- `FLAG_ACTIVITY_NEW_TASK`

1. 作用是为 `Activity` 指定 `“SingleTask”` 启动模式。跟在 `AndroidMainfest.xml` 指定效果同样

- `FLAG_ACTIVITY_SINGLE_TOP`

1. 作用是为 `Activity` 指定 `“SingleTop”` 启动模式，跟在 `AndroidMainfest.xml` 指定效果同样。

- `FLAG_ACTIVITY_CLEAN_TOP`

1. 具有此标记位的 `Activity` ，启动时会将与该 `Activity` 在同一任务栈的其他 `Activity` 出栈。
2. 一般与 `SingleTask` 启动模式一起出现。
3. 它会完毕 `SingleTask` 的作用。
4. 但事实上 `SingleTask` 启动模式默认具有此标记位的作用

- `FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS`

1. 具有此标记位的 `Activity` 不会出如今历史 `Activity` 的列表中
2. 使用场景：当某些情况下我们不希望用户通过历史列表回到 `Activity` 时，此标记位便体现了它的效果。
3. 它等同于在 `xml` 中指定 `Activity` 的属性.



## 2.7 onNewInstent()方法什么时候执行

![onNewInstent()方法什么时候执行](https://img-blog.csdnimg.cn/20191024205057467.png)

这个是启动模式中的了，当此 `Activity` 的实例已经存在，并且此时的启动模式为 `SingleTask` 和 `SingleInstance` ，另外当这个实例位于栈顶且启动模式为 `SingleTop` 时也会触发 `onNewInstent()` 。




# 3. 数据
----

## 3.1 Activity 间通过 Intent 传递数据大小限制

![Activity 间通过 Intent 传递数据大小限制](https://img-blog.csdnimg.cn/20191027150103249.png)

- `Intent` 在传递数据时是有大小限制的，这里官方并未详细说明，不过通过实验的方法可以测出数据应该被限制在 `1MB` 之内（ `1024KB` ）
- 我们采用传递 `Bitmap` 的方法，发现当图片大小超过 `1024`（准确地说是 `1020` 左右）的时候，程序就会出现闪退、停止运行等异常(不同的手机反应不同)
- 因此可以判断 `Intent` 的传输容量在 `1MB` 之内。



## 3.2 内存不足时系统会杀掉后台的Activity，若需要进行一些临时状态的保存，在哪个方法进行

![onSaveInstanceState()](https://img-blog.csdnimg.cn/20191027150813475.png)

- `Activity` 的 `onSaveInstanceState()` 和 `onRestoreInstanceState()` 并不是生命周期方法，它们不同于 `onCreate()` 、`onPause()` 等生命周期方法，它们并不一定会被触发。

- `onSaveInstanceState()` 方法，当应用遇到意外情况（如：内存不足、用户直接按 `Home` 键）由系统销毁一个 `Activity` ，`onSaveInstanceState()` 会被调用。
- 但是当用户主动去销毁一个 `Activity` 时，例如在应用中按返回键，`onSaveInstanceState()` 就不会被调用。
- 除非该 `activity` 不是被用户主动销毁的，通常 `onSaveInstanceState()` 只适合用于保存一些临时性的状态，而 `onPause()` 适合用于数据的持久化保存。

## 3.3 onSaveInstanceState() 被执行的场景

![onSaveInstanceState() 被执行的场景](https://img-blog.csdnimg.cn/20191027151028772.png)

- 系统不知道你按下 `HOME` 后要运行多少其他的程序，自然也不知道 `activity A` 是否会被销毁
- 因此系统都会调用 `onSaveInstanceState()` ，让用户有机会保存某些非永久性的数据。以下几种情况的分析都遵循该原则：
1. 当用户按下 `HOME` 键时
2. 长按 `HOME` 键，选择运行其他的程序时
3. 锁屏时
4. 从 `activity A` 中启动一个新的 `activity` 时
5. 屏幕方向切换时


## 3.4 两个 Activity 之间跳转时必然会执行的方法

![两个 Activity 之间跳转时必然会执行的方法](https://img-blog.csdnimg.cn/20191028094936943.png)

一般情况下比如说有两个 `activity` , 分别叫 `A` , `B` ,当在 `A` 里面激活 `B` 组件的时候, `A` 会调用 `onPause()` 方法,然后 `B` 调用 `onCreate()` , `onStart()` , `onResume()` 。

这个时候 `B` 覆盖了窗体, `A` 会调用 `onStop()` 方法. 如果 `B` 是个透明的,或者 是对话框的样式, 就不会调用 `A` 的 `onStop()` 方法。

## 3.5 用 Intent 去启动一个Activity 之外的方法

![用 Intent 去启动一个Activity 之外的方法](https://img-blog.csdnimg.cn/2019102809534584.png)

- 使用 `adb shell am` 命令
1. `am` 启动一个 `activity`
2. `adb shell am start com.example.fuchenxuan/.MainActivity`
3. `am` 发送一个广播，使用 `action`
4. `adb shell am broadcast -a magcomm.action.TOUCH_LETTER`

## 3.6 scheme 跳转协议

![scheme跳转协议](https://img-blog.csdnimg.cn/2019102810001391.png)

#### 3.6.1 定义

![定义](https://img-blog.csdnimg.cn/20191029233104733.png)

- 服务器可以定制化跳转 `app` 页面

- `app` 可以通过 `Scheme` 跳转到另一个 `app` 页面

- 可以通过 `h5` 页面跳转 `app` 原生页面

#### 3.6.2 协议格式：

![协议格式](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcubXVrZXdhbmcuY29tLzVhYzljNjJkMDAwMWQ0NjgwNjE1MDA1MC5wbmc?x-oss-process=image/format,png)

![协议格式](https://img-blog.csdnimg.cn/20191029233514274.png)

- `qh` 代表 `Scheme` 协议名称

- `test` 代表 `Scheme` 作用的地址域

- `8080` 代表改路径的端口号

- `/goods` 代表的是指定页面(路径)

- `goodsId` 和 `name` 代表传递的两个参数

#### 3.6.3 Scheme使用

- 定义一个 `Scheme`

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcubXVrZXdhbmcuY29tLzVhYzljNzE4MDAwMWE4YTEwNzkwMDQwMy5wbmc?x-oss-process=image/format,png)

- 获取 `Scheme` 跳转的参数


![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcubXVrZXdhbmcuY29tLzVhYzljOTRkMDAwMWM1ZmQwNjg1MDQ4NC5wbmc?x-oss-process=image/format,png)

- 调用方式

1. 原生调用


![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcubXVrZXdhbmcuY29tLzVhYzljN2FkMDAwMTMyNDgwOTQ5MDA0NC5wbmc?x-oss-process=image/format,png)

2. html调用


![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcubXVrZXdhbmcuY29tLzVhYzljN2RmMDAwMTkzODEwNTgyMDAzMC5wbmc?x-oss-process=image/format,png)

3. 判断某个Scheme是否有效


![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcubXVrZXdhbmcuY29tLzVhYzljODVhMDAwMWRhODEwOTc2MDEwNi5wbmc?x-oss-process=image/format,png)

- 关于scheme跳转协议，可以查看下面的博客，站在巨人的肩膀上，才能看得更远
[Android产品研发（十一）-->应用内跳转Scheme协议](https://link.jianshu.com/?t=http://blog.csdn.net/qq_23547831/article/details/51685310)


<br>


# 4. Context
----


## 4.1 Context , Activity , Appliction 的区别

![Context, Activity, Appliction 的区别](https://img-blog.csdnimg.cn/20191028100354753.png)

- 相同:`Activity` 和 `Application` 都是 `Context` 的子类。
- `Context` 从字面上理解就是上下文的意思, 在实际应用中它也确实是起到了管理 上下文环境中各个参数和变量的总用, 方便我们可以简单的访问到各种资源。
- 不同:维护的生命周期不同。`Context` 维护的是当前的 `Activity` 的生命周期, `Application` 维护的是整个项目的生命周期。
- 使用 `context` 的时候,  小心内存泄露, 防止内存泄露


## 4.2 Context 是什么

![Context 是什么](https://img-blog.csdnimg.cn/20191028100926104.png)

- 它描述的是一个应用程序环境的信息,即上下文。

- 该类是一个抽象( `abstract class` )类,  `Android` 提供了该抽象类的具体实 现类( `ContextIml` )。

- 通过它我们可以获取应用程序的资源和类, 也包括一些应用级别操作,  例如:启动一个 `Activity` ,发送广播,接受 `Intent` ,信息,等。

#### 4.2.1 附加一张 Context 继承关系图


![Context继承关系图](https://img-blog.csdnimg.cn/20191024184446554.png)


## 4.3 获取当前屏幕 Activity 的对象

![获取当前屏幕Activity的对象](https://img-blog.csdnimg.cn/20191028101904310.png)

- 使用 ActivityLifecycleCallbacks
[Android 如何获取当前Activity实例对象？](https://link.jianshu.com/?t=http://blog.csdn.net/vfush/article/details/51483436)










## 4.4 Activity 的管理机制

![Activity的管理机制](https://img-blog.csdnimg.cn/20191029155048429.png)

- [Activity的管理机制](https://www.jianshu.com/p/33729a7a66da)

- 面试官问这个问题，想看看大家对Activity了解是否深入：
1. 什么是 ActivityRecord
2. 什么是 TaskRecord
3. 什么是 ActivityManagerService



## 4.5 什么是 Activity

![什么是 Activity](https://img-blog.csdnimg.cn/20191029155224987.png)

- 四大组件之一，通常一个用户交互界面对应一个 `activity` 。
- `activity` 是 `Context` 的子类，同时实现了 `window.callback` 和 `keyevent.callback` ，可以处理与窗体用户交互的事件。
- 开发中常用的有 `FragmentActivity` 、`ListActivity` 、`TabActivity`（ `Android 4.0` 被 `Fragment` 取代）







# 5. 进程
----


## 5.1 Android 进程优先级

- 前台 / 可见 / 服务 / 后台 / 空

![前台 / 可见 / 服务 / 后台 / 空](https://img-blog.csdnimg.cn/20191029195312926.png)

#### 5.1.1 前台进程：Foreground process

![前台进程：Foreground process](https://img-blog.csdnimg.cn/20191029195950457.png)

- 用户正在交互的 `Activity`（ `onResume()` ）
- 当某个 `Service` 绑定正在交互的 `Activity`
- 被主动调用为前台 `Service`（ `startForeground()` ）
- 组件正在执行生命周期的回调（ `onCreate()` 、`onStart()` 、`onDestory()` ）
- `BroadcastReceiver` 正在执行 `onReceive()`

#### 5.1.2 可见进程：Visible process

![可见进程：Visible process](https://img-blog.csdnimg.cn/20191029200049245.png)

- 我们的 `Activity` 处在 `onPause()`（没有进入 `onStop()` ）
- 绑定到前台 `Activity` 的 `Service`

#### 5.1.3 服务进程：Service process

![服务进程](https://img-blog.csdnimg.cn/20191029200134749.png)

- 简单的 `startService()` 启动。

#### 5.1.4 后台进程：Background process

![后台进程：Background process](https://img-blog.csdnimg.cn/20191029200217348.png)

- 对用户没有直接影响的进程 --- `Activity` 处于 `onStop()` 的时候。
- `android:process=":xxx"`

#### 5.1.5 空进程：Empty process

![空进程：Empty process](https://img-blog.csdnimg.cn/20191029200258263.png)

- 不含有任何的活动的组件。（ `Android` 设计的，处于缓存的目的，为了第二次启动更快，采取的一个权衡）

## 5.2 可见进程

![可见进程](https://img-blog.csdnimg.cn/20191029201755712.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzc3NzQ5,size_16,color_FFFFFF,t_70)

> 可见进程指部分程序界面能够被用户看见，却不在前台与用户交互的进程。例如，我们在一个界面上弹出一个对话框（该对话框是一个新的 `Activity` ），那么在对话框后面的原界面是可见的，但是并没有与用户进行交互，那么原界面就是可见进程。

- 一个进程满足下面任何一个条件都被认为是可视的：
1. 寄宿着一个不是前台的活动，但是它对用户仍可见（它的 `onPause()` 方法已经被调用）。举例来说，这可能发生在，如果一个前台活动在一个对话框（其他进程的）运行之后仍然是可视的，比如输入法的弹出时。
2. 寄宿着一个服务，该服务绑定到一个可视的活动。

- 一个可视进程被认为是及其重要的且不会被杀死，除非为了保持前台进程运行。


## 5.3 服务进程

![服务进程](https://img-blog.csdnimg.cn/20191029201840820.png)

- 服务进程是通过 `startService()` 方法启动的进程，但不属于前台进程和可见进程。例如，在后台播放音乐或者在后台下载就是服务进程。

- 系统保持它们运行，除非没有足够内存来保证所有的前台进程和可视进程。



## 5.4 后台进程

![后台进程](https://img-blog.csdnimg.cn/20191029202122401.png)

- 后台进程是一个保持着一个当前对用户不可视的活动（已经调用 `Activity` 对象的 `onStop()` 方法）（如果还有除了 `UI` 线程外其他线程在运行话，不受影响）。

> 例如我正在使用 `qq` 和别人聊天，这个时候 `qq` 是前台进程，但是当我点击 `Home` 键让 `qq` 界面消失的时候，这个时候它就转换成了后台进程。

- 这些进程没有直接影响用户体验，并且可以在任何时候被杀以收回内存用于一个前台、可视、服务进程。
- 一般地有很多后台进程运行着，因此它们保持在一个 `LRU`（ `least recently used` ，即最近最少使用，如果您学过操作系统的话会觉得它很熟悉，跟内存的页面置换算法 `LRU` 一样）列表以确保最近使用最多的活动的进程最后被杀。


## 5.5 空进程

![空进程](https://img-blog.csdnimg.cn/20191029202608803.png)

- 空进程是一个没有保持活跃的应用程序组件的进程，不包含任何活跃组件。

- 保持这个进程可用的唯一原因是作为一个 `cache` 以提高下次启动组件的速度。系统进程杀死这些进程，以在进程 `cache` 和潜在的内核 `cache` 之间平衡整个系统资源。

- `android` 进程的回收顺序从先到后分别是：空进程，后台进程，服务进程，可见进程，前台进程。


## 5.6 什么是 ANR，如何避免

![什么是 ANR，如何避免](https://img-blog.csdnimg.cn/20191029202839860.png)

#### 5.6.1 什么是ANR

![什么是ANR](https://img-blog.csdnimg.cn/20191029203108127.png)

- `ANR` ，全称为 `Application Not Responding` 。
- 在 `Android` 中，如果你的应用程序有一段时间没有响应，系统会向用户显示一个对话框，这个对话框称作应用程序无响应对话框。

#### 5.6.2 用户行为

![用户行为](https://img-blog.csdnimg.cn/20191029203205701.png)

- 用户可以选择让程序继续运行，也可以让程序停止运行。
- 他们在使用你的应用程序时，并不希望每次都要处理这个对话框。
- 因此，在程序里对响应性能的设计很重要，这样，系统不会显示 `ANR` 给用户。

#### 5.6.3 Android不同组件ANR超时时间不同

![Android不同组件ANR超时时间不同](https://img-blog.csdnimg.cn/20191029203329185.png)

- 不同的组件发生 `ANR` 的时间不一样，主线程（ `Activity` 、`Service` ）是 `5` 秒，`BroadCastReceiver` 是 `10` 秒。

#### 5.6.4 解决方案

![解决方案](https://img-blog.csdnimg.cn/20191029203441463.png)

1. 将所有耗时操作，比如访问网络，`Socket` 通信，查询大量 `SQL` 语句，复杂逻辑计算等都放在子线程中去，然后通过 `handler.sendMessage` 、`runonUITread` 、`AsyncTask` 等方式更新 `UI` ，以确保用户界面操作的流畅度。
2. 如果耗时操作需要让用户等待，那么可以在界面上显示进度条。


## 5.7 android的任务栈 Task

![android的任务栈 Task](https://img-blog.csdnimg.cn/20191029203553524.png)

- 一个 `Task` 包含的就是 `activity` 集合，`android` 系统可以通过任务栈有序的管理 `activity` 
- 一个app当中可能不止一个任务栈，在某些情况下，一个 `activity` 也可以独享一个任务栈（ `singleInstance` 模式启动的 `activity` ）


![Android](https://img-blog.csdnimg.cn/20191029235448320.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzc3NzQ5,size_16,color_FFFFFF,t_70)
