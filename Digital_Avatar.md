# Unity 实现数字人

## 如何实现数字人

通过语音或者触摸屏幕方式进行文本输入，将文本转化成可以驱动数字人模型执行动作的指令。

路径拆解：

1. 文本转成数字人模型输入指令，技术点：Android跟Unity通信通道，网路请求（云端将文本转成操作模型的指令）
2. 数字人模型的生成 技术点：美工设计导出，需要什么格式，什么规范要求，方便在Unity编辑工具中加载控制
3. 性能数据和优化方案 技术点：测量性能方法，评估维度指标，优化方案

### Unity 和 Android 工程开发调试

Unity 可以方便导出 Android 工程，包括 launcher 和 unitylibrary 两部分，Android 工程只需要直接依赖 unitylibrary 就可以直接编译运行

Android 工程开发工具Android Studio

Unity 工程开发工具 Unity 

Unity 脚本编辑工具 Visual Studio

Unity 工程导出unitylibrary 到 Android 工程集成

### Unity 和 Android 工程通信方式

详见 https://github.com/Unity-Technologies/uaal-example.git

Limitations Unity作为Android 工程一个library包含进APK的限制

While we tested many scenarios for Unity as a library hosted by a native app, Unity does not control anymore the lifecycle of the runtime, so we cannot guarantee it'll work in all possible use cases. For example:

* Unity as a Library supports rendering only full screen, rendering on a part of the screen isn’t supported. 只能全屏幕渲染，不能局部渲染
* Loading more than one instance of the Unity runtime isn’t supported. 只能一个Unity runtime示例
* You may need to adapt 3rd party Plug-ins (native or managed) to work properly. 可能需要三方插件
* Overhead of having Unity in unloaded state is: 90Mb for Android and 110Mb for iOS. 内存占用 Android 90Mb / iOS 110Mb


#### Unity 调用 Android 

Unity 可以调用 Android 静态和非静态代码

##### Unity 调用 Android 非静态代码方式：

Unity 代码

    AndroidJavaClass jc = new AndroidJavaClass ("com.unity3d.player.UnityPlayer");
    AndroidJavaObject jo = jc.GetStatic<AndroidJavaObject> ("currentActivity");
    jo.Call ("login","");

Android 代码

    public void login( String str ) {      
        // 写上自己的操作
    }

Android这边的login()不一定要写在com.unity3d.palyer包名下的UnityPalyer类下。

你只需要把login()写在你自己定义包名下的UnityPlayerActivity.java中就可以了。当然了，该类肯定是继承Activity的。

你可以把鼠标放在unity代码的Call上查看方法可以填写的参数， 你会发现方法可以填写的参数可以是params object[]。也就是可以传递多个参数，以数组的形式传递给Android。

如果需要传递多个参数的话，

Unity 代码 

    public void PKBtnClick() {
    　　this.test("test1", "test2", "test3");
    }

    public void test( params object[] args ){
    　　AndroidJavaClass jc = new AndroidJavaClass ("com.unity3d.player.UnityPlayer");
    　　AndroidJavaObject jo = jc.GetStatic<AndroidJavaObject> ("currentActivity");
    　　jo.Call ("login", args);
    }

或者

    public void PKBtnClick() {
    　　AndroidJavaClass jc = new AndroidJavaClass ("com.unity3d.player.UnityPlayer");
    　　AndroidJavaObject jo = jc.GetStatic<AndroidJavaObject> ("currentActivity");
    　　jo.Call("login", "test1", "test2", "test3");
    }

Android 代码

    public void login( String str1, String str2, String str3 ) {
            Log.e("test", str1 + "==" + str2 + "==" + str3 );
    }

错误接收例子

    public void login( String []str1 ) {        
        Log.e("test", str1[0] + "==" + str1[1] + "==" + str1[2] );
    }

unity传递多少个参数，java就必须接收多少个参数，不然肯定会报错的！！

如果unity传递给Android整形数据或者布尔值

Unity 代码

    public void PKBtnClick() {
        this.test("test1", "test2", "test3", 1, true);
    }

    public void test( params object[] args ){
        AndroidJavaClass jc = new AndroidJavaClass ("com.unity3d.player.UnityPlayer");
        AndroidJavaObject jo = jc.GetStatic<AndroidJavaObject> ("currentActivity");
        jo.Call ("login", args);
    }

Android 代码

    public void login( String str1, String str2, String str3, int a, boolean isShow ) {
            if( isShow ){            
                Log.e("test", str1 + "==" + str2 + "==" + str3 + "==" + a );
            }
    }


#####  Unity 调用 Android 非静态方法

Unity 代码

    public void PKBtnClick() {
        this.test("test1", "test2", "test3", 1, true);
    }

    public void test( params object[] args ){
        AndroidJavaClass jc = new AndroidJavaClass ("com.Indra.Dark.UnityPlayerActivity");
        jc.CallStatic ("login", args);
    }

Android 代码

    public static  void login( String str1, String str2, String str3, int a, boolean isShow ) {
        if( isShow ){            
            Log.e("test", str1 + "==" + str2 + "==" + str3 + "==" + a );
    　　}
    }


#### Android 调用 Unity

UnityPlayer.UnitySendMessage("Main Camera", "AgentPurchaseCancelled",msg);

使用android 脚本向 unity发送消息，

参数一为unity脚本挂载的gameobject

参数二为unity脚本中要调用的方法名

参数三为传递的数据

当unity脚本中的方法为静态方法时，这个方法无效，所以只能调用非静态的方法

例子：

    UnityPlayer.UnitySendMessage("btnTest", "showLog", "a#b#c");

UnitySendMessage的第一个参数是unity控件的名字，第二个参数是方法名，第三个参数是要传递的参数。而且只能传递一个参数（感觉好坑）。不过还好。你可以把你要传递的参数做成一个字符串传递过去，unity那边做分割字符串就行了。例如上面就是把a,b,c用#连接起来。unity那边用#作为分隔符去分割就OK了。

那么问题来了，脚本挂在的unity控件名字不好找怎么办。其实还有个办法轻松搞定，那就是在脚本的Start()方法中指定name为你方法传递的控件名字就ＯＫ了。

如我上面的方法中要找的控件是btnTest，则：

unity代码：

    void Start () {
        this.name = "btnTest";
    }

    void showLog（string str）{
    　　Debug.Log（"str:" + str);
    }


#### 回调

#### Android 和 Unity 通信组件封装

浅谈Unity与Android原生的桥接



https://juejin.cn/post/6844904165760565261


#### Android 和 Unity 开发常见错误

https://docs.unity3d.com/2019.3/Documentation/Manual/TroubleShootingAndroid.html


### Unity 模型的生成方式


## 参考

[Unity上玩转数字人（Avatar）](https://blog.csdn.net/grace_yi/article/details/125072726)

[Unity 学习路径](https://ac.csdn.net/unity/)

[Unity移动设备输入](https://docs.unity.cn/cn/current/Manual/MobileInput.html)

[unity和Android交互](https://www.cnblogs.com/colored-mr/p/5677209.html)

[浅谈Unity与Android原生的桥接](https://juejin.cn/post/6844904165760565261)