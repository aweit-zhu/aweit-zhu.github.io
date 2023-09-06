### Java 8 CompletableFuture

參考資料：<https://medium.com/javarevisited/java-completablefuture-c47ca8c885af>

#### 建構方法

|  方法   |
|  ----  |
| runAsync(Runnable runnable) |
| runAsync(Runnable runnable, Executor executor)  |
| supplyAsync(Supplier supplier)|
| supplyAsync(Supplier supplier, Executor executor)|

`runAsync()`: 執行非同步工作，沒有回傳值。
`supplyAsync()` 執行非同步工作，有一回傳值。
`executor`: 自定義Thread Pool。

```
// Example using runAsync()
CompletableFuture<Void> future1 = CompletableFuture.runAsync(() -> {
    // Perform some asynchronous task
    System.out.println("Task executed asynchronously");
});

// Example using supplyAsync()
CompletableFuture<Integer> future2 = CompletableFuture.supplyAsync(() -> {
    // Perform some asynchronous computation
    return 42;
});
```

#### Blocking (阻斷) and Non-blocking (非阻塞)

阻斷

```
CompletableFuture<Integer> future1 = CompletableFuture.supplyAsync(() -> "hello!")
        .thenApplyAsync(s -> s.length());
try {
    Integer length = future1.get(); // 會等到 Fucure 物件處理完畢了，並且取到結果值了，才會往下執行。故，主執行緒被阻斷在這裡了。
    System.out.println("length:" + length);
} catch (InterruptedException | ExecutionException e) {
    // TODO Auto-generated catch block
    e.printStackTrace();
} 
System.out.println("end");
```

非阻斷

```
CompletableFuture future2 = CompletableFuture.supplyAsync(() -> "hello!")
        .thenApplyAsync(s -> s.length())
        .thenAccept(length -> System.out.println("length: " + length));

// Non Blocking Code 

future2.join(); // Wait for the future to complete

// 等到 future2 執行完畢後，才會繼續執行以下。
System.out.println("end");
```

#### Exception Handling

```
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
    int result = 10 / 0; // Causes an ArithmeticException
    return result;
});

future.exceptionally(ex -> {
    System.out.println("Exception occurred: " + ex.getMessage());
    return 0; // Default value to return if there's an exception
}).thenAccept(result -> {
    System.out.println("Result: " + result);
});

---
Exception occurred: java.lang.ArithmeticException: / by zero
Result: 0
```

#### Composible

```
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> "Hello");

CompletableFuture<Integer> transformedFuture = future.thenApplyAsync(s -> {
    System.out.println("Thread: " + Thread.currentThread().getName());
    return s.length();
});

transformedFuture.thenAccept(length -> {
    System.out.println("Thread: " + Thread.currentThread().getName());
    System.out.println("Length of Hello: " + length);
});
```

- xxxx(function): function會用前個執行的thread去呼叫。
- xxxxAsync(function): function會用非同步的方式呼叫，並用預設的thread pool。
- xxxxAsync(function, executor): function會用非同步的方式呼叫，並用指定的thread pool。

|方法|描述|
|--|--|
|thenRun()|同一執行緒，執行回傳 void 的任務|
|thenRunAsync()|不同執行緒，執行回傳 void 的任務|
|thenApply()|同一執行緒，執行回傳物件的任務 |
|thenApplyAsync()|不同執行緒，執行回傳物件的任務 |

```
CompletableFuture.supplyAsync(() -> "hello").thenApplyAsync(s -> s.toUpperCase())
        .thenApplyAsync(s -> s + ", world!").thenApplyAsync(s -> s.toUpperCase())
        .whenComplete((r, ex) -> System.out.println(r)).join();

CompletableFuture.runAsync(() -> sleep(1000)).thenRunAsync(() -> sleep(100)).thenRunAsync(() -> sleep(100))
        .whenComplete((r, ex) -> System.out.println(r)).join();
```

Compose 兩種比較：thenApplyAsync()、thenCompose()

```
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> 5).thenApplyAsync(r -> r * 2);
future.thenAccept(s -> System.out.println("Final result: " + s));


CompletableFuture<Integer> future1 = CompletableFuture.supplyAsync(() -> 5);
CompletableFuture<Integer> future2 = future1.thenCompose(result -> {
    int doubledResult = result * 2;
    return CompletableFuture.supplyAsync(() -> doubledResult);
});
future2.thenAccept(result -> System.out.println("Final result: " + result));
```

#### Demo

##### 需求：

有一個計數器會統計目前的數字。由於有不同執行緒會按下計數器，所以需要將計數器物件進行鎖定，讓該計數器增加一下的動作，僅能有一個執行緒進行操作，以避免計數器數字不正確。

##### Count：

```
public class Count {
 int cnt = 0;
 
 public void addctn() {
  synchronized (this) {
   try {
    Thread.sleep(100);
   } catch (InterruptedException e) {
    e.printStackTrace();
   }
   cnt++;
  }
 }

 public int getctn() {
  synchronized (this) {
   return cnt;
  }
 }
}
```

##### 模擬 20 個執行緒，共同對計數器進行操作。並且使用固定的Thread Pool進行管理。

```
 public static void main(String[] args) throws InterruptedException {

  Count count = new Count();

  ExecutorService executor = Executors.newFixedThreadPool(5);

  for (int i = 0; i < 20; i++) {
   CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
    String threadName = Thread.currentThread().getName();
    System.out.println("run " + threadName + " thread");
    count.addctn();
   }, executor);
   future.join();
  }

  executor.shutdown();

  System.out.println("計數器:" + count.getctn());
 }
---
run pool-1-thread-1 thread
run pool-1-thread-2 thread
run pool-1-thread-3 thread
run pool-1-thread-4 thread
run pool-1-thread-5 thread
run pool-1-thread-1 thread
run pool-1-thread-2 thread
run pool-1-thread-3 thread
run pool-1-thread-4 thread
run pool-1-thread-5 thread
run pool-1-thread-1 thread
run pool-1-thread-2 thread
run pool-1-thread-3 thread
run pool-1-thread-4 thread
run pool-1-thread-5 thread
run pool-1-thread-1 thread
run pool-1-thread-2 thread
run pool-1-thread-3 thread
run pool-1-thread-4 thread
run pool-1-thread-5 thread
計數器:20
```
