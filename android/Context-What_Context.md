#What Context?
英文原文：[Context, What Context?](http://possiblemobile.com/2013/06/context/)

##Context

Context 估计是 Android 开发中最常用的元素了，它的获取和使用如此普遍，加载资源，启动新的 Activity，获取系统服务，获取内部文件路径以及创建 View 都离不开 Context。同时，Context 也是最容易误操作的元素，以致于很容易把你带到坑里面去。下面就让我们全面对比了解一下 Context，让你的开发更得心应手。

##Context 类型

不同类型的Context各异：根据 Android 组件的不同，获得到的 Context 也是不同的。

###Application

Application 是存在于 app 进程的一个单例 Context 对象。在 Activity 或者 Service 中，可以使用 getApplication() 方法获取这个 Application 对象。除此之外，从他继承了 Context 的组件，都可以通过 getApplicationContext() 来获取到这个 Application 对象。不过，不论是通过什么方法获得，最后得到的 Application 对象都是同一个。

*【译者注：这个单例 Context 对象在后文用 application context 指代】*

###Activity/Service

Activity/Service 继承自同一个基类 Context —— ContextWrapper，因此两者拥有相同的 Context API，但是具体任务还是通过将调用委托代理给实际的内部对象来完成。每当你创建一个新的 Activity 或者 Service 的时候，同时就会创建一个新的 ContextImpl，ContextImpl 就是最终处理所有 Context API 方法的内部对象。不同的Activity 或者 Service 的 Context 都是不一样的。

###BroadcastReceiver

BroadcastReceiver 本身并不是一个 Context，但是 Android framework 会在每一个广播事件发生的时候，给相应 BroadcastReceiver 的 onReceive() 传递一个 Context，这个 ReceiverRestrictedContext 有个两个方法不可用： registerReceiver() 和 bindService()。BroadcastReceiver 每次处理 broadcast 的时候，传递给它的 Context 都是一个新的实例。

###ContentProvider

ContentProvider 本身也不是 Context，但是调用 getContext() 可以获取一个 Context。如果调用者与 ContentProvider 运行在同一个进程内，那么这个返回的 Context 就是上文的 application context。如果调用者与 ContentProvider 不是运行在同一个进程之内，那么这个方法会返回一个指代 provider 所在包的新 Context 实例。

##保存 Context 引用

第一个问题：当我们将一个 Context 的引用保存到一个存活时间比 Context 本身生命周期还长的对象时，问题就来了。比如，我们有一个单例对象，要求使用一个 Context 来执行资源加载或访问 ContentProvider，并且传入的是一个 Activity 或 Service 对象：

错误的 Singleton 实现：

    public class CustomManager {
        private static CustomManager sInstance;

        public static CustomManager getInstance(Context context) {
            if (sInstance == null) {
                sInstance = new CustomManager(context);
            }

            return sInstance;
        }

        private Context mContext;

        private CustomManager(Context context) {
            mContext = context;
        }
    }

我们知道，单例对象是一个静态变量，受其所在类的生命周期控制。这就意味着，在这个单例对象的存活时间内，这个单例持有的对象（引用）都不会被垃圾回收。
所以这种实现的问题就是你无法知道 Context 到底来自何处，如果这个 Context 是一个 Activity 或者 Service， 就会变得不安全：这个被持有的 Activity，以及其内部所有的 View 或者其他的耗内存对象都无法被回收，从而引发内存泄露。

为了防止这种问题，我们改为让单例持有 application context：

改进的实现：

    public class CustomManager {
        private static CustomManager sInstance;

        public static CustomManager getInstance(Context context) {
            if (sInstance == null) {
                //Always pass in the Application Context
                sInstance = new CustomManager(context.getApplicationContext());
            }

            return sInstance;
        }

        private Context mContext;

        private CustomManager(Context context) {
            mContext = context;
        }
    }

如此以来，我们就不用再关心 Context 来自哪里，也不用关心 Context 是什么类型，因为最终持有的都会是 application context，因此就避免了内存泄露的问题。这个处理技巧在后台线程或者 Handler 处理中同样有效。

既然如此，是不是意味着我们可以在任何情况下都用 application context 来处理呢？这样就永远不用担心内存泄露了。答案显然是否定的，就如上文说的一样，不同情况下的 Context 各不相同。这就像葫芦娃一样，虽然都是葫芦娃，但是每个娃的技能都不一样。

##Context 特点

Context 能实现哪些功能，主要还是取决于 Context 从何而来，下表列出了不同 Context 的一些不同点：

<table border="1" width="90%" align="center">
<thead>
<tr>
<th></th>
<th align="center">Application</th>
<th align="center">Activity</th>
<th align="center">Service</th>
<th align="center">ContentProvider</th>
<th align="center">BroadcastReceiver</th>
</tr>
</thead>
<tbody>
<tr>
<td>Show a Dialog</td>
<td align="center">NO</td>
<td align="center">YES</td>
<td align="center">NO</td>
<td align="center">NO</td>
<td align="center">NO</td>
</tr>
<tr>
<td>Start an Activity</td>
<td align="center">NO<sup>1</sup></td>
<td align="center">YES</td>
<td align="center">NO<sup>1</sup></td>
<td align="center">NO<sup>1</sup></td>
<td align="center">NO<sup>1</sup></td>
</tr>
<tr>
<td>Layout Inflation</td>
<td align="center">NO<sup>2</sup></td>
<td align="center">YES</td>
<td align="center">NO<sup>2</sup></td>
<td align="center">NO<sup>2</sup></td>
<td align="center">NO<sup>2</sup></td>
</tr>
<tr>
<td>Start a Service</td>
<td align="center">YES</td>
<td align="center">YES</td>
<td align="center">YES</td>
<td align="center">YES</td>
<td align="center">YES</td>
</tr>
<tr>
<td>Bind to a Service</td>
<td align="center">YES</td>
<td align="center">YES</td>
<td align="center">YES</td>
<td align="center">YES</td>
<td align="center">NO</td>
</tr>
<tr>
<td>Send a Broadcast</td>
<td align="center">YES</td>
<td align="center">YES</td>
<td align="center">YES</td>
<td align="center">YES</td>
<td align="center">YES</td>
</tr>
<tr>
<td>Register BroadcastReceiver</td>
<td align="center">YES</td>
<td align="center">YES</td>
<td align="center">YES</td>
<td align="center">YES</td>
<td align="center">NO<sup>3</sup></td>
</tr>
<tr>
<td>Load Resource Values</td>
<td align="center">YES</td>
<td align="center">YES</td>
<td align="center">YES</td>
<td align="center">YES</td>
<td align="center">YES</td>
</tr>
</tbody>
</table>

1. application context 可以启动 Activity， 但是前提是需要创建一个新任务。在某些情况下，我们可以使用这种方式实现某种特殊目的，这种方式会创建一个非标准的回退栈，一般不推荐使用，至少不是一个好的实践。
2. 这个是合法的调用，但是 inflation 获得的 View 只会应用系统的主题，而不是当前 app 的自定义主题。
3. 在 4.2 及以上系统版本中， 允许注册 receiver 为 null 的广播监听，主要目的是为了获取 sticky broadcast 的当前值。

##User Interface

从上面的列表可以看到，application context 不能胜任很多场景，而且都是与UI相关的情况。实际上，只有 Activity 拥有处理 UI 的能力，其他类型的 Context 在这方面都大同小异。

上面的三种行为，除了 Activity， 其他的 Context 也都无法处理，从而避免误用。试图显示一个使用 application context 创建的 Dialog，或者从 application context 启动一个Activity，都会导致 app 崩溃，系统通过这种方式告诉你：你用错了。

另一个问题就是 inflating layout。如果你读过 [layout inflation](http://www.doubleencore.com/2013/05/layout-inflation-as-intended/)一文， 你就会知道 inflating layout 有一些容易让人迷惑的地方，如何正确使用 Context 就是其中一个。如果你使用 application context 来进行 inflating layout，并不会发生任何错误，但是当前 app 所定义的 themes 以及 styles 都被忽略了。究其原因，正如你在 manifest 中定义的一样， 只有 Activity 才是唯一能够响应 themes 定义的组件。任何其他组件所含有的 Context 都会应用 Android 系统自身的主题，所以，最终的 UI 表现可能出乎你的意料。

##规则冲突

可能有人会提出这样一种场景：在 app 的当前设计下，由于涉及到处理 UI 的操作，所以需要长期持有一个 Activity 的引用。如果是这样的话，我只能说：请重新考虑你的设计。

##经验法则

简而言之，在组件的生命周期之内可以直接使用自身的 Context， 一旦需要在超出组件生命周期之外的对象中使用 Context ，就应该只用 application context，哪怕只是临时的引用，也是如此。


####Android分享 Q群：315658668