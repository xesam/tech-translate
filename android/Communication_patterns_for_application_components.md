##目标:避免紧耦合

本文对原文进行了精简

原文链接:[Communication patterns for application components](http://vinsol.com/blog/2014/11/04/communication-patterns-for-application-components/)


##紧耦合
组件之间相互持有引用,以及直接调用方法.在下面的代码中,MenuFragment持有MagazineActivity的直接引用,因此, MenuFragment 就与 MagazineActivity紧耦合了.
一旦没有了MagazineActivity,就无法工作了.

    // 紧耦合示例

    class MenuFragment extends Fragment {
        private void onArticleClick(int articleId) {
            MagazineActivity magAct = (MagazineActivity) getActivity();
            magAct.showArticle(articleId);
        }
    }

在这样的设计中,一个类的修改可能会影响到一大波相关的类,如果开发中还涉及到多个开发人员,那么这种情况带来的只会是痛苦和悲哀..

松耦合的核心就是一种减少组件之间的依赖关系,一个松耦合系统可以很方便的分解为良构的元素.这样就使得系统更具有弹性与扩展性.
每个开发人员可以维护一个单独的模块,并使用标准的协议与其他部分通信.

##常规解耦方式:接口
(个人补充:接口并不是单纯指代的java里面的interface关键字,而是指的一种消息规范/协议)
接口是一个强大的解耦工具:类可以通过接口进行通信,而不必要直接引用另一个具体类.一个类可以向外提供一个接口供其他类与之通信.在这个抽象层上,其他类不用关心具体的实现是什么.

    //接口示例

    class MenuFragment extends Fragment {
        public static MenuFragment instantiate(ArticleSwitcher articleSwitcher) {
            MenuFragment menuFragment = new MenuFragment();
            menuFragment.articleSwitcher = articleSwitcher;
            return menuFragment;
        }

        ArticleSwitcher articleSwitcher;

        public void onArticleClick(int articleId) {
    	    articleSwitcher.showArticle(articleId);
        }
    }

使用接口的缺点:

1. 组件之间还是需要相互了解以传递接口,部分依赖依旧存在
2. 接口不能通过intent传递
3. 在大项目中,接口数量会急剧增加,导致大量的模板代码
4. 当接口在组件之间传递时会形成接口链,导致复杂度上升

##优雅解决方案:消息总线

基于发布/订阅者模式的通信方案.发布者发布一个通知,订阅者得到通知并做出反应,因此,发布者和订阅者之间就实现了解耦.

###实现消息总线

####隐式intent + BroadcastReceiver

通过隐式Intent的通信可以看作是消息队列的一种形式:
sendBroadcast(Intent) 或者 startActivity(Intent)是发布者方法,含有相应IntentFilter的BroadcastReceiver 或者 Activity则扮演订阅者的角色.

优点与缺点

优点:可以跨应用传递,也是Android里面的标准方法

缺点:无法传递复杂数据,必须通过bundle来传递

[FluffyEvents](https://github.com/alexvasilkov/fluffy-events) 就是一个通过 BroadcastReceiver 实现的消息总线.


#### EventBus: 基于事件的消息总线

("事件"本身也是一种消息,原理还是订阅/发布者模式)

事件本身可以是任何java类,如下:

    // 一个示例事件类

    class DownloadProgressEvent {
        private float progress;
        public DownloadProgressEvent(float progress) {
            this.progress = progress;
        }

        public float getProgress() {
            return progress;
        }
    }

#####EventBus: 大致原理

EventBus使用合适的数据结构来维持事件以及订阅者的对应关系,比如Otto使用的方式:

    /** Cache event bus subscriber methods for each class. */
    private static final Map<Class<?>, Map<EventClass, Set>> SUBSCRIBERS_CACHE = new HashMap<Class<?>, Map<EventClass, Set>>();

每当有订阅者注册或者注销的时候,都会同时更新相应的对应关系,通常,Activity 或者 Fragment会在onResume阶段进行注册,在onPause阶段进行注销.(译者注:可以减少内存泄露的问题)

每当有事件发布的时候,EventBus遍历整个对应关系,找到所有符合条件的订阅者方法并执行.

#####Sticky Event

类似Android原生的StickyBroadcast,使得一个订阅者在任何时候注册都可以获得这个事件.

##总结

由于EventBus并不是Android的内在机制,因此无法跨应用传递事件.

比较常用的两个EventBus开源类库:

[Square’s Otto](http://square.github.io/otto/):基于注解,在Guava EventBus基础上针对Android平台进行了优化

[GreenRobot’s EventBus](https://github.com/greenrobot/EventBus):更好的线程分发机制.

(译者注: EventBus比较适合组件之间的解耦, 而接口是一个更抽象的概念,还可以用在view或者其他方面)



####Android分享 Q群：315658668








