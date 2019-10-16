# VFramework
private unity framework of Evideo 

## 原则
>精简、模块化、易用、跨平台、低耦合
>算法类、工具类放框架；业务类封装放插件

## 入门

1. 拷贝** VFDLL.dll、Newtonsoft.Json.dll、zxing.unity.dll **到项目中
2. 打开Unity编辑器 ** Edit -> Project Settings -> Player -> Configuration **。
    Script Runtime Version 设置为 .Net 3.5以上
	API Compability Level 设置为 .Net 3.5以上
3. 脚本中引用功能相关包名即可

```
using UnityEngine;
using VFramework.Net;
using VFramework;
using UnityEngine.Networking;
public class HttpTest : MonoBehaviour
{
    void Start()
    {     
		//http get方法
        new VHttp<DownloadHandlerBuffer>("www.baidu.com")
            .SetHeads(null)
            .SetTimeout(10)
            .OnError(res => this.LogW("下载失败：" + res))
            .OnSuccess(res => this.LogI("获取结果：" + res.text))
            .Start();
    }
}

``` 

## 功能(详细见Samples目录)

### Singleton 单例模板

继承Singleton可实现普通单例，继承MonoSingleton实现带Unity脚本生命周期的单例。
```
namespace VFramework.Example{

    using UnityEngine;
    using VFramework;

    public class SimpleSingleton : Singleton<SimpleSingleton> {

        private SimpleSingleton() { }

        public override void OnSingletonInit() {
            base.OnSingletonInit();
            Debug.Log("初始化成功");
        }

        public override void Dispose() {
            base.Dispose();
            Debug.Log("反初始化");
        }

        int i = 0;

        public void PrintLog() {
            Debug.Log("i = " + i++);
        }
    }

    public class MonoSingleton1 : MonoSingleton<MonoSingleton1> {
        private MonoSingleton1() { }

        public override void OnSingletonInit() {
            base.OnSingletonInit();

        }

        public override void Dispose() {
            base.Dispose();
        }


        int i = 0;

        public void PrintLog() {
            Debug.Log("i = " + i++);
        }

        public void Start() {
            //StartCoroutine("");
        }

        public void Awake() {
            print("Awake");
        }

        private new void OnDestroy() {
            print("OnDestroy");
        }
    }
}

```
### EventBus 消息总线

订阅&反订阅：
```
EventBus.Register("CutSong", OnEvent);
EventBus.Register("CutSong", (key, param) => this.LogI("收到切歌消息"));
EventBus.UnRegister("CutSong", OnEvent);
private void OnEvent(string key, params object[] param) {
	//do sth..
}
```
发布消息：
线程安全，可在子线程发布。
通知方式参考Android EventBus。分为当前线程通知、转到子线程通知、转到主线程通知。
默认当前线程通知
```
EventBus.Post("CutSong", "test", 1);//当前线程通知
EventBus.Post("CutSong", EventBus.POST_THREAD.Main, "test", 1);//主线程通知
EventBus.Post("CutSong", EventBus.POST_THREAD.BackGround, "test", 1);//子线程通知

```

### Loom 异步调度中心
用来调度异步任务
```
Loom.InvokeAsync(()=>this.LogI("async log")); //子线程调用
Loom.InvokeMain(()=>this.LogI("main thread log")); //主线程调用
Loom.Instance.StartCoroutine(DownLoad()); //委托启动协程

```

### FSM 有限状态机模板
实现状态切换

创建：
```
mPlayerFsm = new FSM()
        .AddTranslation(STATE_RUN, EVENT_TOUCH_DOWN, STATE_JUMP, JumpThePlayer) //跑步状态经过触摸屏事件变为跳跃状态
        .AddTranslation(STATE_JUMP, EVENT_TOUCH_DOWN, STATE_DOUBLE_JUMP, DoubleJumpThePlayer)
        .AddTranslation(STATE_JUMP, EVENT_LAND, STATE_RUN, RunThePlayer)
        .AddTranslation(STATE_DOUBLE_JUMP, EVENT_LAND, STATE_RUN, RunThePlayer)
        .Start(STATE_RUN); //初始状态

void JumpThePlayer(object[] args)
{
	Debug.Log("让Player跳跃");
}
```

状态切换：
```
mPlayerFsm.HandleEvent(EVENT_TOUCH_DOWN); //触摸屏按下事件
```

### VLog 日志模块
* 日志分为四个等级，LogD、LogI、LogW、LogE，分别对应调试、信息、警告、出错四种日志类型。

