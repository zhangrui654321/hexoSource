

## 线程池总结

**`ThreadPoolExecutor` 3 个最重要的参数：**

- **`corePoolSize` :** 核心线程数线程数定义了最小可以同时运行的线程数量。

- **`maximumPoolSize` :** 当队列中存放的任务达到队列容量的时候，当前可以同时运行的线程数量变为最大线程数。

- **`workQueue`:** 当新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中。

  ##### 线程池的执行策略

  ![](E:\javaProject\hexo\source\images\220707-2.png)

  ![](/images/220707-2.png)

  ##### 线程池和callable一起使用，一般用来有返回值得类型

  第一 创建线程池

  ```
  private static ThreadPoolExecutor executorService = new ThreadPoolExecutor(9, 9, 1000, TimeUnit.MILLISECONDS, WORK_QUEUE, HANDLER);
  ```

​		第二 重写callable方法

​		Callable<List> bannerListCallable = () -> adService.queryIndex();

​		第三步 调用线程池submit方法，获取future对象

​		Future<List> banner=executorService.submit(bannerListCallable);

​		第四步，调用future.get方法获取返回值

​		Map<String, Object> entity = new HashMap<>();

​		entity.put("banner", banner.get());

