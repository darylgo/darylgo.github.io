---
title: 深入理解 AsyncTask
date: 2017-04-10 10:57:00
categories:
- Android
tags:
---

AsyncTask 是一个简单实用的多线程异步任务工具类。

Android 开发中经常遇到需要将耗时的操作放到子线程中进行异步执行，等执行完毕之后再通知主线程更新 UI 的情况，例如异步加载网络图片、异步读取文件等，如果在主线程中执行这些耗时操作，就会造成卡顿的情况，甚至是 ANR。开启一个线程处理耗时任务很简单，创建一个 Thread 对象，在 run() 方法中去执行耗时操作即可，但是当耗时任务执行完毕要通知主线程更新 UI 的时候就比较麻烦了，我们需要创建一个主线程的 Handler，然后通过它向主线程发送更新 UI 的消息进行 UI 更新，这一整个过程十分麻烦。

基于上诉问题， AsyncTask 就出现了，按照官方的说法，AsyncTask 是为了解决主线程和子线程之间的交互问题，它能够在后台线程中执行耗时操作，并在把执行结果发送到主线程，而这一系列的操作都不需要你创建任何的 Thread 或 Handler 对象。AsyncTask 适合处理耗时不是很长的任务（几秒钟），所以你不应该拿它来处理耗时很长的任务，至于为什么，我们会在后面解释原因。

# 1 用法

假设我们有一个加载本地文件的任务，需要从网络上加载 3 个文件，并且在加载每个文件的时候通知用户当前正在加载什么文件，整个过程大概需要 10 秒钟，这很明显是一个十分耗时的任务，我们就用 AsyncTask 进行一步加载。

## 1.1 继承 AsyncTask

```java
public abstract class AsyncTask<Params, Progress, Result> { ... }
```

AsyncTask 本身是一个抽象类，它要求我们必须继承它并指定三个泛型参数类型，这三个泛型参数类型依次是执行参数类型（Params）、进度值类型（Progress）和结果类型（Result）：

* **执行参数类型（Params）**

  指的是在执行异步任务的时候指定的额外参数类型，例如加载文件的 URL。

* **进度值类型（Progress）**

  指的是在执行异步任务期间要更新进度情况时使用的数据类型，例如用 Integer 类型更新 ProgressBar。

* **结果类型（Result）**

  指的是异步任务执行结束后返回的结果类型，例如返回 一个 Boolean 类型的数据代表异步任务是否成功。

所以接下来我们要做的第一步就是继承 AsyncTask 并指定参数类型：

```java
/**
 * 加载文件的异步任务。
 */  
public class LoadFileTask extends AsyncTask<String, String, Boolean> {

}
```

在上面的代码里，我们指定执行参数类型为 String 类型，因为我们在加载文件的时候需要指定三个 URL 作为参数；进度值类型为 String 类型，因为每加载一个文件的时候都要通知用户当前加载的文件名；结果类型为 Boolean，当三个任务都加载成功的时候我们返回 true 代表文件加载成功。

指定完参数类型之后，我们还要实现几个方法，并且在这些方法里面写入我们的异步操作逻辑：

* **void onPreExecute()**

  该方法会主线程中执行，用于在执行异步任务之前的业务逻辑，例如弹出一个 ProgressDialog。

* **Result doInBackground(Params... params)**

  从方法名称就可以知道它是在子线程中执行的，我们可以在这个方法里面执行耗时操作，例如加载文件，该方法的参数类型就是前面提到的执行参数类型，并且它是一个可变参数，也就是说我们可以一次指定多个执行参数。另外，该方法要求你返回一个你指定好的结果类型数据，该数据会在异步任务结束的时候作为 onPostExecute(Result result) 的参数。

  > 注意该方法是一个必须实现的抽象方法。

* **void onPostExecute(Result result)**

  该方法会在主线程中执行，用于在执行异步任务完成之后的业务逻辑，例如关闭 ProgressDialog，该方法的参数类型就是前面提到的结果类型。

* **void onProgressUpdate(Progress... values)**

  该方法会在主线程中执行，用于在执行异步任务期间更新进度情况，例如在加载文件的时候不断刷新进度条，你需要通过调用 publishProgress(Progress... values) 触发该回调，从而更新进度情况。

接下来，我们就要实现上述几个方法，在开始加载文件之前以 Toast 形式提示用户“开始加载文件”，在加载某一个文件的时候提示用户“正在加载文件：xxx”，在加载完所有文件的时候根据加载文件的结果提示用户文件是否都加载成功：