* 分为两种运行模式。DEBUG、RELEASE。
Debug模式下，四种日志都会打印，并显示调用者的对象名、类名、方法名。(方法名获取采用反射的方式，故较耗费性能)
``` D/Main Camera (LogTest).Start() ==> 日志测试 ```
Release模式下，不打印LogD日志，发布到市场时用。日志内容只显示调用者对象名、类名。方法名不显示。
``` W/Main Camera (LogTest) ==> 日志测试7 ```
LogE出错日志默认方法名、类名、对象名全打印。

调用方式：
```
this.LogD("日志测试");
VLog.LogW("日志测试7");
new Test().LogE("日志测试8");
new Thread(()=>this.LogW("日志测试3")).Start();
```
** 尽量使用this.LogX() **

UI显示：
```
VLog.SetVisible(true);
VLog.SetVisible(true, new Rect(100, 100, 500, 500)); //控制显示区域
```

### Net 网络操作模块(待完善)
目前仅支持http的get方法，后续增加http post、socket等

链式方式：
```
new VHttp<DownloadHandlerTexture>("http://192.168.82.249:9266/stb/photomv/3.jpg")
    .SetHeads(null)
    .SetTimeout(10)
    .OnError(res => this.LogW("====>下载失败：" + res))
    .OnSuccess(res => txrResult= res.texture)
    .Start();
```
Unity自带的利用DownloadHandler的派生类，可加载纹理、text、文件、音频等资源。

普通方式：
```
//下载文件
VHttp.DownloadFile("http://192.168.82.249:9266/stb/photomv/1.jpg",
    @"E:\m\mm\mmm\1.jpg", bSuccess => this.LogI("下载结果：" + bSuccess));

//下载纹理
VHttp.GetTexture("http://192.168.82.249:9266/stb/photomv/2.jpg", txr => txrResult = txr);

//获取数据
VHttp.GetText("www.baidu.com", txr => {
    if (string.IsNullOrEmpty(txr)) {
        this.LogI("获取失败");
    } else { 
        this.LogI("获取到：" + txr);
    }
});
```

### Kits 工具类

#### Json解析

将对象序列化为字符串
```
        Person ps = new Person(22) {
            IsMarry = false
        };
        //序列化对象成字符串
        string json = JsonTool.ToJsonStr(ps);
        string json1 = ps.ToJsonStr(); //this扩展
        Debug.Log(json);
        Debug.Log(json1);
```

将字符串反序列化为对象
```
        //反序列化json字符串成对象
        Person ps1 = json.ToJsonObj<Person>(); //this扩展
        Person ps2 = JsonTool.ToJsonObj<Person>(json); //泛型
        Person ps3 = new Person(21).LoadFromJsonStr(json); // this扩展
```

#### XML解析

将对象序列化并保存
```
		//对象序列化为xml文档
		XmlTool.SaveToXml(city1, @"E:\tmp\output004.xml");
		city1.SaveToXml(@"E:\tmp\output005.xml"); //类的this扩展
```

将文件读取并序列化到对象
```
        //xml数据序反列化为对象
        City city3 = XmlTool.LoadFromXml<City>(@"E:\tmp\output005.xml"); //泛型
        City city4 = new City().LoadFromXml(@"E:\tmp\output005.xml"); //this扩展
```

#### 二维码

生成二维码：
```
		rawImg.texture = BarCode.GetTexture("www.baidu.com");
```
#### LimitQueue
定长队列，用法跟普通Queue一致。当Queue达到上线时，继续Enqueue会自动剔除队首item，线程安全。

```
LimitQueue<LogElement> m_queStrLogs = new LimitQueue<LogElement>(50); //新建，上限50
m_queStrLogs.Enqueue(item); //入队
m_queStrLogs.Dequeue(); //出队
foreach(var item in m_queStrLogs){ //遍历
	//do sth..
}
```

#### 文件操作
略

## 下版本规划
	* Socket 封装，tcp、udp [参考](https://blog.csdn.net/subin_iecas/article/details/80289915)
	* EventBus 跨进程&跨设备
	* Queue\List\Dictionary\HashTable封装。
	* 引入链式编程 & UniRx E:\code\unity\UniRX\UniRxMVCExample
	* 参考QFramework UIKit | UniRx 响应式属性&响应式事件
	* 参考QFramwwork ResKit，支持资源resource目录读取、本地读取、远程读取
	* 参考UniRx、QFramework，引入与蛇体初始化、设置对象大小等封装
	* 参考QFramework 区分测试、研发阶段，分角色
	* 【待定】ijk播放器、hdmi-in放到Framework OR plugin。

## 鸣谢

>六方集团首席网络安全专家--噶抓同志