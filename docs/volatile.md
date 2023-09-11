### Volatile & synchronized

參考資料：

1. <https://www.tpisoftware.com/tpu/articleDetails/1753>
2. Chat GPT

#### 變數可視性(visibility)

> 可視性問題指的是當一個執行緒修改了共享變數的值時，其他執行緒能夠立即看到這個修改。

#### 資料競速(race condition)

> 競態條件是指多個執行緒在沒有適當同步的情況下，對共享資源進行讀寫操作時可能出現的不確定性結果。當多個執行緒同時讀取和修改共享變數時，它們的操作順序是不確定的，可能導致不一致的結果或錯誤的計算。

#### Volatile & synchronized

1. synchronized： 同時解決 「變數可視性」與「資料競速」問題。

2. volatile：解決「變數可視性」。(使用此修飾符的變數，不使用各執行緒的working memory，永遠從主記憶體(main memory) 做存取與讀寫)

#### 如何證明 volatile 的效果？

1. 沒有用 volatile

```
public class VisibilityProblem {
	
	static int num;

	public static void main(String[] args) {

		Thread readerThread = new Thread(() -> {
			int temp = 0;
			while (true) {
				if (temp != num) {
					temp = num;
					System.out.println("reader: value of num = " + num);
				}
			}
		});

		Thread writerThread = new Thread(() -> {
			for (int i = 0; i < 5; i++) {
				num++;
				System.out.println("writer: changed value to = " + num);

				try {
					Thread.sleep(1000);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}

			// 離開程式，否則readerThread會一直等待變數int num的值改變
			System.exit(0);
		});

		readerThread.start();
		writerThread.start();
	}
}

---
reader: value of num = 1
writer: changed value to = 1
writer: changed value to = 2
writer: changed value to = 3
writer: changed value to = 4
writer: changed value to = 5
```

2. 有用 volatile：讓 num 變數，永遠從主記憶體中讀取與寫入。

```
public class VisibilityProblem {
	
	static volatile int num;

	public static void main(String[] args) {

		Thread readerThread = new Thread(() -> {
			int temp = 0;
			while (true) {
				if (temp != num) {
					temp = num;
					System.out.println("reader: value of num = " + num);
				}
			}
		});

		Thread writerThread = new Thread(() -> {
			for (int i = 0; i < 5; i++) {
				num++;
				System.out.println("writer: changed value to = " + num);

				try {
					Thread.sleep(1000);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}

			// 離開程式，否則readerThread會一直等待變數int num的值改變
			System.exit(0);
		});

		readerThread.start();
		writerThread.start();
	}
}
---
reader: value of num = 1
writer: changed value to = 1
writer: changed value to = 2
reader: value of num = 2
writer: changed value to = 3
reader: value of num = 3
reader: value of num = 4
writer: changed value to = 4
writer: changed value to = 5
reader: value of num = 5
```