```java
/**
 * 加载文件的异步任务。
 */  
public class LoadFilesTask extends AsyncTask<String, String, Boolean> {

    private Context mContext;
    
    public LoadFileTask(Context context) {
        mContext = context;
    }

    @Override
    public void onPreExecute() {
        // 在异步任务开始前提示用户。
        Toast.makeText(mContext, "开始加载文件", Toast.LENGTH_SHORT).show();
    }
    
    @Override
    public Boolean doInBackground(String... params) {
        String file1 = params[0];
        String file2 = params[1];
        String file3 = params[2];
        
        boolean isSuccess = true;
        
        // 加载第一个文件，并且通知用户当前加载的文件名称。
        publishProgress("File1");
        isSuccess = isSuccess && loadFile(file1);
        
        // 加载第二个文件，并且通知用户当前加载的文件名称。
        publishProgress("File2");
        isSuccess = isSuccess && loadFile(file2);
        
        // 加载第三个文件，并且通知用户当前加载的文件名称。
        publishProgress("File3");
        isSuccess = isSuccess && loadFile(file3);
        
        // 返回文件加载结果。
        return isSuccess;
    }
    
    @Override
    public void onProgressUpdate(String... values) {
        // 提示用户当前在加载的是哪个文件。
        String fileName = values[0];
        String message = "正在加载文件：" + fileName;
        Toast.makeText(mContext, message, Toast.LENGTH_SHORT).show();
    }
    
    @Override
    public void onPostExecute(Boolean result) {
        // 在异步任务结束的时候根据不同的加载结果提示用户。
        boolean isSuccess = result;
        if (isSuccess) {
            Toast.makeText(mContext, "成功加载所有文件", Toast.LENGTH_SHORT).show();
        } else {
            Toast.makeText(mContext, "加载文件失败", Toast.LENGTH_SHORT).show();      
        }
    }
    
}
```

## 1.2 执行异步任务

继承 AsyncTask 创建异步加载文件的任务之后，接下来要做的就是执行这个异步任务了，AsyncTask 提供了两个执行异步任务的方法：

* **execute(Params...)**

  使用 AsyncTask 提供的线程池执行异步任务，并且指定若干个执行参数。

  > 关于 AsyncTask 的线程池我们会在后面详细说明。

* **executeOnExecutor(Executor, Params...)**

  使用自己定义的线程池执行异步任务，并且指定若干个执行参数。

在这里我们就直接使用 AsyncTask 提供的线程池执行我们的文件加载任务，并且指定三个要加载的文件的路径作为执行参数：

```java
String file1 = "http://www.example.com/file1";
String file2 = "http://www.example.com/file2";
String file3 = "http://www.example.com/file3";
LoadFilesTask loadFilesTask = new LoadFilesTask(mContext);// 创建异步任务实例
loadFilesTask.execute(file1, file2, file3);// 执行异步任务
```

到此为止，我们的异步加载文件任务就开发完成了，可以说 AsyncTask 的使用方法还是很简单的。

# 2 版本变化

AsyncTask 从引入到现在，经历了几次较大的改动，我们有必要了解这几次的改动以免在实际开发的时候踩坑：

* **Android 1.5**

  AsyncTask 作为一个异步任务工具类首次被引入 SDK，此时的 AsyncTask 只能按照串行的方式，一个接一个的执行每一个异步任务，也就是说无论你通过 AsyncTask 创建多少个异步任务，它们都会被加入队列中按顺序依次执行，如果某一个异步任务耗时很久，就会导致后面的任务一直无法被执行。

* **Android 1.6 至 3.0（不包括3.0）**

  AsyncTask 开始支持多线程并发的情况，其内部有一个全局共享的线程池，该线程池的最小线程数为 5，最大线程数为 128。也就是说现在你可以创建多个 AsyncTask 实例，并且让它们同时被执行了。

* **Android 3.0 及以上**

  AsyncTask 又变成了默认情况下单线程串行方式依次执行每一个异步任务，Google 给出的理由是避免某些并发问题，具体问题估计要跟业务场景有关。如果你希望并行执行多个异步任务，可以通过 executeOnExecutor(Executor, Params...) 方法执行多线程的线程池。

# 3 源码分析

现在，是时候来看看 AsyncTask 的源码实现了，让我们从 execute(Params…) 方法开始吧，看看里面都干了啥：

```java
@MainThread
public final AsyncTask<Params, Progress, Result> execute(Params... params) {
  return executeOnExecutor(sDefaultExecutor, params);
}
```

可以看到 execute(Params…) 方法里面就一句话，调用 executeOnExecutor(Executor, Params…) 方法执行异步任务，并且指定了一个默认的线程池叫做 sDefaultExecutor，我们来看看这个线程池是什么样子的：

