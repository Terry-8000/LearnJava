# Fork/Join框架

## 1. Fork/Join框架简介

Fork/Join框架是java7提供的一个用于并行执行任务的框架，是一个把大任务分割成若干个小任务，最终汇总每个小任务结果得到大任务结果的框架。Fork指的就是把一个大任务分割成若干子任务并行的执行，Join就是合并这些子任务的执行结果，最后得到这个大任务的结果。

比如计算1+2+3+4+5+……+10000，可以分割成10个子任务，每个子任务负责分别对1000个数进行求和，最终汇总这10个子任务的结果。

## 2. Fork/Join框架使用说明

使用Fork/Join框架我们需要两个类：

- ForkJoinTask：Fork/Join任务，提供fork()和Join()方法，通常情况下继承它的两个子类：
  - RecursiveAction：返回没有结果的任务。
  - RecursiveTask：返回有结果的任务。
- ForkJoinPool：ForkJoinTask需要通过ForkJoinPool来执行。

从ForkJoinTask的两个子类的名字中就可以看到，这是一种采用递归方式实现的任务分割，任务分割的子任务会添加到当前工作线程所维护的双端队列中，进入队列的头部，当一个工作线程的队列里面暂时没有任务时，它会随机从其他工作线程队列的尾部获取一个任务去执行。

## 3. Fork/Join框架实例

#### 3.1 需求：

计算1 + 2 + 3 +……+10 的结果。

#### 3.2 需求分析设计：

使用Fork/Join框架首先要考虑的就是如何分割任务，和分割任务的粒度，这里我们考虑每个子任务最多执行两个数相加，那我们分割的阈值就是2，Fork/Join框架会把这个任务fork成5个子任务，然后再join5个子任务的结果。因为是有结果的任务，所以必须继承RecursiveTask。

#### 3.3 代码实例

```java
package com.wangjun.thread;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.Future;
import java.util.concurrent.RecursiveTask;

public class ForkJoinTaskTest extends RecursiveTask<Integer>{
	
	private static final int THRESHOLD = 2;  // 阈值
	private int start;
	private int end;
	
	public ForkJoinTaskTest(int start, int end) {
		this.start = start;
		this.end = end;
	}
	
	@Override
	protected Integer compute() {
		int sum = 0;
		 if(end - start < THRESHOLD) {
			 System.out.println(Thread.currentThread().getName() + ":" + start + "-" + end);
			 for(int i = start; i <= end; i++) {
				 sum += i;
			 }
		 }else {
			 // 任务大于阈值，拆分子任务
			 int middle = start + (end - start)/2;
			 ForkJoinTaskTest task1 = new ForkJoinTaskTest(start, middle);
			 ForkJoinTaskTest task2 = new ForkJoinTaskTest(middle + 1, end);
			 // 执行子任务
			 task1.fork();
			 task2.fork();
			 // 等待子任务执行完成，并得到其结果
			 int result1 = task1.join();
			 int result2 = task2.join();
			 // 合并子任务
			 sum = result1 + result2;
		 }
		return sum;
	}
	
	public static void main(String[] args) throws InterruptedException, ExecutionException {
		int start = 1;
		int end = 10;
		ForkJoinTaskTest task = new ForkJoinTaskTest(start, end);
		// 用来执行ForkJoinTask
		ForkJoinPool pool = new ForkJoinPool();
		// 执行任务
		Future<Integer> result = pool.submit(task);
		System.out.println("最后的结果为：" + result.get());
	}

}
```

**运行结果：**

```
ForkJoinPool-1-worker-2:3-3
ForkJoinPool-1-worker-0:4-5
ForkJoinPool-1-worker-3:6-7
ForkJoinPool-1-worker-1:1-2
ForkJoinPool-1-worker-3:8-8
ForkJoinPool-1-worker-0:9-10
最后的结果为：55
```

可以看到ForkJoinTask与一般的任务在于它需要实现`compute()`方法，在这个方法里面首先需要判断是否需要分割子任务，如果需要分割，每个子任务再调用fork方法时，就会进入compute()方法，如果不需要分割，则执行当前任务并返回结果，使用join方法会等待子任务结果的返回。可以看到这是一种递归的实现。



