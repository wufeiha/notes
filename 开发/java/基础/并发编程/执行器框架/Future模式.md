### 关键对象

- FutureTask

### 例子

#### 并行计算

```java
package futuretask;
 
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.FutureTask;
 
public class FutureTaskForMultiCompute {
    
    public static void main(String[] args) {
        
        FutureTaskForMultiCompute inst=new FutureTaskForMultiCompute();
        // 创建任务集合
        List<FutureTask<Integer>> taskList = new ArrayList<FutureTask<Integer>>();
        // 创建线程池
        ExecutorService exec = Executors.newFixedThreadPool(5);
        for (int i = 0; i < 10; i++) {
            // 传入Callable对象创建FutureTask对象
            FutureTask<Integer> ft = new FutureTask<Integer>(inst.new ComputeTask(i, ""+i));
            taskList.add(ft);
            // 提交给线程池执行任务，也可以通过exec.invokeAll(taskList)一次性提交所有任务;
            exec.submit(ft);
        }
        
        System.out.println("所有计算任务提交完毕, 主线程接着干其他事情！");
 
        // 开始统计各计算线程计算结果
        Integer totalResult = 0;
        for (FutureTask<Integer> ft : taskList) {
            try {
                //FutureTask的get方法会自动阻塞,直到获取计算结果为止
                totalResult = totalResult + ft.get();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
        }
 
        // 关闭线程池
        exec.shutdown();
        System.out.println("多任务计算后的总结果是:" + totalResult);
 
    }
 
    private class ComputeTask implements Callable<Integer> {
 
        private Integer result = 0;
        private String taskName = "";
        
        public ComputeTask(Integer iniResult, String taskName){
            result = iniResult;
            this.taskName = taskName;
            System.out.println("生成子线程计算任务: "+taskName);
        }
        
        public String getTaskName(){
            return this.taskName;
        }
        
        @Override
        public Integer call() throws Exception {
            // TODO Auto-generated method stub
 
            for (int i = 0; i < 100; i++) {
                result =+ i;
            }
            // 休眠5秒钟，观察主线程行为，预期的结果是主线程会继续执行，到要取得FutureTask的结果是等待直至完成。
            Thread.sleep(5000);
            System.out.println("子线程计算任务: "+taskName+" 执行完成!");
            return result;
        }
    }
}
```

#### 高并发下确保只执行一次

获取线程池链接

```java
package com.along.executor;

import java.sql.Connection;
import java.util.concurrent.Callable;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.FutureTask;

/**
 * @author wxp
 * @Title:
 * @Package
 * @Description:
 * @date 2021/6/28 下午2:18
 */
public class TestFuture {
  private ConcurrentHashMap<String, FutureTask<Connection>> connectionPool = new ConcurrentHashMap<String, FutureTask<Connection>>();

  public Connection getConnection(String key) throws Exception {
    FutureTask<Connection> connectionTask = connectionPool.get(key);
    if (connectionTask != null) {
      return connectionTask.get();
    } else {
      Callable<Connection> callable = new Callable<Connection>() {
        @Override
        public Connection call() throws Exception {
          // TODO Auto-generated method stub
          //注意这块只有run的时候才会执行，所以只会创建一次
          return createConnection();
        }
      };
      FutureTask<Connection> newTask = new FutureTask<Connection>(callable);
      //链接虽然只创建了一次，但是FutureTask，Callable创建了多次，不知道会不会有内存占用问题
      connectionTask = connectionPool.putIfAbsent(key, newTask);
      if (connectionTask == null) {
        connectionTask = newTask;
        //执行创建链接方法
        connectionTask.run();
      }
      return connectionTask.get();
    }
  }

  //创建Connection
  private Connection createConnection() {
    return null;
  }

}
```

