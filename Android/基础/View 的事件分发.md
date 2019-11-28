
# 前言

- Android View 的 事件处理在我们的编程中，可谓是无处不在了。但对于大多数人而言，一直都是简单的使用，对其原理缺乏深入地认识。
- 学 Android  有一段时间了，最近发现，很多基础知识开始有些遗忘了，所以从新复习了 View 的事件分发。特地整理成了这篇文章分享给大家。
- 本文不难，可以作为大家茶余饭后的休闲。

> 祝大家阅读愉快！

![Android View 的事件处理](https://img-blog.csdnimg.cn/20191125142724573.png)

# 方便大家学习，我在 GitHub 上建立个 仓库

----

- 仓库内容与博客同步更新。由于我在 `稀土掘金` `简书` `CSDN` `博客园` 等站点，都有新内容发布。所以大家可以直接关注该仓库，即使获得精彩内容

- 仓库地址：
[超级干货！精心归纳 `Android` 、`JVM` 、算法等，各位帅气的老铁支持一下！给个 Star ！](https://github.com/FishInWater-1999/android_interviews)

# 一、View 的事件回调

- 我们结合源码看看 `View` 的事件分发是个怎样的过程，首先我们建立一个类 `MyButton` 类继承 `AppCompatButton` 用于测试：

```java
public class MyButton extends AppCompatButton {

    private final String TAG = "DeBugMyButton";
        public MyButton(Context context) {
        super(context);
    }

    public MyButton(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public MyButton(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

}
```

## 1.1 事件分发流程
- 我们都知道有一个方法叫做 `public boolean dispatchTouchEvent(MotionEvent event)`  。首先我们要知道，对于我们这个自定义控件，他的触摸事件都是从我们 `dispatchTouchEvent` 这个方法开始往下去分发的。所以可以说：这个方法是一个入口方法。

#### 1.1.1 onTouchEvent 作用

- 现在我们重写该方法和另一个方法：`onTouchEvent` ，并且打印一行日志：

```java
@Override
public boolean dispatchTouchEvent(MotionEvent event) {
    Log.d(TAG, "----on dispatch Touch Event----");
    return super.dispatchTouchEvent(event);
}

@Override
public boolean onTouchEvent(MotionEvent event) {
    switch (event.getAction()) {
        case MotionEvent.ACTION_DOWN:
            Log.d(TAG, "----on touch event----");
    }
    return super.onTouchEvent(event);
}
```

- 然后我们在 `MainActivity` 中，设置一个实例化一个 `MyButton` 控件对象用于测试，并且给他添加一个 `onClickListenter` 和 `setOnTouchListener`

```java
public class MainActivity extends AppCompatActivity {

    private final String TAG = "DeBugMainActivity";

    /**
     * 自定义控件 MyButton
     */
    private MyButton mMyButton;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        iniView();
    }

    /**
     * 实例化控件
     */
    private void iniView() {
        mMyButton = findViewById(R.id.my_button);

	mMyButton.setOnTouchListener(new View.OnTouchListener() {
	    @Override
	    public boolean onTouch(View v, MotionEvent event) {
	        switch (event.getAction()) {
	            case MotionEvent.ACTION_DOWN:
	                Log.d(TAG, "----on touch----");
	                break;
	            default:
	                break;
	        }
	        return false;
	    }
	});
	
	mMyButton.setOnClickListener(new View.OnClickListener() {
	    @Override
	    public void onClick(View v) {
	        Log.d(TAG, "----on click----");
	    }
	});
    }
}
```

- 然后我们运行这个 `Demo` ，点击 `MyButton` 按钮，会的到如下日志：

![](https://img-blog.csdnimg.cn/20191117143321969.png)

- 我们可以看到首先回调了这个 `dispatchTouchEvent` ，然后是它的监听器 `OnTouch` ，接着是它的 `onTouchEvent`，最后又执行了 `dispatchTouchEvent` ，那么这是为什么呢？

- 这是因为我们这儿只监听了 `ACTION_DOWN` 而当手指抬起时它同样还回去回调 `dispatchTouchEvent` ，最后我们打印 `OnClick` 的回调。

- 总结一下就是：
`dispatchTouchEvent` -> `setOnTouchListener` -> `onTouchEvent` -> `setOnClickListener` 

- 说明我们 `setOnClickListener`  是通过 `onTouchEvent`  处理，产生了 `OnClick` 。一会我们再来看看其中的原理。

- 既然说 `dispatchTouchEvent` 像一个入口，就先让我们来看下它是怎么处理和操作的： 首先，既然我们调用了 `super.dispatchTouchEvent(event)` ，那么我们就来看看它父类中是怎么实现该方法的。不信的是，它的父类 `AppCompatButton` 也没有实现该方法 ，最后经过层层搜寻，我们发现这个方法是属于 `View` 的方法。

#### 1.1.2 dispatchTouchEvent 的实现

- 那么现在我们来看看 `View` 的 `dispatchTouchEvent` 怎么实现的：


```java
public boolean dispatchTouchEvent(MotionEvent event) {
    ......
        //noinspection SimplifiableIfStatement
        ListenerInfo li = mListenerInfo;
        if (li != null && li.mOnTouchListener != null
                && (mViewFlags & ENABLED_MASK) == ENABLED
                && li.mOnTouchListener.onTouch(this, event)) {
            result = true;
        }

        if (!result && onTouchEvent(event)) {
            result = true;
        }
    }

    if (!result && mInputEventConsistencyVerifier != null) {
        mInputEventConsistencyVerifier.onUnhandledEvent(event, 0);
    }

    // Clean up after nested scrolls if this is the end of a gesture;
    // also cancel it if we tried an ACTION_DOWN but we didn't want the rest
    // of the gesture.
    if (actionMasked == MotionEvent.ACTION_UP ||
            actionMasked == MotionEvent.ACTION_CANCEL ||
            (actionMasked == MotionEvent.ACTION_DOWN && !result)) {
        stopNestedScroll();
    }

    return result;
}
```

- 在 `dispatchTouchEvent` 中，我们可以发现下面这样一个代码块

```java
if (li != null && li.mOnTouchListener != null
        && (mViewFlags & ENABLED_MASK) == ENABLED
        && li.mOnTouchListener.onTouch(this, event)) {
    result = true;
}
```

- 不难看出：如果执行了这个代码段，那么后面的方法就不会执行了，并且 `dispatchTouchEvent`  会返回 `true` 。我们再仔细观察下其中的条件：在 `if` 条件中我们发现：只有当其满足 `li.mOnTouchListener != null && (mViewFlags & ENABLED_MASK) == ENABLED && li.mOnTouchListener.onTouch(this, event))` 时才会执行 `if` 内的操作

- 经过上面分析，我们可以知道： `onTouch` 事件必须返回 `true` 时，才会执行该方法块。那么我们就回到 `MainActivity` 中。我们发现 `setOnTouchListener` 的 `onTouch` 默认返回值是 `false`（ 不满足返回值为 `true` ),  这就表明他会继续去执行下一个代码块：

```java
if (!result && onTouchEvent(event)) {
    result = true;
}
```

- 执行这个 `if` 语句的过程中。首先调用了 `onTouchEvent` 方法。这就解释了，为什么它先执行了 `mOnTouchListener` ，然后再执行 `onTouchEvent` 。

- 现在我们就可以总结一下：首先我们回调了 `dispatchTouchEvent`  ，然后回调 `OnTouchListener` 。这个时候，如果 `TouchListener` 没有 `return true` ，那么就会接着去运行 `onTouchEvent` （ 当然，如果 `return true` 后面的层级就不会执行了 。一句话说就是：到那个层级 `return true` 那么哪个层级就消费掉了这个事件 ）。

#### 1.1.3 onTouchEvent 的处理

- 同时我们还有一个结果：我们 `onClick` （ 包括我们的 `onLongClick` ） 是来自于我们 `onTouchEvent` 这个方法的处理。那么下面我们就来看看 `View` 中是怎么处理 `onTouchEvent` 的：

```java
public boolean onTouchEvent(MotionEvent event) {
    。。。

    if (clickable || (viewFlags & TOOLTIP) == TOOLTIP) {
        switch (action) {
            case MotionEvent.ACTION_UP:
                。。。
                break;

            case MotionEvent.ACTION_DOWN:
                if (event.getSource() == InputDevice.SOURCE_TOUCHSCREEN) {
                    mPrivateFlags3 |= PFLAG3_FINGER_DOWN;
                }
                mHasPerformedLongPress = false;

                if (!clickable) {
                    checkForLongClick(0, x, y);
                    break;
                }

                if (performButtonActionOnTouchDown(event)) {
                    break;
                }

                // Walk up the hierarchy to determine if we're inside a scrolling container.
                boolean isInScrollingContainer = isInScrollingContainer();

                // For views inside a scrolling container, delay the pressed feedback for
                // a short period in case this is a scroll.
                if (isInScrollingContainer) {
                    mPrivateFlags |= PFLAG_PREPRESSED;
                    if (mPendingCheckForTap == null) {
                        mPendingCheckForTap = new CheckForTap();
                    }
                    mPendingCheckForTap.x = event.getX();
                    mPendingCheckForTap.y = event.getY();
                    postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());
                } else {
                    // Not inside a scrolling container, so show the feedback right away
                    setPressed(true, x, y);
                    checkForLongClick(0, x, y);
                }
                break;

            case MotionEvent.ACTION_CANCEL:
                。。。
                break;

            case MotionEvent.ACTION_MOVE:
                if (clickable) {
                    drawableHotspotChanged(x, y);
                }

                // Be lenient about moving outside of buttons
                if (!pointInView(x, y, mTouchSlop)) {
                    // Outside button
                    // Remove any future long press/tap checks
                    removeTapCallback();
                    removeLongPressCallback();
                    if ((mPrivateFlags & PFLAG_PRESSED) != 0) {
                        setPressed(false);
                    }
                    mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
                }
                break;
        }

        return true;
    }

    return false;
}
```

# 二、onClick 和 OnLongClick 

- 因为我们是拿 `ACTION_DOWN` 作为举例的。那么我们先来分析一下 `case MotionEvent.ACTION_DOWN` : 中 `onTouchEvent` 是怎么执行的，以及 `onClick` 和 `OnLongClick` 是如何产生的：

## 2.1 onClick 和 OnLongClick 的产生 

- 首先，当我们手指按下时，有一个 `mHasPerformedLongPress` 标识会先被设为 `false` 。再往下会执行一行 `postDelayed(mPendingCheckForTap` 和 `ViewConfiguration.getTapTimeout())`; 我们来看看这一行的作用：

- 首先，从名字我们就可以猜测，这是个延时执行的方法。我们进一步阅读发现 `mPendingCheckForTap` 是一个 `Runnable` 动作； `ViewConfiguration.getTapTimeout()` 是一个 `100mm` 的延时。也就是说延时 `100mm` 后去执行 `mPendingCheckForTap` 中的动作。那么我们就来看看 `mPendingCheckForTap`  中做了什么：

```java
private final class CheckForTap implements Runnable {
    public float x;
    public float y;

    @Override
    public void run() {
        mPrivateFlags &= ~PFLAG_PREPRESSED;
        setPressed(true, x, y);
        checkForLongClick(ViewConfiguration.getTapTimeout(), x, y);
    }
}
```

- 也就是说，停一百秒后就开始检查，用户的手指是否离开了屏幕。（ 就是当前 `ACTION_DOWN` 之后，有没有触发了 `ACTION_UP` 这个环节 ），但是 `ACTION_DOWN` 后，我们还有一个 `ACTION_MOVE` 过程。在这个 `ACTION_MOVE` 中，如果 `100mm` 内离开了屏幕、或者离开了这个控件就会触发 `ACTION_UP` ，那么就认为这是一个点击事件 `onClick` 。如果没有触发 `ACTION_UP` 的话，就会再延时 `400mm` 。

## 2.2 ACTION_DOWN 之后流程
- 在 `ACTION_DOWN` 之后，会先等 `100mm` 
- 如果没有离开屏幕或者离开控件，就是没有触发 `ACTION_UP` 的话，就会再延时 400mm。
- 这 `500mm` 后就会触发 `onLongClick` 事件。


## 2.3 那么我们现在来验证一下 onLongClick ：

- 首先再 `MainActivity` 中加上：

```java
mMyButton.setOnLongClickListener(new View.OnLongClickListener() {
    @Override
    public boolean onLongClick(View v) {

        return true;
    }
});
```

- 接着，我们发现 `OnLongClick` 是有返回值的，如果返回值是 `false` 还会接着去触发 `onClick` 事件，如果返回 `true` 的话，那么这个长按事件就直接被消费掉了（ 也就是这个点击事件就不会完后传递到 `OnClickListener` 中去了 ）。

## 2.4 总结
- `100mm` 时为点击，`500mm` 时为长按，接着触发长按事件。
- 再看长按事件的返回值，如果时 `true` 就结束。
- 如果时 `false` 那么 `OnClickListener` 就同样也被执行。
- 这就是由 `obTouchEvent` 产生出来的 `onClick/onLongClick` 的来龙去脉。

# 总结
- 我们 `View` 的事件方法，基本上就是这么一个思路，从 `dispatchTouchEvent` 到 `OnTouchListener` 监听器，再到 `onTouchEvent`，接着 `onTouchEvent` 由产生了 `onClick/onLongClick` 。
- 如果大家感兴趣的话可以更深入的去阅读源码。

# 码字不易，你的点 star 是我总结的最大动力！

![Android](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS8xMS84LzE2ZTQ4YWM0NDAwMjZlNTQ?x-oss-process=image/format,png)
