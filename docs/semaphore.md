### Thread & Semaphore

在 Java 中，Semaphore 可以用來控制多個執行緒對同一物件的存取。Semaphore 會維護一個數字，稱為 permits，代表可用的許可數量。當一個執行緒想要存取該物件時，它會先呼叫 acquire() 方法來獲取一個許可。如果 permits 大於 0，執行緒會成功獲取一個許可，並且 permits 的數量會減少。如果 permits 等於 0，執行緒會等待直到有其他執行緒釋放許可。

當一個執行緒完成對該物件的存取後，它會呼叫 release() 方法來釋放許可。這會導致 permits 的數量增加，並且可能會喚醒正在等待許可的其他執行緒。

Semaphore 的使用可以簡化程式碼，讓它更易讀和理解。相較於使用 synchronized、wait 和 notifyAll 來實現同樣的功能，Semaphore 提供了一種更直觀的方式來控制多個執行緒對同一物件的存取。

```
public class SharedObject {
    private Semaphore semaphore = new Semaphore(1);
    
    public void doSomething() {
        try {
            semaphore.acquire();
            // 存取共享物件的程式碼
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            semaphore.release();
        }
    }
}
```
在上面的範例中，我們創建了一個 Semaphore 物件，並且初始化 permits 為 1，表示只有一個許可。在 doSomething() 方法中，我們首先呼叫 acquire() 方法來獲取許可，然後進行對共享物件的存取操作。最後，我們呼叫 release() 方法來釋放許可。

這樣，當多個執行緒同時存取 SharedObject 物件時，只有一個執行緒可以成功獲取許可，其他執行緒則需要等待。這樣可以確保同一時間只有一個執行緒在存取該物件，避免了錯亂的情況。


>參考：How to use a Semaphore in Java with code examples
<https://davidvlijmincx.com/posts/how-to-use-java-semaphore/>

#### 範例

如何用不同執行緒，呼叫不同方法，讓其每次輸出能依序列印出 1、2、3。

```
class Foo {

    public Foo() {
    }

    public void first() throws InterruptedException {
        System.out.println("1");
    }

    public void second() throws InterruptedException {
        System.out.println("2");
    }

    public void third() throws InterruptedException {
        System.out.println("3");
    }

    public static void main(String[] args) throws InterruptedException {
        Foo foo = new Foo();

        Runnable printFirst = new Runnable() {
            @Override
            public void run() {
                try {
                    foo.first();
                } catch (InterruptedException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
            }
        };

        Runnable printSecond = new Runnable() {
            @Override
            public void run() {
                try {
                    foo.second();
                } catch (InterruptedException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }

            }

        };

        Runnable printThird = new Runnable() {
            @Override
            public void run() {
                try {
                    foo.third();
                } catch (InterruptedException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
            }
        };

        Thread one = new Thread(printFirst);
        Thread two = new Thread(printSecond);
        Thread three = new Thread(printThird);

        three.start();
        two.start();
        one.start();
	}
}
```

#### 方法一、用 synchronized、wait、notifyAll 方法實現

```
public class Foo {

    boolean a;

    boolean b;

    boolean c;

    public synchronized void first() throws InterruptedException {
        System.out.println("1");
        a = true;
        notifyAll();
    }

    public synchronized void second() throws InterruptedException {
        while(!a) {
            wait();
        }
        System.out.println("2");
        b = true;
        notifyAll();
    }

    public synchronized void third() throws InterruptedException {
        while(!b) {
            wait();
        }
        System.out.println("3");
        c = true;
        notifyAll();
    }

    public Foo() {
        a = false;
        b = false;
        c = false;
    }

    // main 方法省略
}
```

#### 方法二、用 Semaphore 方法實現

```
public class Foo2 {

	private Semaphore a = new Semaphore(1);

	private Semaphore b = new Semaphore(0);

	private Semaphore c = new Semaphore(0);

	public Foo2() {
	}

	public void first() throws InterruptedException {
		a.acquire(1);
		System.out.println("1");
		b.release(1);
	}

	public void second() throws InterruptedException {
		b.acquire(1);
		System.out.println("2");
		c.release(1);
	}

	public void third() throws InterruptedException {
		c.acquire(1);
		System.out.println("3");
	}

    // main 方法省略
}
```