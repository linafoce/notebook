1. newFixedThreadPool(int nThreads) 

创建一个固定长度的线程池，每当提交一个任务就创建一个线程，直到达到线程池的最大数 

量，这时线程规模将不再变化，当线程发生未预期的错误而结束时，线程池会补充一个新的 

线程。 

2. newCachedThreadPool() 

创建一个可缓存的线程池，如果线程池的规模超过了处理需求，将自动回收空闲线程，而当 

需求增加时，则可以自动添加新线程，线程池的规模不存在任何限制 

3. newSingleThreadExecutor() 

这是一个单线程的 Executor，它创建单个工作线程来执行任务，如果这个线程异常结束，会 

创建一个新的来替代它；它的特点是能确保依照任务在队列中的顺序来串行执行 

4. newScheduledThreadPool(int corePoolSize) 

创建了一个固定长度的线程池，而且以延迟或定时的方式来执行任务，类似于 Timer。

```java
private static final Executor exec = Executors.newFixedThreadPool(50);
	Runnable runnable = new Runnable() {
		public void run() { ...}
	}exec.execute(runnable);
	Callable<Object> callable = new Callable<Object>() {
		public Object call() throws Exception {
			return null;
		}
	};
	Future future = executorService.submit(callable);
	future.get(); // 等待计算完成后，获取结果 
	future.isDone(); // 如果任务已完成，则返回 true 
	future.isCancelled(); // 如果在任务正常完成前将其取消，则返回 true 
	future.cancel(true); // 试图取消对此任务的执行，true 中断运行的任务，false 允许正在运行 的任务运行完成
```

