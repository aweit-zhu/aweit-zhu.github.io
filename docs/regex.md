### Java Regular Expression

![Alt text](image-181.png)

1. W3C <https://www.w3schools.com/java/java_regex.asp>
2. Chat GPT

#### Regular Expression 正則表達式

**"正規表示式"**（Regular Expression）是一種用於描述和匹配字串模式的工具。它是由一系列特殊字符和字元組成的模式，用於在字串中搜尋、替換和驗證特定的模式。

正規表示式在各種程式語言和文本處理工具中都被廣泛使用。它們提供了一種強大的方式來處理字串，例如：

- Find & Group：搜尋和匹配字串中的特定模式
- Find & Replace：替換字串中的特定模式為指定的字元或字串
- Find & Valid：驗證字串是否符合特定的模式

正規表示式使用特殊的字元和符號來表示不同的模式，例如：

- `.`：匹配任何單個字元
- `*`：匹配前面的模式零次或多次
- `+`：匹配前面的模式一次或多次
- `?`：匹配前面的模式零次或一次
- `[]`：匹配方括號中的任何單個字元
- `()`：創建群組，可以對群組中的模式進行操作
- `\d`：匹配任何數字字符。等效於 `[0-9]`。
- `\D`：匹配任何非數字字符。等效於 `[^0-9]`。
- `\w`：匹配任何字母、數字和底線字符。等效於 `[a-zA-Z0-9_]`。
- `\W`：匹配任何非字母、非數字和非底線字符。等效於 `[^a-zA-Z0-9_]`。
- `\s`：匹配任何空白字符，包括空格、製表符、換行符等。
- `\S`：匹配任何非空白字符。
- `\b`：匹配單詞的邊界。例如，`\bword\b` 可以匹配單詞 "word"，但不會匹配 "words" 或 "sword"。
- `\B`：匹配非單詞的邊界。
- `{n,m}`: 匹配前面的模式n次~m次之間。

正規表示式可以非常複雜，但它們提供了一種強大且靈活的方式來處理字串模式。學習和掌握正規表示式將有助於你更有效地處理和操作字串。

#### Patern、Match

```
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class Main {
  public static void main(String[] args) {
    Pattern pattern = Pattern.compile("w3schools", Pattern.CASE_INSENSITIVE);
    Matcher matcher = pattern.matcher("Visit W3Schools!");
    boolean matchFound = matcher.find();
    if(matchFound) {
      System.out.println("Match found");
    } else {
      System.out.println("Match not found");
    }
  }
}
```
當你需要在 Java 中使用正規表示式時，可以使用 `Pattern.compile` 方法將正規表示式編譯成一個 `Pattern` 對象。第一個參數為 `正規表示式`。第二個參數為 可選值，代表 `忽略大小寫`。

當你已經將正規表示式編譯為 `Pattern` 對象後，接著可以使用 `Matcher` 對象來執行具體的操作，如匹配、搜尋、替換等。


**匹配**

```
String input = "The cat is on the mat.";
String regex = "cat";
Pattern pattern = Pattern.compile(regex);
Matcher matcher = pattern.matcher(input);

if (matcher.find()) {
    System.out.println("匹配成功");
} else {
    System.out.println("匹配失敗");
}
```

**搜尋**
```
String input = "The cat is on the mat.";
String regex = "\\b\\w{3}\\b";
Pattern pattern = Pattern.compile(regex);
Matcher matcher = pattern.matcher(input);

while (matcher.find()) {
    System.out.println("找到匹配：" + matcher.group());
}

---
找到匹配：The
找到匹配：cat
找到匹配：the
找到匹配：mat
```

**替換**
```
String input = "The cat is on the mat.";
String regex = "\\bcat\\b";
Pattern pattern = Pattern.compile(regex);
Matcher matcher = pattern.matcher(input);
String replaced = matcher.replaceAll("dog");

System.out.println("替換後的結果：" + replaced);

---
替換後的結果：The dog is on the mat.
```

**分組搜尋**

```
String input = "John Doe, Jane Smith";
String regex = "(\\w+) (\\w+)";
Pattern pattern = Pattern.compile(regex);
Matcher matcher = pattern.matcher(input);

while (matcher.find()) {
    String fullName = matcher.group(0); // 整個匹配
    String firstName = matcher.group(1); // 第一個分組
    String lastName = matcher.group(2); // 第二個分組

    System.out.println("完整姓名：" + fullName);
    System.out.println("名字：" + firstName);
    System.out.println("姓氏：" + lastName);
}

---
完整姓名：John Doe
名字：John
姓氏：Doe
完整姓名：Jane Smith
名字：Jane
姓氏：Smith
```

> - `(\\w+)`：這是第一個分組，用於匹配一個或多個字母、數字或下劃線字符。`\\w` 表示匹配任何字母、數字或下劃線字符，而 `+` 表示匹配前面的模式一次或多次。
>- 空格：這個空格用於匹配一個空格字符。
> - `(\\w+)`：這是第二個分組，與第一個分組的模式相同，用於匹配另一個一個或多個字母、數字或下劃線字符。


#### 範例


1. 找出所有數字。

```
String input = "ab12c6";
String regex = "\\d+";
Pattern pattern = Pattern.compile(regex);
Matcher matcher = pattern.matcher(input);
while(matcher.find()) {
    System.out.println(matcher.group());
}

---
12
6
```

2. 找出所有非數字

```
String input = "ab12c6@";
String regex = "\\D+";
Pattern pattern = Pattern.compile(regex);
Matcher matcher = pattern.matcher(input);
while(matcher.find()) {
    System.out.println(matcher.group());
}
---
ab
c
@
```

3. 找出所有字母，但不包含數字。

   `\w`: 匹配任何字母、數字和底線字符。等同 `[a-zA-Z0-9_]`

```
String input = "ab12c6@";
String regex = "[a-zA-Z]";
Pattern pattern = Pattern.compile(regex);
Matcher matcher = pattern.matcher(input);
while(matcher.find()) {
    System.out.println(matcher.group());
}
```

4. 將 004193 去除 0，輸出 4193

```
String input = ""004193"";
String regex = "^0*(\\d{1,}$)";
Pattern pattern = Pattern.compile(regex);
Matcher matcher = pattern.matcher("004193");

if (matcher.find()) {
    System.out.println(m.group(1));
}

---
4193
```