```java
private static volatile Executor sDefaultExecutor = SERIAL_EXECUTOR;
public static final Executor SERIAL_EXECUTOR = new SerialExecutor();

// 顺序执行异步任务的线程池。
private static class SerialExecutor implements Executor {
    final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
    Runnable mActive;

    public synchronized void execute(final Runnable r) {
        mTasks.offer(new Runnable() {
            public void run() {
                try {
                    r.run();
                } finally {
                    scheduleNext();
                }
            }
        });
        if (mActive == null) {
            scheduleNext();
        }
    }

    protected synchronized void scheduleNext() {
        if ((mActive = mTasks.poll()) != null) {
          	// 最终使用的是另一个线程池来执行异步任务，这个后面会解释。
            THREAD_POOL_EXECUTOR.execute(mActive);
        }
    }
}
```

原来 sDefaultExecutor 指向的是 SerialExecutor，其内部使用了 ArrayDeque 按顺序存储要执行的异步任务，这些异步任务是按顺序依次被执行的，当前一个任务执行完毕之后，就从队列中取出下一个任务继续执行。也许你已经看到了 THREAD_POOL_EXECUTOR 这个线程池，发现 SerialExecutor 实际上只是一个异步任务的调度中心，最终执行任务的线程池是 THREAD_POOL_EXECUTOR，我们顺藤摸瓜继续往下看：

```java
private static final int CPU_COUNT = Runtime.getRuntime().availableProcessors();
private static final int CORE_POOL_SIZE = CPU_COUNT + 1;
private static final int MAXIMUM_POOL_SIZE = CPU_COUNT * 2 + 1;
private static final int KEEP_ALIVE = 1;
private static final BlockingQueue<Runnable> sPoolWorkQueue = new LinkedBlockingQueue<Runnable>(128);

// 创建 AsyncTask 线程时名称类似：AsyncTask #1
private static final ThreadFactory sThreadFactory = new ThreadFactory() {
    private final AtomicInteger mCount = new AtomicInteger(1);
    public Thread newThread(Runnable r) {
        return new Thread(r, "AsyncTask #" + mCount.getAndIncrement());
    }
};

/**
 * An {@link Executor} that can be used to execute tasks in parallel.
 */
public static final Executor THREAD_POOL_EXECUTOR = new ThreadPoolExecutor(
        CORE_POOL_SIZE,
        MAXIMUM_POOL_SIZE,
        KEEP_ALIVE,
        TimeUnit.SECONDS,
        sPoolWorkQueue,
        sThreadFactory);
```

从代码中可以看出 AsyncTask 内部使用的线程池就是 THREAD_POOL_EXECUTOR，我们要关注的是 CORE_POOL_SIZE、MAXIMUM_POOL_SIZE 和 sPoolWorkQueue 三个常量的定义：

* CORE_POOL_SIZE：AsyncTask 默认的线程池最小线程数是 CPU 数加 1
* MAXIMUM_POOL_SIZE：最大线程数是 CPU 数的两倍再加 1
* sPoolWorkQueue：线程池队列中最多允许 128 个异步任务

所以，我们在使用 AsyncTask 的时候需要注意了，当我们直接使用 execute(Params…) 方法按顺序执行异步任务的时候，所有的任务都是按顺序依次执行的，如果某一个任务过于耗时，会导致后面任务都处于长时间等待状态；当我们使用 executeOnExecutor(Executor, Params…) 方法执行异步任务的时候，如果直接使用 AsyncTask 的 THREAD_POOL_EXECUTOR 作为并行执行异步任务的线程池时，它最多支持 MAXIMUM_POOL_SIZE 个异步任务并行执行，并且在极端情况下你最多添加 128 个异步任务，所以我们的建议是自己创建并行执行异步任务的线程池。

讨论完 AsyncTask 的线程之后，我们来看看它的执行流程，我们从 executeOnExecutor(Executor, Params…) 方法看起：

```java
@MainThread
public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
        Params... params) {
    if (mStatus != Status.PENDING) {
        switch (mStatus) {
            case RUNNING:
                throw new IllegalStateException("Cannot execute task:"
                        + " the task is already running.");
            case FINISHED:
                throw new IllegalStateException("Cannot execute task:"
                        + " the task has already been executed "
                        + "(a task can be executed only once)");
        }
    }

    mStatus = Status.RUNNING;

    onPreExecute();

    mWorker.mParams = params;
    exec.execute(mFuture);

    return this;
}
```

首先我们注意到如果你重复调用 AsyncTask 的 executeOnExecutor(Executor, Params…) 方法的话，系统会抛出异常告诉你这个 AsyncTask 已经被执行或者已经结束。当我们调用 executeOnExecutor(Executor, Params…) 方法的时候 onPreExecute() 回调就会被触发，之后就是调用线程池安排执行我们的异步任务。你应该注意到了 mWorker 和 mFuture 两个成员变量了吧？我们先来看下这两个是什么东西：

