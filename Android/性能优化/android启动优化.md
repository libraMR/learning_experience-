# android启动优化。
**启动分类**

**App startup time**

**1.冷启动**

**2.热启动**

**3.温启动**





**一.冷启动**

**特点：耗时最多，衡量标准**

 **Click Event -----> IPC -----> Process.start -----> ActivityThread --------> bindApplication ---------> LifeCycle ------> ViewRootImpl**

![image](images/76dcea6f-cb06-4871-8af4-b4bcfe716400.jpg)



**二.热启动**

**特点：最快**

 **后台 ---------> 前台**

![image](images/c2512ae9-988d-49e3-b6f7-123cbf43f8ee.jpg)



**三.温启动**  

**特点：较快**(介于冷启动和热启动之间的速度)；只会重走Activity生命周期，而不会重走那些进程的创建还有Application的创建以及Application的生命周期等。\*\*\*\*  

 **LifeCycle**

\*\*

![image](images/fa99a717-1328-4b53-a782-cfcc29d06290.jpg)





**一.启动时间测量方式**  

1.adb命令：

    adb shell am start -W packagename/首屏\*\*

\*\*

packagename.\*\*

activity

\*\*

\*\*

\*\*

特点：

    a.线下使用方便，不能带到线上

    b.非严禁、精确时间  

\*\*\*\*\*\*

![image](images/b6f30ed9-29bf-41ce-830f-6d4d2f134d30.jpg)



\*\*2.手动打点（推荐使用）：

\*\*

特点：

    a.精确，可带到线上，推荐使用

    b.避开误区，采用feed第一条显示

    c.addOnDrawListener要求API 16

\*\*

             \*\*\*\*\*\*\*\*\*\*\*\*\*\*

![image](images/60981680-4378-40a9-95b3-49503c76d4fb.jpg)

             

![image](images/49fbf008-7f16-4c8f-bb30-0479800293ec.png)

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*             

```java
    private boolean mHasRecorded;
    private OnFeedShowCallBack mCallBack;
    @Override
    public void onBindViewHolder(@NonNull final ViewHolder holder, int position) {
        if (position == 0 && !mHasRecorded) {
            mHasRecorded = true;
            holder.layout.getViewTreeObserver()
                    .addOnPreDrawListener(new ViewTreeObserver.OnPreDrawListener() {
                        @Override
                        public boolean onPreDraw() {
                            holder.layout.getViewTreeObserver().removeOnPreDrawListener(this);
                            LogUtils.i("FeedShow");
                            LaunchTimer.endRecord("FeedShow");
                            if (mCallBack != null) {
                                mCallBack.onFeedShow();
                            }
                            return true;
                        }
                    });
        }
    }
```
\*\*

\*\*

\*\*

\*\*

\*\*

LaunchTimer代码

\*\*

\*\*

\*\*

\*\*

\*\*

```java
import android.util.Log;

public class LaunchTimer {
    private static long sTime;
    public static void startRecord(){
        sTime = System.currentTimeMillis();
    }

    public static void endRecord(){
        long cost = System.currentTimeMillis() - sTime;
        Log.d("LaunchTimer", "endRecord: " + cost);
    }
}
```
\*\*

**二.启动优化工具选择**

**注意：**

\*\*    **a.两种方式互相补充**    **b.正确认识工具及不同场景选择合适的工具**

---
1.traceview

特点：

\*\*\*\*\*\*\*\*\*\*    \*\*\*\*\*\*\*\*\*\*a.图形的形式展示执行时间、调用栈等

    b.信息全面，包含所有线程  

    c.运行开销严重，整体都会变慢

    d.因为c点可能会带偏优化方向

    e.traceview与cpu profiler，traceview可以更好的埋代码



使用代码：\*\*\*\*\*\*\*\*\*\*

```java
Debug.startMethodTracing("APP");
......
Debug.stopMethodTracing();
```
---
**编译运行APP生成文件，在sd卡：Android/data/packagename/files下**

打开App.strace文件：

![image](images/d9ad8dda-5e86-4f6b-884c-4b986f68eec0.jpg)



---
Call Chart

---
---
![image](images/bcb30afe-d564-4553-9672-fde8045ee278.jpg)

---
---
Flame Chart(火焰图，收集相同的调用顺序)

---
![image](images/6f66c52c-8d24-469a-b9ca-ee70d7acccee.jpg)



Top Down(wall clock time、thread time)

![image](images/af7a8f16-7181-4a84-a26b-b94bf7dc9261.jpg)

