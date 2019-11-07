# 二、ContentProvider
- `ContentProvider` 应用程序间非常通用的共享数据的一种方式，也是 `Android` 官方推荐的方式。
- `Android` 中许多系统应用都使用该方式实现数据共享，比如通讯录、短信等。


![ContentProvider](https://img-blog.csdnimg.cn/2019110712210021.png)

## 2.1 Android 为什么要设计 ContentProvider 这个组件？

![为什么要设计 ContentProvider](https://img-blog.csdnimg.cn/20191107111728578.png)

- 很多做 `Android` 开发的人都不怎么使用它，觉得直接读取数据库会更简单方便。
- 那么 `Android` 搞一个内容提供者在数据和应用之间，只是为了装高大上，故弄玄虚？我认为其设计用意在于：

1. 封装。对数据进行封装，提供统一的接口，使用者完全不必关心这些数据是在 `DB` ，`XML` 、`Preferences` 或者网络请求来的。当项目需求要改变数据来源时，使用我们的地方完全不需要修改。
2. 提供一种跨进程数据共享的方式。
3. 应用程序间的数据共享还有另外的一个重要话题，就是数据更新通知机制了。因为数据是在多个应用程序中共享的，当其中一个应用程序改变了这些共享数据的时候，它有责任通知其它应用程序，让它们知道共享数据被修改了，这样它们就可以作相应的处理。


## 2.2 如何访问自定义 ContentProvider

![如何访问自定义 ContentProvider](https://img-blog.csdnimg.cn/20191107112119671.png)

- `ContentResolver` 接口的 `notifyChange` 函数来通知那些注册了监控特定 URI的ContentObserver 对象，使得它们可以相应地执行一些处理。
- ContentObserver 可以通过 registerContentObserver 进行注册。
- 通过 `ContentProvider` 的 `Uri` 访问开放的数据。

1. `ContenResolver` 对象通过 `Context` 提供的方法 `getContenResolver()` 来获得。
2. `ContenResolver` 提供了以下方法来操作：`insert` `delete` `update` `query` 这些方法分别会调用 `ContenProvider` 中与之对应的方法并得到返回的结果。

## 2.3 通过 ContentResolver 获取 ContentProvider 内容的基本步骤

![ContentResolver 获取 ContentProvider 内容的基本步骤](https://img-blog.csdnimg.cn/20191107112310552.png)

1. 得到 `ContentResolver` 类对象：`ContentResolver cr = getContentResolver ( )`。
2. 定义要查询的字段 `String` 数组。
3. 使用 `cr.query()` ; 返回一个 `Cursor` 对象。
4. 使用 `while` 循环得到 `Cursor` 里面的内容。


## 2.4 ContentProvider 是如何实现数据共享的：

![ContentProvider 是如何实现数据共享的](https://img-blog.csdnimg.cn/20191107112741556.png)

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

![为什么要用 ContentProvider ?它和 sql 的实现上有什么差别](https://img-blog.csdnimg.cn/20191107112907157.png)

- `ContentProvider` 屏蔽了数据存储的细节 , 内部实现对用户完全透明 , 用户只需要关心操作数据的 `uri` 就可以了, `ContentProvider` 可以实现不同 `app`之间 共享。
- `Sql` 也有增删改查的方法, 但是 `sql` 只能查询本应用下的数据库。
- 而 `ContentProvider` 还可以去增删改查本地文件. `xml` 文件的读取等。

## 2.6 Uri 介绍

![Uri 介绍](https://img-blog.csdnimg.cn/20191107113345318.png)


- 为系统的每一个资源给其一个名字，比方说通话记录。
1. 每一个 `ContentProvider` 都拥有一个公共的 `URI` ，这个 `URI` 用于表示这个 `ContentProvider` 所提供的数据。 
2. `Android` 所提供的 `ContentProvider` 都存放在 `android.provider` 包中。 

- 将其分为 `A，B，C，D` 4个部分：
- `A`：标准前缀，用来说明一个 `Content Provider` 控制这些数据，无法改变的；`"content://"`；
- `B`：`URI` 的标识，用于唯一标识这个 `ContentProvider` ，外部调用者可以根据这个标识来找到它。它定义了是哪个 `ContentProvider` 提供这些数据。对于第三方应用程序，为了保证 `URI` 标识的唯一性，它必须是一个完整的、小写的类名。这个标识在元素的 `authorities` 属性中说明：一般是定义该 `ContentProvider` 的包类的名称；
- `C`：路径（ `path` ），通俗的讲就是你要操作的数据库中表的名字，或者你也可以自己定义，记得在使用的时候保持一致就可以了；`"content://com.bing.provider.myprovider/tablename"`。
- `D`：如果URI中包含表示需要获取的记录的 `ID`；则就返回该id对应的数据，如果没有 `ID`，就表示返回全部； `"content://com.bing.provider.myprovider/tablename/#"` `#` 表示数据 `id` 。


## 2.7 如何访问 asserts 资源目录下的数据库?

![访问 asserts 资源目录下的数据库](https://img-blog.csdnimg.cn/20191107113504103.png)

- 把数据库 `db` 复制到 `/data/data/packagename/databases/` 目录下, 然后直接就能访问了。
 

## 2.8 多个进程同时调用一个 ContentProvider 的 query 获取数据，ContentPrvoider 是如何反应的呢？

![调用一个 ContentProvider 的 query 获取数据，ContentPrvoider 是如何反应的](https://img-blog.csdnimg.cn/20191107114742923.png)

- 一个 `ContentProvider` 可以接受来自另外一个进程的数据请求。
- 尽管 `ContentResolver` 与 `ContentProvider` 类隐藏了实现细节，但是 `ContentProvider` 所提供的 `query()`，`insert()`，`delete()`，`update()` 都是在 `ContentProvider` 进程的线程池中被调用执行的，而不是进程的主线程中。
- 这个线程池是有 `Binder` 创建和维护的，其实使用的就是每个应用进程中的 `Binder` 线程池。

## 2.9 Android 设计 ContentProvider 的目的是什么呢？

![设计 ContentProvider 的目的](https://img-blog.csdnimg.cn/20191107114949656.png)

- 隐藏数据的实现方式，对外提供统一的数据访问接口；
- 更好的数据访问权限管理。`ContentProvider` 可以对开发的数据进行权限设置，不同的 `URI` 可以对应不同的权限，只有符合权限要求的组件才能访问到 `ContentProvider` 的具体操作。
- `ContentProvider` 封装了跨进程共享的逻辑，我们只需要 `Uri` 即可访问数据。由系统来管理 `ContentProvider` 的创建、生命周期及访问的线程分配，简化我们在应用间共享数据（ 进程间通信 ）的方式。我们只管通过 `ContentResolver` 访问 `ContentProvider` 所提示的数据接口，而不需要担心它所在进程是启动还是未启动。

## 2.10 运行在主线程的 ContentProvider 为什么不会影响主线程的UI操作?

![ContentProvider 为什么不会影响主线程的UI操作](https://img-blog.csdnimg.cn/2019110711540975.png)

- `ContentProvider` 的 `onCreate()` 是运行在 `UI` 线程的，而 `query()` ，`insert()` ，`delete()` ，`update()` 是运行在线程池中的工作线程的
- 所以调用这向个方法并不会阻塞 `ContentProvider` 所在进程的主线程，但可能会阻塞调用者所在的进程的 `UI` 线程！
- 所以，调用 `ContentProvider` 的操作仍然要放在子线程中去做。
- 虽然直接的 `CRUD` 的操作是在工作线程的，但系统会让你的调用线程等待这个异步的操作完成，你才可以继续线程之前的工作。


## 2.11 外提供数据共享，那么如何限制对方的使用呢？

![如何限制对方的使用](https://img-blog.csdnimg.cn/20191107115200504.png)

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

![如果我们只需要开放部份的  URI 给其他的应用访问呢](https://img-blog.csdnimg.cn/20191107115607274.png)

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

![ContentProvider 接口方法运行在哪个线程中](https://img-blog.csdnimg.cn/20191107115802830.png)

- `ContentProvider` 可以在 `AndroidManifest.xml` 中配置一个叫做 `android:multiprocess` 的属性，默认值是 false ，表示 ContentProvider 是单例的
- 无论哪个客户端应用的访问都将是同一个 `ContentProvider` 对象，如果设为 `true` ，系统会为每一个访问该 `ContentProvider` 的进程创建一个实例。

#### 2.12.1 这点还是比较好理解的，那如果我要问每个 ContentProvider 的操作是在哪个线程中运行的呢?（ 其实我们关心的是 UI 线程和工作线程 ）

![每个 ContentProvider 的操作是在哪个线程中运行的](https://img-blog.csdnimg.cn/20191107120017748.png)

- 比如我们在UI线程调用getContentResolver().query查询数据，而当数据量很大时（或者需要进行较长时间的计算）会不会阻塞UI线程呢？

- 要分两种情况回答这个问题：

1. `ContentProvider` 和调用者在同一个进程，`ContentProvider` 的方法（ `query/insert/update/delete` 等 ）和调用者在同一线程中；
2. `ContentProvider` 和调用者在不同的进程，`ContentProvider` 的方法会运行在它自身所在进程的一个 Binder 线程中。
但是，注意这两种方式在 `ContentProvider` 的方法没有执行完成前都会 `blocked` 调用者。所以你应该知道这个上面这个问题的答案了吧。
3. 也可以看看 `CursorLoader` 这个类的源码，看 `Google` 自己是怎么使用 `getContentResolver().query` 的。

## 2.13 ContentProvider 是如何在不同应用程序之间传输数据的？

![ContentProvider 是如何在不同应用程序之间传输数据](https://img-blog.csdnimg.cn/20191107120231873.png)

- 一个应用进程有 `16` 个 `Binder` 线程去和远程服务进行交互，而每个线程可占用的缓存空间是 `128KB` 这样，超过会报异常。
- `ContentResolver` 虽然是通过 `Binder` 进程间通信机制打通了应用程序之间共享数据的通道，但 `ContentProvider` 组件在不同应用程序之间传输数据是基于匿名共享内存机制来实现的。
- 有兴趣的可以查看一下老罗的文章[Android系统匿名共享内存Ashmem（Anonymous Shared Memory）简要介绍和学习计划](https://blog.csdn.net/luoshengyang/article/details/6651971)。

[](https://upload-images.jianshu.io/upload_images/1685558-982648eb6de31f91.png?imageMogr2/auto-orient/strip|imageView2/2/w/600/format/webp)

![ContentProvider](https://img-blog.csdnimg.cn/20191105184103232.png)
