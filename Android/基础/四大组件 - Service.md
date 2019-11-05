
# 前言

- 学 `Android` 有一段时间了，想必不少人也和我一样，平时经常东学西凑，感觉知识点有些凌乱难成体系。所以趁着这几天忙里偷闲，把学的东西归纳下，捋捋思路。

> 这篇文章主要针对 `Service` 相关的知识点，进行详细的梳理，祝大家食用愉快！

# 文章目录

![文章目录](https://user-gold-cdn.xitu.io/2019/11/3/16e2f2db1055287a?w=1640&h=1535&f=png&s=296260)

# 第一篇：Service 是什么
----

![Service 是什么](https://user-gold-cdn.xitu.io/2019/11/3/16e2f2db10737061?w=585&h=229&f=png&s=15104)

## 1.1 什么是 Service

![什么是 Service](https://user-gold-cdn.xitu.io/2019/11/3/16e2f2db14395a8a?w=1099&h=229&f=png&s=32153)

- `Service` (服务) 是一个一种可以在后台执行长时间运行操作而没有用户界面的应用组件。
- 服务可由其他应用组件启动（如 `Activity` ），服务一旦被启动将在后台一直运行，即使启动服务的组件（ `Activity` ）已销毁也不受影响。
- 此外，组件可以绑定到服务，以与之进行交互，甚至是执行进程间通信 ( `IPC` )。

## 1.2 Service 通常总是称之为 “后台服务”

![Service 通常总是称之为 “后台服务”](https://user-gold-cdn.xitu.io/2019/11/3/16e2f2db15e42e63?w=1574&h=205&f=png&s=42694)

- 其中 “后台” 一词是相对于前台而言的，具体是指：其本身的运行并不依赖于用户可视的 `UI` 界面
- 因此，从实际业务需求上来理解，`Service` 的适用场景应该具备以下条件：

1. 并不依赖于用户可视的 `UI` 界面（当然，这一条其实也不是绝对的，如前台 `Service` 就是与 `Notification` 界面结合使用的）

2. 具有较长时间的运行特性

3. 注意: 是运行在主线程当中的


## 1.3 服务进程

![服务进程](https://user-gold-cdn.xitu.io/2019/11/3/16e2f2db15f5377f?w=685&h=227&f=png&s=25144)

- 服务进程是通过 `startService()` 方法启动的进程，但不属于前台进程和可见进程。例如，在后台播放音乐或者在后台下载就是服务进程。

- 系统保持它们运行，除非没有足够内存来保证所有的前台进程和可视进程。



# 第二篇：生命周期
----

![生命周期](https://user-gold-cdn.xitu.io/2019/11/3/16e2f2db165413d9?w=1083&h=257&f=png&s=33531)

## 2.1 Service 的生命周期
- 我们先来看看 `Service` 的生命周期 的基本流程
- 一张闻名遐迩的图
![Service的生命周期](https://user-gold-cdn.xitu.io/2019/11/3/16e2f2dcb42f7251?w=432&h=521&f=png&s=82327)





## 2.2 开启 Service 的两种方式

![开启 Service 的两种方式](https://user-gold-cdn.xitu.io/2019/11/3/16e2f2dcb4682125?w=1248&h=318&f=png&s=75803)

#### 2.2.1 startService()

![startService()](https://user-gold-cdn.xitu.io/2019/11/3/16e2f2dcb4bf8fdb?w=764&h=241&f=png&s=30061)

1. 定义一个类继承 `Service`

2. 在 `Manifest.xml` 文件中配置该 `Service` 

3. 使用 `Context` 的 `startService(intent)` 方法开启服务。

4. 使用 `Context` 的 `stopService(intent)` 方法关闭服务。

5. 该启动方式，`app` 杀死、`Activity` 销毁没有任何影响，服务不会停止销毁。

#### 2.2.2 bindService()

![bindService()](https://user-gold-cdn.xitu.io/2019/11/3/16e2f2dcb4c7f792?w=995&h=298&f=png&s=56092)

1. 创建 `BindService` 服务端，继承 `Service` 并在类中，创建一个实现 `IBinder` 接口的实例对象，并提供公共方法给客户端（ `Activity` ）调用。

2. 从 `onBinder()` 回调方法返回该 `Binder` 实例。

3. 在客户端（ `Activity` ）中, 从 `onServiceConnection()` 回调方法参数中接收 `Binder` ，通过 `Binder` 对象即可访问 `Service` 内部的数据。

4. 在 `manifests` 中注册 `BindService` , 在客户端中调用  `bindService()` 方法开启绑定 `Service` , 调用 `unbindService()` 方法注销解绑 `Service` 。

5. 该启动方式依赖于客户端生命周期，当客户端 `Activity` 销毁时,  没有调用 `unbindService()` 方法 , `Service` 也会停止销毁。



## 2.3 Service 有哪些启动方法，有什么区别，怎样停用 Service

![Service 的启动与绑定](https://user-gold-cdn.xitu.io/2019/11/3/16e2f2dcb4dc73c3?w=520&h=227&f=png&s=17087)

- 在 `Service` 的生命周期中，被回调的方法比 `Activity` 少一些，只有 `onCreate` , `onStart` , `onDestroy` , `onBind` 和 `onUnbind` 。



- 通常有两种方式启动一个 `Service` , 他们对 `Service` 生命周期的影响是不一样的。



#### 2.3.1 通过 `startService`

![被启动的服务的生命周期](https://user-gold-cdn.xitu.io/2019/11/3/16e2f2dcb819e4d5?w=1419&h=229&f=png&s=50715)

- `Service` 会经历 `onCreate` 到 `onStart` ，然后处于运行状态，`stopService` 的时候调用 `onDestroy`
方法。

> 如果是调用者自己直接退出而没有调用 `stopService` 的话，`Service` 会一直在后台运行。

#### 2.3.2 通过 `bindService`

![被绑定的服务的生命周期](https://user-gold-cdn.xitu.io/2019/11/3/16e2f2dce02e174e?w=1181&h=216&f=png&s=40280)

`Service` 会运行 `onCreate` ，然后是调用 `onBind` ， 这个时候调用者和 `Service` 绑定在一起。调用者退出了，`Srevice` 就会调用 `onUnbind` -> `onDestroyed` 方法。

> 所谓绑定在一起就共存亡了。调用者也可以通过调用 `unbindService` 方法来停止服务，这时候 `Srevice` 就会调用 `onUnbind` -> `onDestroyed`  方法。

#### 2.3.3 需要注意的是如果这几个方法交织在一起的话，会出现什么情况呢？

![被启动又被绑定的服务的生命周期](https://user-gold-cdn.xitu.io/2019/11/3/16e2f2dce1301175?w=1177&h=229&f=png&s=34421)


1. 一个原则是 `Service` 的 `onCreate` 的方法只会被调用一次，就是你无论多少次的 `startService` 又 `bindService` ，`Service` 只被创建一次。

2. 如果先是 `bind` 了，那么 `start` 的时候就直接运行 `Service` 的 `onStart` 方法，如果先是 `start` ，那么 `bind` 的时候就直接运行 `onBind` 方法。

3. 如果 `service` 运行期间调用了 `bindService` ，这时候再调用 `stopService` 的话，`service` 是不会调用 `onDestroy` 方法的，`service` 就 `stop` 不掉了，只能调用 `UnbindService` , `service` 就会被销毁

4. 如果一个 `service` 通过 `startService` 被 `start` 之后，多次调用 `startService` 的话，`service` 会多次调
用 `onStart` 方法。多次调用 `stopService` 的话，`service` 只会调用一次 `onDestroyed` 方法。

5. 如果一个 `service` 通过 `bindService` 被 `start` 之后，多次调用 `bindService` 的话，`service` 只会调用一次 `onBind` 方法。多次调用 `unbindService` 的话会抛出异常。


# 第三篇：Service 与 Thread
----

![Service 与 Thread](https://user-gold-cdn.xitu.io/2019/11/3/16e2f2dce98d1364?w=1263&h=422&f=png&s=66535)

## 3.1 Service 和 Thread 的区别

![Service 和 Thread 的区别](https://user-gold-cdn.xitu.io/2019/11/3/16e2f2dcf8e6a041?w=2089&h=321&f=png&s=132016)

#### 3.1.1 首先第一点定义上

![定义上](https://user-gold-cdn.xitu.io/2019/11/3/16e2f2dcf922bdb1?w=1681&h=229&f=png&s=46952)

1. `thread` 是程序执行的最小单元，他是分配 `cpu` 的基本单位安卓系统中，我们常说的主线程，`UI` 线程，也是线程的一种。当然，线程里面还可以执行一些耗时的异步操作。
2. 而 `service` 大家记住，它是安卓中的一种特殊机制，`service` 是运行在主线程当中的，所以说它不能做耗时操作，它是由系统进程托管，其实 `service`  也是一种轻量级的 `IPC` 通信，因为 `activity` 可以和 `service` 绑定，可以和 `service` 进行数据通信。
3. 而且有一种情况，`activity` 和 `service` 是处于不同的进程当中，所以说它们之间的数据通信，要通过 `IPC` 进程间通信的机制来进行操作。

#### 3.1.2 第二点是在实际开发的过程当中

![第二点是在实际开发的过程当中](https://user-gold-cdn.xitu.io/2019/11/3/16e2f38aa2ba12de?w=1847&h=231&f=png&s=50262)

1. 在安卓系统当中，线程一般指的是工作线程，就是后台线程，做一些耗时操作的线程，而主线程是一种特殊的线程，它只是负责处理一些 `UI` 线程的绘制，`UI` 线程里面绝对不能做耗时操作，这里是最基本最重要的一点。（这是 `Thread` 在实际开发过程当中的应用）
2. 而 `service` 是安卓当中，四大组件之一，一般情况下也是运行在主线程当中，因此 `service` 也是不可以做耗时操作的，否则系统会报 ANR 异常（ `ANR` 全称：`Application Not Responding` ），就是程序无法做出响应。
3. 如果一定要在 `service` 里面进行耗时操作，一定要记得开启单独的线程去做。

#### 3.1.3 第三点是应用场景上

![应用场景上](https://user-gold-cdn.xitu.io/2019/11/3/16e2f2ddf4d1caad?w=1301&h=231&f=png&s=37547)

1. 当你需要执行耗时的网络，或者这种文件数据的查询，以及其它阻塞 `UI` 线程的时候，都应该使用工作线程，也就是开启一个子线程的方式。
2. 这样才能保证 `UI` 线程不被占用，而影响用户体验。
3. 而 `service` 来说，我们经常需要长时间在后台运行，而且不需要进行交互的情况下才会使用到服务，比如说，我们在后台播放音乐，开启天气预报的统计，还有一些数据的统计等等。


## 3.2 为什么要用 Service 而不是 Thread 


![为什么要用 Service 而不是 Thread ](https://user-gold-cdn.xitu.io/2019/11/3/16e2f2ddf4c5e5ff?w=1438&h=231&f=png&s=35955)

- `Thread` 的运行是独立于 `Activity` 的，也就是当一个 `Activity` 被 `finish` 之后，如果没有主动停止 `Thread` 或者 `Thread` 中的 `run` 没有执行完毕时那么这个线程会一直执行下去。
- 因此这里会出现一个问题：当 `Activity` 被 `finish` 之后，你不再持有该 `Thread` 的引用。
- 另一方面，你没有办法在不同的 `Activity` 中对同一 `Thread` 进行控制。



 

  
## 3.3 Service 里面是否能执行耗时的操作

![Service 里面是否能执行耗时的操作](https://user-gold-cdn.xitu.io/2019/11/3/16e2f2de416d50a3?w=1697&h=238&f=png&s=79662)

- service 里面不能执行耗时的操作(网络请求,拷贝数据库,大文件 )

- `Service` 不是独立的进程，也不是独立的线程，它是依赖于应用程序的主线程的，也就是说，在更多时候不建议在 `Service` 中编写耗时的逻辑和操作（比如:网络请求，拷贝数据库，大文件），否则会引起 `ANR` 。

- 如果想在服务中执行耗时的任务。有以下解决方案：

1. 在 `service` 中开启一个子线程

```java
new Thread(){}.start();
```

2. 可以使用 `IntentService` 异步管理服务（ 有关 `IntentService` 的内容在后文中给出 ）

## 3.4 Service 是否在 main thread 中执行

![Service 是否在 main thread 中执行](https://user-gold-cdn.xitu.io/2019/11/3/16e2f2de905972c7?w=1667&h=315&f=png&s=71266)

- 默认情况, 如果没有显示的指 `service` 所运行的进程, `Service` 和 `activity` 是运 行在当前 `app` 所在进程的 `main thread` ( `UI` 主线程)里面。
- `Service` 和 `Activity` 在同一个线程，对于同一 `app` 来说默认情况下是在同一个线程中的 `main Thread` ( `UI Thread` )
- 特殊情况 ,可以在清单文件配置 `service` 执行所在的进程 ,让 `service` 在另 外的进程中执行 `Service` 不死之身

> 注：如果是在 `application` 里创建的 `Thread` ：这个 `Thread` 的生命周期就跟 `app` 生命周期一致了，不同 `activity` 也可操作它，这时 `service` 和这个 `Thread` 就很相似了（前提是这个 `service` 只提供给自身 `app` 使用，第三方 `app` 不能访问）


#### 3.4.1 在 `onStartCommand` 方法中将 `flag` 设置为 `START_STICKY` ;

```xml
<service android:name="com.baidu.location.f" android:enabled="true" android:process=":remote" >
</service>
```


```java
return Service.START_STICKY;
```

#### 3.4.2 在 xml 中设置了 `android:priority`

```xml
<!--设置服务的优先级为MAX_VALUE-->
 <service android:name=".MyService"
          android:priority="2147483647"
          >
 </service>
```

#### 3.4.3 在 `onStartCommand` 方法中设置为前台进程

```java
@Override
public int onStartCommand(Intent intent, int flags, int startId) {
  Notification notification = new Notification(R.mipmap.ic_launcher, "服务正在运行",System.currentTimeMillis());
   Intent notificationIntent = new Intent(this, MainActivity.class);
    PendingIntent pendingIntent = PendingIntent.getActivity(this, 0,notificationIntent,0);
    RemoteViews remoteView = new RemoteViews(this.getPackageName(),R.layout.notification);
    remoteView.setImageViewResource(R.id.image, R.mipmap.ic_launcher);
    remoteView.setTextViewText(R.id.text , "Hello,this message is in a custom expanded view");
    notification.contentView = remoteView;
    notification.contentIntent = pendingIntent;
    startForeground(1, notification);
    return Service.START_STICKY;
}
```


#### 3.4.4 在 `onDestroy` 方法中重启 `service`

```java
@Override
public void onDestroy() {
    super.onDestroy();
    startService(new Intent(this, MyService.class));
}
```


#### 3.4.5 用 `AlarmManager.setRepeating(…)` 方法循环发送闹钟广播, 接收的时候调用 `service` 的 `onstart` 方法

```java
Intent intent = new Intent(MainActivity.this,MyAlarmReciver.class);
PendingIntent sender = PendingIntent.getBroadcast( MainActivity.this, 0, intent, 0);

// We want the alarm to go off 10 seconds from now.
Calendar calendar = Calendar.getInstance();
calendar.setTimeInMillis(System.currentTimeMillis());
calendar.add(Calendar.SECOND, 1);
AlarmManager am = (AlarmManager) getSystemService(ALARM_SERVICE);
//重复闹钟
/**
 *  @param type
 * @param triggerAtMillis t 闹钟的第一次执行时间，以毫秒为单位
 * go off, using the appropriate clock (depending on the alarm type).
 * @param intervalMillis 表示两次闹钟执行的间隔时间，也是以毫秒为单位
 * of the alarm.
 * @param operation 绑定了闹钟的执行动作，比如发送一个广播、给出提示等等
 */
am.setRepeating(AlarmManager.RTC_WAKEUP, calendar.getTimeInMillis(), 2 * 1000, sender);
```


#### 3.4.6 目前市场面的很多三方的消息推送 `SDK` 唤醒 `APP` , 例如 `Jpush`


> `PS`: 以上这些方法并不代表着你的 `Service` 就永生不死了，只能说是提高了进程的优先级。迄今为止我没有发现能够通过常规方法达到流氓需求 (通过长按 `home` 键清除都清除不掉) 的方法，目前所有方法都是指通过 `Android` 的内存回收机制和普通的第三方内存清除等手段后仍然保持运行的方法，有些手机厂商把这些知名的 `app` 放入了自己的白名单中，保证了进程不死来提高用户体验（如微信、`QQ` 、陌陌都在小米的白名单中）。如果从白名单中移除，他们终究还是和普通 `app` 一样躲避不了被杀的命运。







# 第四篇：IntentService
----

- 作为一个老司机，如果连 `Interservice` 都没听说过，那就有点那个啥了

![InterService](https://user-gold-cdn.xitu.io/2019/11/3/16e2f2de9095e245?w=1101&h=315&f=png&s=43201)


## 4.1 什么是 IntentService 

![什么是 IntentService ](https://user-gold-cdn.xitu.io/2019/11/3/16e2f2de9b547c8e?w=1075&h=195&f=png&s=31011)

- `IntentService` 是 `Service` 的子类，比普通的 `Service` 增加了额外的功能。

- 我们常用的 `Service` 存在两个问题：

1. `Service` 不会专门启动一条单独的进程，`Service` 与它所在应用位于同一个进程中

2. `Service` 也不是专门一条新线程，因此不应该在 `Service` 中直接处理耗时的任务


## 4.2 IntentService 的特征

![IntentService 的特征](https://user-gold-cdn.xitu.io/2019/11/3/16e2f2dea0c31bef?w=948&h=249&f=png&s=41825)

- 会创建独立的 `worker` 线程来处理所有的 `Intent` 请求

- 会创建独立的 `worker` 线程来处理 `onHandleIntent()` 方法实现的代码，无需处理多线程问题

- 所有请求处理完成后，`IntentService` 会自动停止，无需调用 `stopSelf()` 方法停止 `Service`

- 为 `Service` 的 `onBind()` 提供默认实现，返回 `null`

- 为 `Service` 的 `onStartCommand` 提供默认实现，将请求 `Intent` 添加到队列中



## 4.3 Service 和 IntentService 区别

![Service 和 IntentService 区别](https://user-gold-cdn.xitu.io/2019/11/3/16e2f2def2c9aa9d?w=1721&h=298&f=png&s=67309)

#### 4.3.1 `Service` 是用于后台服务的

![Service 是用于后台服务的](https://user-gold-cdn.xitu.io/2019/11/3/16e2f2df0f4ee34c?w=1552&h=229&f=png&s=69954)

1. 当应用程序被挂到后台的时候，为了保证应用某些组件仍然可以工作而引入了 `Service` 这个概念
2. 那么这里面要强调的是：`Service` 不是独立的进程，也不是独立的线程，它是依赖于应用程序的主线程的，也就是说，在更多时候不建议在 `Service` 中编写耗时的逻辑和操作，否则会引起 `ANR` 。

> 也就是，service 里面不可以进行耗时的操作。虽然在后台服务。但是也是在主线程里面。

#### 4.3.2 当我们编写的耗时逻辑，不得不被 `service` 来管理的时候，就需要引入 `IntentService` 。

![耗时逻辑](https://user-gold-cdn.xitu.io/2019/11/3/16e2f2df0f7cdac3?w=958&h=229&f=png&s=31611)

1. `IntentService` 是继承 `Service` 的，那么它包含了 `Service` 的全部特性，当然也包含 `service` 的生命周期。
2. 那么与 `service` 不同的是，`IntentService` 在执行 `onCreate` 操作的时候，内部开了一个线程，去你执行你的耗时操作。

#### 4.3.3 使用：

![使用](https://user-gold-cdn.xitu.io/2019/11/3/16e2f2df12174670?w=585&h=146&f=png&s=7241)

1. 重写 `protected abstract void onHandleIntent(Intent intent)`

#### 4.3.4 `IntentService` 是一个通过 `Context.startService(Intent)` 启动可以处理异步请求的 `Service` 

![通过 Context.startService(Intent) 启动](https://user-gold-cdn.xitu.io/2019/11/3/16e2f2df12afab88?w=1459&h=234&f=png&s=40550)

1. 使用时你只需要继承 `IntentService` 和重写其中的 `onHandleIntent(Intent)` 方法接收一个 `Intent` 对象 , 在适当的时候会停止自己 ( 一般在工作完成的时候 ) 。
2. 所有的请求的处理都在一个工作线程中完成 , 它们会交替执行 ( 但不会阻塞主线程的执行 ) ，一次只能执行一个请求。


#### 4.3.5 是一个基于消息的服务

![是一个基于消息的服务](https://user-gold-cdn.xitu.io/2019/11/3/16e2f2df1c4bb0fc?w=1271&h=229&f=png&s=43142)

1. 每次启动该服务并不是马上处理你的工作，而是首先会创建对应的 `Looper` ，`Handler` 并且在 `MessageQueue` 中添加的附带客户 `Intent` 的 `Message` 对象。
2. 当 `Looper` 发现有 `Message` 的时候接着得到 `Intent` 对象通过在 `onHandleIntent((Intent)msg.obj)` 中调用你的处理程序，处理完后即会停止自己的服务。
3. 意思是 `Intent` 的生命周期跟你的处理的任务是一致的，所以这个类用下载任务中非常好，下载任务结束后服务自身就会结束退出。


#### 4.3.6 总结 `IntentService` 的特征有：

![总结 IntentService 的特征](https://user-gold-cdn.xitu.io/2019/11/3/16e2f2df1ccc7b0e?w=961&h=229&f=png&s=30941)

1. 会创建独立的 `worker` 线程来处理所有的 `Intent` 请求；

2. 会创建独立的 `worker` 线程来处理 `onHandleIntent()` 方法实现的代码，无需处理多线程问题；

3. 所有请求处理完成后，`IntentService`会自动停止，无需调用 `stopSelf()` 方法停止 `Service` ；




# 第五篇：Service 与 Activity
----

![Service 与 Activity](https://user-gold-cdn.xitu.io/2019/11/3/16e2f38ab4b3603f?w=751&h=419&f=png&s=41510)

## 5.1 Activity 怎么和 Service 绑定，怎么在 Activity 中启动对应的 Service

![Service 与 Activity](https://user-gold-cdn.xitu.io/2019/11/3/16e2f38ab686c376?w=1659&h=205&f=png&s=52892)

- `Activity` 通过 `bindService(Intent service, ServiceConnection conn, int flags)` 跟 `Service` 进行绑定，当绑定成功的时候 `Service` 会将代理对象通过回调的形式传给 `conn` ，这样我们就拿到了 `Service` 提供的服务代理对象。

- 在 `Activity` 中可以通过 `startService` 和 `bindService` 方法启动 `Service`。一般情况下如果想获取 `Service` 的服务对象那么肯定需要通过  `bindService()` 方法，比如音乐播放器，第三方支付等。

- 如果仅仅只是为了开启一个后台任务那么可以使用 `startService()` 方法。



## 5.2 说说 Activity 、Intent 、Service 是什么关系

![Activity 、Intent 、Service 是什么关系](https://user-gold-cdn.xitu.io/2019/11/3/16e2f38ab6b3cd9d?w=1529&h=229&f=png&s=40487)

- 他们都是 `Android` 开发中使用频率最高的类。其中 `Activity` 和 `Service` 都属于 `Android` 的四大组件。他俩都是 `Context` 类的子类 `ContextWrapper` 的子类，因此他俩可以算是兄弟关系吧。

- 不过他们各有各自的本领，`Activity` 负责用户界面的显示和交互，`Service` 负责后台任务的处理。

- `Activity` 和 `Service` 之间可以通过 `Intent` 传递数据，因此可以把 `Intent` 看作是通信使者。



## 5.3 Service 和 Activity 在同一个线程吗

![Service 和 Activity 在同一个线程吗](https://user-gold-cdn.xitu.io/2019/11/3/16e2f38ab9df04f1?w=975&h=204&f=png&s=15712)

对于同一 `app` 来说默认情况下是在同一个线程中的，`main Thread` （ `UI Thread` ）。




## 5.4 Service 里面可以弹吐司么

![Service 里面可以弹吐司么](https://user-gold-cdn.xitu.io/2019/11/3/16e2f38abb5d73e4?w=1015&h=227&f=png&s=25846)

- 可以
- 弹吐司有个条件是：得有一个 `Context` 上下文，而 `Service` 本身就是 `Context` 的子类
- 因此在 `Service` 里面弹吐司是完全可以的。比如我们在 `Service` 中完成下载任务后可以弹一个吐司通知给用户。





## 5.5 与 Service 交互方式

![与 Service 交互方式](https://user-gold-cdn.xitu.io/2019/11/3/16e2f38bb35e297c?w=635&h=227&f=png&s=17920)

#### 5.5.1 广播交互

![广播交互](https://user-gold-cdn.xitu.io/2019/11/3/16e2f38bb387c1dd?w=805&h=229&f=png&s=29278)

1. `Server` 端将目前的下载进度，通过广播的方式发送出来，`Client` 端注册此广播的监听器，当获取到该广播后，将广播中当前的下载进度解析出来并更新到界面上。
2. 定义自己的广播，这样在不同的 `Activity` 、`Service` 以及应用程序之间，就可以通过广播来实现交互。

#### 5.5.2 共享文件交互

![共享文件交互](https://user-gold-cdn.xitu.io/2019/11/3/16e2f38c3162d4a4?w=924&h=229&f=png&s=31658)

1. 我们使用 `SharedPreferences` 来实现共享，当然也可以使用其它 `IO` 方法实现，通过这种方式实现交互时需要注意，对于文件的读写的时候，同一时间只能一方读一方写，不能两方同时写。
2. `Server` 端将当前下载进度写入共享文件中，`Client` 端通过读取共享文件中的下载进度，并更新到主界面上。

#### 5.5.3  `Messenger` 交互 ( 信使交互 ) 

![Messenger 交互 ( 信使交互 ) ](https://user-gold-cdn.xitu.io/2019/11/3/16e2f38c31915405?w=1365&h=229&f=png&s=43049)

1. `Messenger` 翻译过来指的是信使，它引用了一个 `Handler` 对象，别人能够向它发送消息 ( 使用 `mMessenger.send ( Message msg )` 方法)。
2. 该类允许跨进程间基于 `Message` 通信，在服务端使用 `Handler` 创建一个 `Messenger` ，客户端只要获得这个服务端的 `Messenger` 对象就可以与服务端通信了
3. 在 `Server` 端与 Client 端之间通过一个 `Messenger` 对象来传递消息，该对象类似于信息中转站，所有信息通过该对象携带

#### 5.5.4 自定义接口交互

![自定义接口交互](https://user-gold-cdn.xitu.io/2019/11/3/16e2f38c3182ffa2?w=1475&h=229&f=png&s=43333)

1. 其实就是我们自己通过接口的实现来达到 `Activity` 与 `Service` 交互的目的，我们通过在 `Activity` 和 `Service` 之间架设一座桥樑，从而达到数据交互的目的，而这种实现方式和 `AIDL` 非常类似
2. 自定义一个接口，该接口中有一个获取当前下载进度的空方法。`Server` 端用一个类继承自 `Binder` 并实现该接口，覆写了其中获取当前下载进度的方法。`Client` 端通过 `ServiceConnection` 获取到该类的对象，从而能够使用该获取当前下载进度的方法，最终实现实时交互。

#### 5.5.5 `AIDL` 交互

![AIDL交互](https://user-gold-cdn.xitu.io/2019/11/3/16e2f38c31bcf3aa?w=771&h=185&f=png&s=20018)

1. 远程服务一般通过 `AIDL` 来实现，可以进行进程间通信，这种服务也就是远程服务。
2. `AIDL` 属于 `Android` 的 `IPC` 机制，常用于跨进程通信，主要实现原理基于底层 `Binder` 机制。

- [Android 面试，与Service交互方式](https://www.cnblogs.com/yydcdut/p/3961545.html)



# 第六篇：使用
----

![使用](https://user-gold-cdn.xitu.io/2019/11/3/16e2f38c466c75d4?w=892&h=413&f=png&s=49838)

## 6.1 什么情况下会使用 Service

![什么情况下会使用 Service](https://user-gold-cdn.xitu.io/2019/11/3/16e2f38cccd6bd23?w=1952&h=216&f=png&s=66080)

#### 6.1.1 经验总结：

![经验总结](https://user-gold-cdn.xitu.io/2019/11/3/16e2f38ccd4ad625?w=882&h=185&f=png&s=20411)

1. `Service` 其实就是背地搞事情，又不想让别人知道
2. 举一个生活当中的例子，你想知道一件事情不需要直接去问，你可以通过侧面了解。这就是 `Service` 设计的初衷

#### 6.1.2 `Service` 为什么被设计出来

![Service 为什么被设计出来](https://user-gold-cdn.xitu.io/2019/11/3/16e2f38ccd5d42bb?w=1731&h=229&f=png&s=47014)

3. 根据 `Service` 的定义，我们可以知道需要长期在后台进行的工作我们需要将其放在 `Service` 中去做。
4. 得再通熟易懂一点，就是不能放在 `Activity` 中来执行的工作就必须得放到 `Service` 中去做。
5. 如：音乐播放、下载、上传大文件、定时关闭应用等功能。这些功能如果放到 `Activity` 中做的话，那么 `Activity` 退出被销毁了的话，那这些功能也就停止了，这显然是不符合我们的设计要求的，所以要将他们放在 `Service` 中去执行。



## 6.2 onStartCommand() 返回值 int 值的区别

- 有四种返回值,不同值代表的意思如下:

![onStartCommand() 返回值 int 值的区别](https://user-gold-cdn.xitu.io/2019/11/3/16e2f38cd0775a31?w=1407&h=371&f=png&s=80681)

#### 6.2.1 `START_STICKY` : 
1. 如果 `service` 进程被 kill 掉,保留 `service` 的状态为开始状态,但不保留递送的 `intent` 对象。
2. 随后系统会尝试重新创建 `service`, 由于服务状态为开始状态,所以创建服务后一定会调用 `onStartCommand ( Intent, int, int )` 方法。
3. 如果在此期间没有任何启动命令被传递到 `service` , 那么参数  `Intent` 将为 `null` 。

#### 6.2.2 `START_NOT_STICKY` :
1. “非粘性的”。
2. 使用这个返回值时 , 如果在执行完 `onStartCommand` 后 , 服务被异常 `kill` 掉 ，系统不会自动重启该服务。

#### 6.2.3 `START_REDELIVER_INTENT`:
1. 重传 `Intent` 。
2. 使用这个返回值时,如果在执行完 `onStartCommand` 后,服务被异常 kill 掉 
3. 系统会自动重启该服务 , 并将 Intent 的值传入。

#### 6.2.4 `START_STICKY_COMPATIBILITY`: 
1. `START_STICKY` 的兼容版本 , 但不保证服务被 `kill` 后一定能重启。




## 6.3 在 service 的生命周期方法 onstartConmand() 可不可以执行网络操作？如何在 service 中执行网络操作？

![onstartConmand() 可不可以执行网络操作？如何在 service 中执行网络操作？](https://user-gold-cdn.xitu.io/2019/11/3/16e2f38cd4748128?w=762&h=185&f=png&s=20865)

- 可以直接在 `Service` 中执行网络操作 
- 在 `onStartCommand()` 方法中可以执行网络操作




## 6.4 提高 service 的优先级

![提高 service 的优先级](https://user-gold-cdn.xitu.io/2019/11/3/16e2f38d5ae243ea?w=2502&h=249&f=png&s=100226)

- 在 `AndroidManifest.xml` 文件中对于 `intent-filter` 可以通过 `android:priority = “1000”` 这个属性设置最高优先级，`1000` 是最高值，如果数字越小则优先级越低，同时实用于广播。

- 在 `onStartCommand` 里面调用 `startForeground()` 方法把 `Service` 提升为前台进程级别，然后再 `onDestroy` 里面要记得调用 `stopForeground ()` 方法。

- `onStartCommand` 方法，手动返回 `START_STICKY` 。

![广播](https://user-gold-cdn.xitu.io/2019/11/3/16e2f38d66be892d?w=1524&h=204&f=png&s=47525)

- 在 `onDestroy` 方法里发广播重启 `service` 。
1. `service` + `broadcast` 方式，就是当 `service` 走 `ondestory` 的时候，发送一个自定义的广播
2. 当收到广播的时候，重新启动 `service` 。（ 第三方应用或是在 `setting` 里-应用强制停止时，`APP` 进程就直接被干掉了，`onDestroy` 方法都进不来，所以无法保证会执行 ）

- 监听系统广播判断 `Service` 状态。
1. 通过系统的一些广播
2. 比如：手机重启、界面唤醒、应用状态改变等等监听并捕获到，然后判断我们的 `Service` 是否还存活。

- `Application` 加上 `Persistent` 属性。



## 6.5 Service 的 onRebind ( Intent ) 方法在什么情况下会执行

![onRebind ( Intent ) 方法在什么情况下会执行](https://user-gold-cdn.xitu.io/2019/11/3/16e2f38d693da52a?w=773&h=184&f=png&s=15328)

- 如果在 `onUnbind()` 方法返回 `true` 的情况下会执行 , 否则不执行。



# 总结
----

1. 本文基本涵盖了 `Android Service` 相关的知识点。由于篇幅原因，诸如 InterService 具体使用方法等，没办法详细的介绍，大家很容易就能在网上找到资料进行学习。
2. 开始前还以为总结不难，实际写文章的过程中，才知道什么是艰辛。也不知道自己能不能咬牙坚持下去，希望大家给我鼓励，就算只是一个赞，也是我坚持下去的理由！

# 码字不易，你的 `star` 是我总结的最大动力！