![image](images/1e60d526-3242-4737-9a15-84d2c6126380.jpg)

 

![image](images/7068013b-caac-4640-ad1c-10564a7d9290.jpg)

---
Bottom UP(B在上A在下，和Call Chart、Top Down显示效果相反)

---
![image](images/f66d3bb3-37bf-4baf-8f9b-618e61cee01b.jpg)



2.systrace

---
特点：

\*\*\*\*\*\*\*\*\*\*   **a.结合Android内核的数据，生成html报告**

   **b.API 18以上使用，推荐TraceCompat（向下兼容）**

    **c.轻量级，开销小**

    **d.直观反应cpu利用率**

    **e.cpu time（**cpu duration**）与wall time（**wall **duration\*\*\*\*）的区别**

        **1.cpu time（cpu duration）是代码消耗cpu的时间（重点指标）**

        **2.wall time（**wall **duration\*\*\*\*）是代码执行时间**

**使用代码：**

```java
TraceCompat.beginSection("AppOnCreate");
......
TraceCompat.endSection();
```
**使用python指令生成文件：**

     \*\*python systrace.py -t 10 \[other-options\] \[categories\]
\*\*

\*\*

![image](images/f8e18ca3-4e38-4e1b-8896-388532901e36.jpg)





\*\*

**资料网站：****https://developer.android.com/studio/command-line/systrace#command\_options**

**打开生成的html文件：** 

![image](images/a613ad89-7caa-4814-8aae-49f2184feeaa.jpg)



\*\*

**三.优雅获取方法耗时**

![image](images/6a9e2aaf-19dd-4804-a864-899a92fe8241.jpg)



1.常规实现（不推荐这种做，代码过多重复，会导致强耦合）：

---
特点：

\*\*\*\*\*\*\*\*\*\*   **a.侵入性强**

   **b.工作量大**



![image](images/422d84c6-5321-4e23-a1e4-7f43e8bea983.jpg)





---
2.AOP（Aspect Oriented Programming）：

---
特点：

\*\*\*\*\*\*\*\*\*\*\*\*   **a.面向切面编程**

   **b.针对同一类问题的统一处理**

\*\*    **c.无侵入添加代码**    \*\***d.修改方便**



**1.Join Points（程序运行时的执行点，可以作为切面的地方）\*\***    **a.函数调用、执行**    **b.获取、设置变量（get、set方法）**    \*\***c.类初始化**

**2.PointCut**   

    **带条件的Join Points**



**3.Advice(一种Hook，要插入代码的位置)**

    **a.Before：PointCut之前执行\*\***    **b.After：PointCut之后执行**    \*\***c.Around：PointCut之前、之后分别执行**

![image](images/0d3d010c-b9fb-45c3-bd05-bfce6b772b5b.jpg)

![image](images/2bc25a78-3569-4a56-b38d-5758b022c694.jpg)



\*\*

AspectJ使用（3-7）

    classpath 'com.hujiang.aspectjx:gradle-android-plugin-aspectjx:2.0.0'

    implementation 'org.aspect:aspectjrt:1.8.+'

    apply plugin:'android-aspectjx'

\*\*  

![image](images/e65d4a5f-37eb-49d0-9223-d7a45938d8a3.jpg)





\*\*

**四.异步优化**

1.优化小技巧，Theme切换（感观上的快）（3-8）：

    ①：drawable下创建一个layer.xml：

---
```java
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- 背景颜色 -->
    <item android:drawable="@android:color/white"/>

    <!-- 图片 如果尺寸不一致，那么周围的填充色就是背景色-->
    <item>
        <bitmap android:src="@drawable/start_page"
            android:gravity="fill"/>  <!-- clip_horizontal -->
    </item>
</layer-list>
```
        **②.①：在values-styles.xml:**

```java
<resources>
    <!--方法1： 图片不是全屏 -->
    <style name="AppTheme.Launcher" parent="Theme.AppCompat.Light.DarkActionBar">
        <item name="android:windowBackground">@drawable/layer</item>
        <item name="windowActionBar">false</item>
        <item name="android:windowNoTitle">true</item>
        <item name="windowNoTitle">true</item>
    </style>

    <!--方法2： 图片是全屏 -->
    <style name="AppTheme.Launcher" parent="@android:style/Theme.NoTitleBar.Fullscreen">
        <item name="android:windowBackground">@drawable/layer</item>
        <item name="windowActionBar">false</item>
        <item name="android:windowNoTitle">true</item>
        <item name="windowNoTitle">true</item>
    </style>

     <!--方法3： 图片是全屏 -->
    <style name="AppTheme.Launcher" parent="Theme.AppCompat.Light.NoActionBar">
        <item name="android:windowBackground">@drawable/layer</item>
        <item name="android:windowFullscreen">true</item>
    </style>
</resources>
```
       \*\*\*\*\*\*\*\***②.②：**

