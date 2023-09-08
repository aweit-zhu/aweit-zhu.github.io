### Thread & Semaphore

Semaphore (訊號)： 定義有幾個執行緒，允許執行以下代碼 (permits)。Acquire a permit，代表先取得一個 permit，接著以下代碼就可以執行。如果無法取得 permit，則會 lock。需要等到有其他程序執行 Release a permit，該 lock 的執行緒，才會接著繼續進行 (解lock)。

>參考：How to use a Semaphore in Java with code examples
<https://davidvlijmincx.com/posts/how-to-use-java-semaphore/>

#### 需求

如何用不同執行緒，才能每次皆能依序列印出 1、2、3。

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