```java
private final WorkerRunnable<Params, Result> mWorker;
private final FutureTask<Result> mFuture;

private static abstract class WorkerRunnable<Params, Result> implements Callable<Result> {
    Params[] mParams;
}
```

很简单，mWorker 实际上就是 WorkerRunnable， 它实现了 Callable 接口，并且定义了和 AsyncTask 对应的 Params 和 Result 泛型，其内部还存储了用于执行异步任务的参数，而 mFuture 就是 FutureTask 了，用于监听异步任务执行结果，这两个成员变量应该说是 AsyncTask 的核心了，我们继续看下一段代码：

```java
public AsyncTask() {
    mWorker = new WorkerRunnable<Params, Result>() {
        public Result call() throws Exception {
            mTaskInvoked.set(true);
			
          	// 此处有坑，此处有坑，此处真的有坑，重要的事说三遍！
            Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
            //noinspection unchecked
            Result result = doInBackground(mParams);
            Binder.flushPendingCommands();
            return postResult(result);
        }
    };

    mFuture = new FutureTask<Result>(mWorker) {
        @Override
        protected void done() {
            try {
                postResultIfNotInvoked(get());
            } catch (InterruptedException e) {
                android.util.Log.w(LOG_TAG, e);
            } catch (ExecutionException e) {
                throw new RuntimeException("An error occurred while executing doInBackground()",
                        e.getCause());
            } catch (CancellationException e) {
                postResultIfNotInvoked(null);
            }
        }
    };
}
```

很明显 mWorker 负责在其他线程中执行异步逻辑，也就是 doInBackground(Params…) 方法，并且在异步逻辑完成之后调用 postResult(Result) 方法执行 onPostExecute(Result)，mFuture 在 done() 回调方法中处理一些特殊的情况，例如异步任务还没有被执行就被取消。有一个地方需要特别注意的是执行异步逻辑的时候，调用了 Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND) 方法设置当前的线程优先级为后台级别，这是一个深坑，在某些低端的单核手机上会出现主线程由于优先级较高而一直占用 CPU 资源导致 AsyncTask 的异步任务一直无法被执行的情况。

最后，我们看看 AsyncTask 是如何处理 onPostExecute(Reuslt) 和 onProgressUpdate(Progress…) 的，源码如下：

```java
private Result postResult(Result result) {
    @SuppressWarnings("unchecked")
    Message message = getHandler().obtainMessage(MESSAGE_POST_RESULT,
            new AsyncTaskResult<Result>(this, result));
    message.sendToTarget();
    return result;
}

@WorkerThread
protected final void publishProgress(Progress... values) {
    if (!isCancelled()) {
        getHandler().obtainMessage(MESSAGE_POST_PROGRESS,
                new AsyncTaskResult<Progress>(this, values)).sendToTarget();
    }
}

private void finish(Result result) {
    if (isCancelled()) {
        onCancelled(result);
    } else {
        onPostExecute(result);
    }
    mStatus = Status.FINISHED;
}

private static class InternalHandler extends Handler {
    public InternalHandler() {
        super(Looper.getMainLooper());
    }

    @SuppressWarnings({"unchecked", "RawUseOfParameterizedType"})
    @Override
    public void handleMessage(Message msg) {
        AsyncTaskResult<?> result = (AsyncTaskResult<?>) msg.obj;
        switch (msg.what) {
            case MESSAGE_POST_RESULT:
                // There is only one result
                result.mTask.finish(result.mData[0]);
                break;
            case MESSAGE_POST_PROGRESS:
                result.mTask.onProgressUpdate(result.mData);
                break;
        }
    }
}
```

其实原理很简单就是利用 Handler 将数据发送到主线程执行。

# 4 注意事项

最后我们总结下使用 AsyncTask 需要注意的事项：

* Android 1.6 至 3.0（不包括 3.0）的AsyncTask 是支持并行执行异步任务的，其他版本都是按顺序依次执行异步任务。
* 如果你希望并行执行异步任务，建议你创建自己的线程池，通过 executeOnExecutor(Executor, Params…) 方法执行异步任务，尽量不用使用 AsyncTask 提供的默认线程池。
* AsyncTask 的异步线程优先级是 Process.THREAD_PRIORITY_BACKGROUND，在某些低端的单核手机上会出现主线程由于优先级较高而一直占用 CPU 资源导致 AsyncTask 的异步任务一直无法被执行的情况。

AsyncTask 虽然好用，但是请以正确的姿势使用，避免出现预想不到的问题。