\*\*

\*\*

在清单文件的activity中配置：

\*\*

\*\*

---
        **③.①：在values-styles.xml:**

```java
<resources>
    <!-- 无ActionBar -->
    <style name="noBar" parent="Theme.AppCompat.Light.NoActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>
</resources>
```
        **③.②：在Acitivity的Oncreate中：**

```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        setTheme(R.style.noBar);//在super前,好像不写也没问题。
        super.onCreate(savedInstanceState);
    }
```
\*\*

\*\*



\*\*

\*\*

2.常规异步优化（不推荐）（CountDownLatch、ExecutorService）（3-8）：

![image](images/5fcf710a-12db-470f-b230-9ea2f1179ca6.jpg)

![image](images/5072e25a-deaf-4bc0-ac2e-f23f4a0ed5f5.jpg)

![image](images/4817cf72-81ba-4237-9ef3-d2bf522ebb93.jpg)

\*\*

\*\*  



\*\*

---
\*\*

\*\*

\*\*

\*\*

使用代码（CountDownLatch、ExecutorService）

\*\*

\*\*

\*\*

\*\*

---
\*\*

```java
    private CountDownLatch mCountDownLatch = new CountDownLatch(1);//传入1表示CountDownLatch必须被满足1次,即调用countDown()方法一次。

    //系统AsyncTask中设置线程池数量
    private static final int CPU_COUNT = Runtime.getRuntime().availableProcessors();  //获得设备CPU数量
    // We want at least 2 threads and at most 4 threads in the core pool,
    // preferring to have 1 less than the CPU count to avoid saturating
    // the CPU with background work
    private static final int CORE_POOL_SIZE = Math.max(2, Math.min(CPU_COUNT - 1, 4)); 

    @Override
    public void onCreate() {
        super.onCreate();

        ExecutorService executorService = Executors.newFixedThreadPool(CORE_POOL_SIZE);
        executorService.submit(new Runnable() {
            @Override
            public void run() {
                //初始化腾讯Bug上传
                initTetentBugly();  

                mCountDownLatch.countDown();//CountDownLatch满足一次

            }
        });

        try {
            mCountDownLatch.await();//如果CountDownLatch没有被满足，就一直等待。
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
```
---
3.异步优化最优解（启动器）（3-9、3-10）：

核心思想：充分利用CPU多核，自动梳理任务顺序。

![image](images/9c3a4dc4-f23a-4963-a3c6-755ac0e96414.jpg)



![image](images/fc054e24-859c-44b1-81fb-9c8eee16b06d.jpg)





---
![image](images/ef01e076-7725-4589-8edd-796c01ddf519.jpg)



**使用代码：**  

    **①：主线程执行的Task**

```java
/*
*  主线程执行的task
*/
public class InitUmengTask extends MainTask {
    @Override
    public void run() {
        UMConfigure.setLogEnabled(true);
        MobclickAgent.setCatchUncaughtExceptions(false);
        UMConfigure.init(mContext, "5b518009a40fa327a300004d", "Umeng",
                UMConfigure.DEVICE_TYPE_PHONE, "");
        //设置普通Dplus使用场景：
        MobclickAgent.setScenarioType(mContext, MobclickAgent.EScenarioType.E_DUM_NORMAL);
        //Umeng分享
        PlatformConfig.setWeixin("wx29c324f040ddc692", "015fb7cd7da1cb5de7f53a1f75a4932c");
        PlatformConfig.setQQZone("101437145", "f41155986a2f85d0cad9b426e7202ecf");
    }
}
```
    **②.①：异步执行的task**

```java
/*
 *  异步执行的task
 */
public class InitCrashHandlerTask extends Task {
    @Override
    public void run() {
        CrashHandler.getInstance().init(mContext);
    }
}
```
    **②.②：需要被等待的异步Task**

