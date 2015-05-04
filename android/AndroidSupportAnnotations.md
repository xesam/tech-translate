#Android Support Annotations

[查看原文](http://anupcowkur.com/posts/a-look-at-android-support-annotations/)

在Support Library 19.1以及以后的版本中，Android工具小组引入了几个很酷的注解类型，方便开发者在工程中使用，同时Support Library自身也使用了这些注解。

本文的代码都使用android studio完成。首先，添加注解支持：

    compile 'com.android.support:support-annotations:22.1.1'

有三种类型的注解：

1. NonNull & Nullable
2. 资源Id
3. IntDef & StringDef

##Nullness注解
@NonNull 用来修饰不能为null的参数。在下面的代码例子中，我们有一个取值为 null 的 name 变量，它被作为参数传递给 sayHello 方法，而该方法要求这个参数是非null的String类型：

    public class MainActivity extends ActionBarActivity {

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            String name = null;
            sayHello(name);
        }

        void sayHello(@NonNull String name) {
            Toast.makeText(this, "Hello " + s, Toast.LENGTH_LONG).show();
        }

    }

由于代码中参数 name 被 @NonNull 注解修饰，因此 android studio 将会以警告的形式提醒我们这个地方有问题：

![name_is_null_warning](http://anupcowkur.com/images/name_is_null_warning.png)

如果我们给 name 赋值，例如

    String name = “Our Lord Duarte”

那么警告将消失。

【注：这个我试了一下，在android studio里面，就算不用注解，也会有提示的，android studio 就是这么智能。】

@Nullable 用来修饰方法的参数或者返回值可能为 null。示例如下：

    public class MainActivity extends ActionBarActivity {

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            Toast.makeText(this, getName().toLowerCase(), Toast.LENGTH_LONG).show();
        }

        @Nullable
        String getName() {
            return "";
        }

    }

因为 getName 方法的返回值使用 @Nullable 修饰，所以 android studio 会提示

    Method invocation "getName().toLowerCase()" may produce "java.lang.NollPointerException"

##资源注解

资源类型注解可以帮助我们准确的使用资源id，例如，避免我们在要求colorId的地方错误的使用了dimenId。
在下面的代码中，我们的sayHello方法预期接受一个字符串类型的资源Id，并使用@StringRes注解修饰：

    public class MainActivity extends ActionBarActivity {

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);

            sayHello(R.style.AppTheme);
        }


        void sayHello(@StringRes int id) {
            Toast.makeText(this, "Hello " + getString(id), Toast.LENGTH_LONG).show();
        }

    }

而我们传递给它的是一个样式资源Id，与预期的字符串资源Id不符合，这时IDE将提示警告如下：

![wrong_resource_type_error](http://anupcowkur.com/images/wrong_resource_type_error.png)

类似的，我们把警告的地方使用一个字符串资源Id代替警告就消失了：

    sayHello(R.string.name);

基本上，每一种资源类型都有相应的资源注解

    AnimatorRes
    AnimRes
    AnyRes
    ArrayRes
    AttrRes
    BoolRes
    ColorRes
    DimenRes
    DrawableRes
    FractionRes
    IdRes
    IntegerRes
    InterpolatorRes
    LayoutRes
    MenuRes
    PluralsRes
    RawRes
    StringRes
    StyleableRes
    StyleRes
    XmlRes

##IntDef和StringDef注解

最后一种类型的注解是基于Intellij的[“魔数”检查机制](http://blog.jetbrains.com/idea/2012/02/new-magic-constant-inspection/)功能

【注：“魔数”就是那些不能看出有什么含义的数字常量，这里也包括字符串常量】

很多时候，出于性能的考虑，我们会使用整型常量代替枚举类型。例如我们有一个IceCreamFlavourManager类，它定义三种操作：

    VANILLA
    CHOCOLATE
    STRAWBERRY

我们可以定义一个名为@Flavour的新注解，并使用@IntDef指定它可以接受的取值范围，如下例所示：

    public class IceCreamFlavourManager {

        private int flavour;

        public static final int VANILLA = 0;
        public static final int CHOCOLATE = 1;
        public static final int STRAWBERRY = 2;

        @IntDef({VANILLA, CHOCOLATE, STRAWBERRY})
        public @interface Flavour {
        }

        @Flavour
        public int getFlavour() {
            return flavour;
        }

        public void setFlavour(@Flavour int flavour) {
            this.flavour = flavour;
        }
    }

这时如果我们使用直接字面量来调用IceCreamFlavourManager.setFlavour，IDE将提示错误如下：

![wrong_flavour_error](http://anupcowkur.com/images/wrong_flavour_error.png)

IDE甚至会提示我们可以使用的有效取值：

![ide_suggests_flavours](http://anupcowkur.com/images/ide_suggests_flavours.png)

我们也可以指定整型取值可以用作标志，也就是说这些整型值可以使用’｜’或者’&’进行与或等操作。如果我们定义@Flavour如下：

    @IntDef(flag = true, value = {VANILLA, CHOCOLATE, STRAWBERRY})
        public @interface Flavour {
    }

那么可以进行如下调用：

    iceCreamFlavourManager.setFlavour(IceCreamFlavourManager.VANILLA & IceCreamFlavourManager.CHOCOLATE);

@StringDef 用法与 @IntDef 基本差不多，只不过是针对String类型值而已。

更多信息可以参考[tools site](http://tools.android.com/tech-docs/support-annotations)

####Android分享 Q群：315658668