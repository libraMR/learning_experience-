# Okhttp
```java
implementation 'com.squareup.okhttp3:okhttp:3.9.0'
implementation 'com.github.bumptech.glide:glide:3.7.0'
implementation 'io.reactivex:rxjava:1.1.3'
implementation 'io.reactivex:rxandroid:1.1.0'
implementation 'com.squareup.retrofit2:retrofit:2.0.2'
implementation 'com.squareup.retrofit2:converter-gson:2.0.2'
implementation 'com.squareup.retrofit2:converter-scalars:2.0.2'
implementation 'com.squareup.retrofit2:adapter-rxjava:2.0.2'
implementation 'com.squareup.leakcanary:leakcanary-android:1.5.4'
```
```java
package org.devio.hi.library.okhttp;

import android.os.Bundle;

import com.google.android.material.floatingactionbutton.FloatingActionButton;
import com.google.android.material.snackbar.Snackbar;

import androidx.appcompat.app.AppCompatActivity;
import androidx.appcompat.widget.Toolbar;

import android.view.View;

import android.view.Menu;
import android.view.MenuItem;

import java.io.IOException;
import java.util.Objects;
import java.util.concurrent.TimeUnit;

import okhttp3.Call;
import okhttp3.Callback;
import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.Response;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        /*
         * 1.创建OkHttpClient对象
         *  -> 通过OkHttpClient.Builder()来初始化:
         *      -> 分发器类Dispatcher():
         *           1.异步请求: 用来决定异步请求是直接处理还是进行缓存等待；
         *           2.同步请求: 对同步请求没有太多操作，只是把同步请求放到队列里去执行。
         *      -> 连接池类ConnectionPool():
         *           1.将Connection请求放到连接池中，当请求的url是相同的时候，可以选择对其复用
         *           2.用来管理网络连接是保持打开状态还是以后进行复用等策略的设置
         *
         *  -> 最后通过build()创建OkHttpClient对象，并将配置好的Builder()传进去，完成整个对象属性的初始化: 分发器、线程池、拦截器等
         */
        OkHttpClient client = new OkHttpClient.Builder().readTimeout(5, TimeUnit.SECONDS).build();

        /*
         * 2.创建Request对象
         *  -> 通过Request.Builder()来初始化:
         *      -> method请求方式默认为: GET
         *      -> 创建Headers.Builder()对象，保存header头部信息
         *
         *   -> 最后通过build()创建Request对象，将配置好的Builder()传进去，完成整个对象属性的初始化: url、method、headers、body、tag
         */
        Request request = new Request.Builder().url("http://www.baidu.com").get().build();

        /*
         * 3.通过client.newCall创建Call对象(Call是接口，通过具体实现类RealCall来操作)，用来连接client和request
         *  -> client.newCall方法里实际是调用了RealCall.newRealCall来实现的:
         *       -> 通过RealCall.newRealCall创建了RealCall对象
         *           -> 创建好的RealCall对象里持有了:
         *               -> 第一步初始化好的OkHttpClient
         *               -> 第二步初始化好的Request
         *               -> 赋值了一个重定向拦截器对象 RetryAndFollowUpInterceptor
         *
         *  Tips: Call其实是由RealCall.newRealCall来生成的，RealCall是接口Call的实现类，因为接口实现类其实就是接口本身，这个是正常的向上转型
         */
        Call call = client.newCall(request);

        try {
            /*
             * 同步请求
             *   1.会阻塞当前线程，发生请求后，就会进入阻塞状态，直到收到相应。
             *
             * call.execute()调用的其实是具体实现类RealCall的execute()方法
             * -----------------------------------------------------todo 同步异步开始都是这三步---------------------------------------------------------
             *  -> 通过同步代码块synchronized判断executed的值
             *       -> 因为同一个http请求只能执行一次，通过executed这个标志位来判断http请求是否执行过，如果没有执行过就把它设为true，执行过就抛出异常
             *  -> 通过captureCallStackTrace()这个方法捕捉http请求的一些异常的堆栈信息
             *  -> 开启eventListener.callStart(this)监听事件，每次call调用execute()或者enqueue()方法，就会开始这个监听
             *------------------------------------------------------------------------------------------------------------------------------------
             *  -> 通过client.dispatcher()来获得client的分发器类dispatcher
             *       -> 调用client.dispatcher()的executed(this)方法来把当前的RealCall添加到runningSyncCalls这个队列中
             *  -> 调用getResponseWithInterceptorChain()这个拦截器链方法获取Response，这个拦截器链方法中会依次执行拦截器来进行操作
             *  -> 最后在finally中会调用client.dispatcher().finished(this)来主动回收某些同步请求
             *       -> 其实是调用dispatcher中这个finished(runningSyncCalls, call, false)方法
             *           -> 该方法中的一些主要操作是被锁住的，为了线程安全，即:　synchronized (this){ if (!calls.remove(call)...promoteCalls()... }
             *           -> 通过if (!calls.remove(call)) throw new AssertionError("Call wasn't in-flight!")移除这个同步请求，如果不能移除就抛出异常
             *           -> todo 第三个参数传的false，所以promoteCalls()方法不执行，这个在异步的时候会传true来执行
             *           -> 通过runningCallsCount()方法来计算目前还在运行的请求，其实该方法就是返回来正在执行的同步、异步请求的数量总和
             *           -> 最后如果runningCallsCount == 0说明整个dispatcher分发器类中没有可运行的请求了，同时又满足idleCallback != null，就运行idleCallback.run()
             *              -> 即: if (runningCallsCount == 0 && idleCallback != null) {idleCallback.run();}
             */
            Response response = call.execute();
            System.out.println(Objects.requireNonNull(response.body()).toString());
        } catch (IOException e) {
            e.printStackTrace();
        }

        /*
         * 异步请求
         *   1.不会阻塞当前线程，会开启一个新的子线程(AsyncCall)完成网络请求操作。
         *   1.onResponse、onFailure两个回调方法都是在子线程执行的。
         *
         * call.enqueue()调用的其实是具体实现类RealCall的enqueue()方法
         * -----------------------------------------------------todo 同步异步开始都是这三步---------------------------------------------------------
         *  -> 通过同步代码块synchronized判断executed的值
         *       -> 因为同一个http请求只能执行一次，通过executed这个标志位来判断http请求是否执行过，如果没有执行过就把它设为true，执行过就抛出异常
         *  -> 通过captureCallStackTrace()这个方法捕捉http请求的一些异常的堆栈信息
         *  -> 开启eventListener.callStart(this)监听事件，每次call调用execute()或者enqueue()方法，就会开始这个监听
         *------------------------------------------------------------------------------------------------------------------------------------
         *  -> 通过client.dispatcher()来获得client的分发器类dispatcher，调用client.dispatcher().enqueue(new AsyncCall(responseCallback))
         *       -> 首先把传入的Callback转成Runnable对象
         *            -> 即: new AsyncCall(responseCallback)
         *                  -> AsyncCall继承了NamedRunnable并实现了父类的execute()抽象方法，父类NamedRunnable的run()方法里进行的操作实际上是execute()抽象方法
         *                      -> 该方法首先通过拦截器链获得Response
         *                          -> 即: Response response = getResponseWithInterceptorChain();
         *                      -> 判断重定向和重试拦截器: 如果取消了，就会调用responseCallback.onFailure(RealCall.this, new IOException("Canceled"))这个方法
         *                          -> responseCallback就是我们传入的Callback
         *                      -> 如果没取消，就会调用responseCallback.onResponse(RealCall.this, response)这个方法
         *                      -> 最后在finally中会调用client.dispatcher().finished(this)来主动回收某些同步请求
         *                          -> 该方法中的一些主要操作是被锁住的，为了线程安全，即:　synchronized (this){ if (!calls.remove(call)...promoteCalls()... }
         *                          -> 通过if (!calls.remove(call)) throw new AssertionError("Call wasn't in-flight!")把这个请求从正在执行的异步任务队列中移除，如果不能移除就抛出异常
         *                          -> todo 第三个参数传的true，所以promoteCalls()这个方法执行，这个方法就是调整整个异步请求任务队列的
         *                          -> 通过runningCallsCount()方法来计算目前还在运行的请求，其实该方法就是返回来正在执行的同步、异步请求的数量总和
         *                          -> 最后如果runningCallsCount == 0说明整个dispatcher分发器类中没有可运行的请求了，同时又满足idleCallback != null，就运行idleCallback.run()
         *                              -> 即: if (runningCallsCount == 0 && idleCallback != null) {idleCallback.run();}
         *       -> 然后调用client.dispatcher()的enqueue()方法将new AsyncCall(responseCallback)添加到 runningAsyncCalls(正在执行的异步任务队列) 或 readyAsyncCalls(等待队列)中
         *             ->即dispatcher()





































的: synchronized void enqueue(AsyncCall call)方法         *                  -> 该方法首先要判断正在执行的异步任务队列中任务数是否小于maxRequests，并且正在执行的任务的host小于maxRequestsPerHost         *                      -> 即: if (runningAsyncCalls.size() < maxRequests && runningCallsForHost(call) < maxRequestsPerHost){         *                      -> maxRequests(默认值: 64)、maxRequestsPerHost(默认值: 5)         *                  -> 满足上边两个条件，就把我们传进来的异步请求Runnable(AsyncCall)添加到正在执行的异步任务队列中         *                      -> 即: runningAsyncCalls.add(call);         *                  -> 然后通过线程池去执行这个异步请求         *                      -> 即: executorService().execute(call);}         *                          -> 线程池创建方法: public synchronized ExecutorService executorService(){         *                                                  if (executorService == null) {         *                                                      new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60, TimeUnit.SECONDS,...         *                                                  }         *                                            }         *                              -> synchronized保证是创建的线程池是单例的         *                              -> 0 代表核心线程数的数量，设定为0，如果空闲一段时间之后，就会将所有的线程全部销毁         *                              -> 线程池最大线程数为Integer.MAX_VALUE，但是不影响，因为我们每次执行的任务数是低于maxRequests(默认值: 64)         *                              -> 60 当我们的线程数大于核心线程数的时候，那么多余的空闲线程最多存活60秒         *                          -> 调用execute(call)来执行传入进来的异步请求Runnable(AsyncCall)的run方法         *                  -> 如果不满足以上两个条件，就把我们的异步请求Runnable(AsyncCall)添加到等待队列中         *                      -> 即: else { readyAsyncCalls.add(call); }         *         */        call.enqueue(new Callback() {//new Callback()就是用来请求结束后进行接口回调的            @Override            public void onFailure(Call call, IOException e) {            }            @Override            public void onResponse(Call call, Response response) throws IOException {                System.out.println(request.body().toString());            }        });        //dispatcher = new Dispatcher();    }}
```