```java
 /*
 *  异步执行的task
 */
public class InitXingeTask extends Task {

    /*
    * 重写等待方法，使这个异步task执行完之后再执行其他task
    */
    @Override
    public boolean needWait() {
        return true;
    }

    @Override
    public void run() {
        XGPushConfig.enableDebug(mContext, true);
        XGPushManager.registerPush(mContext, new XGIOperateCallback() {
            @Override
            public void onSuccess(Object data, int flag) {
                //token在设备卸载重装的时候有可能会变
                Log.e("TPush", "注册成功，设备token为：" + data);
            }

            @Override
            public void onFail(Object data, int errCode, String msg) {
                Log.e("TPush", "注册失败，错误码：" + errCode + ",错误信息：" + msg);
            }
        });
    }
}
```
      **②.**

---
\*\*

\*\*

③

\*\*

\*\*

---
**：有依赖关系的异步Task**

```java
/*
 *  异步执行的task
 */
public class InitBaiduLocationTask extends Task{

    /*
    *  重写依赖关系方法，添加依赖的类，依赖的类执行完毕之后再执行这个类。
    */
    @Override
    public List<Class<? extends Task>> dependsOn() {
        List<Class<? extends Task>> task = new ArrayList<>();
        task.add(InitCrashHandlerTask.class);
        return task;
    }

    @Override
    public void run() {
        /***
         * 初始化定位sdk，建议在Application中创建
         */
        LocationService locationService = new LocationService(mContext);
        //震动传感器
        Vibrator mVibrator = (Vibrator) mContext.getSystemService(Service.VIBRATOR_SERVICE);
        SDKInitializer.initialize(mContext);
    }
}
```
    **③：调用启动器:**

```java
TaskDispatcher.init(getApplicationContext());
        TaskDispatcher instance = TaskDispatcher.createInstance();
        instance.addTask(new InitXingeTask())
                .addTask(new InitCrashHandlerTask())
                .addTask(new InitBaiduLocationTask())
                .addTask(new InitUmengTask())
                .start();
        instance.await();//使needWait()生效;
```
---
\*\*

\*\*

**五.更优秀的延迟初始化方案**

\*\*

\*\*

1.常规方案（3-11）：

---
![image](images/68f1358e-53f5-4866-9e7b-5eb2111988b1.jpg)





![image](images/c68de609-54cd-46ff-882d-a97ac959fbaf.jpg)

---
2.更优方案（3-11）：

---
![image](images/0084fdbd-97a0-4944-b77c-6e0e6a529627.jpg)



![image](images/0b82ec2e-6a22-432d-8c76-27a256cf8367.jpg)

```java
import android.os.Looper;
import android.os.MessageQueue;


import com.fsh.lfmf.launchstarter.task.DispatchRunnable;
import com.fsh.lfmf.launchstarter.task.Task;

import java.util.LinkedList;
import java.util.Queue;

public class DelayInitDispatcher {

    private Queue<Task> mDelayTasks = new LinkedList<>();

    private MessageQueue.IdleHandler mIdleHandler = new MessageQueue.IdleHandler() {
        @Override
        public boolean queueIdle() {
            if(mDelayTasks.size()>0){
                Task task = mDelayTasks.poll();
                new DispatchRunnable(task).run();
            }
            return !mDelayTasks.isEmpty();
        }
    };

    public DelayInitDispatcher addTask(Task task){
        mDelayTasks.add(task);
        return this;
    }

    public void start(){
        Looper.myQueue().addIdleHandler(mIdleHandler);
    }

}
```
---
---
**五.优化总方针**



![image](images/54f51a4d-b8fe-4956-83ad-b328314039b5.jpg)

  

![image](images/85efaa12-0824-4a89-8299-fd5755cc3d0a.jpg)



![image](images/969f1ef0-f0b5-4c36-a4d7-fc3334d8fc61.jpg)

![image](images/634dc53c-3322-4b7b-a614-9d35321dffe2.jpg)



![image](images/e9cad9cc-8c91-4051-8ed1-5134931869f6.jpg)

  

![image](images/2dd4b4a8-13ae-4d77-8cdb-7191edc5d19e.jpg)









\*\*\*\*\*\*\*\*\*\*\*\*1.懒加载（3-12）：

用到的时候初始化，不用到的时候不初始化，例如：某个页面需要用到高德地图，当用户在这个页面用到高德地图的时候进行初始化，在其他地方用不到的时候不进行初始化。





---
---
2.提前加载SharedPreferences（3-12）：

![image](images/ec074996-8b6b-44a8-b2c9-90ce4df434e7.jpg)



---
3.自动阶段不启动子进程（3-12）：

---
![image](images/855bb6a1-c56a-40d0-ada8-8eb821414ba3.jpg)

---
4.提前类加载（3-12）：

---
![image](images/0e08339f-bbe6-49d2-8ae8-ab16d8505742.jpg)











---