### 2SUM 總和問題

#### 雜湊表

```
public static int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> index = new HashMap<>();

    for (int i = 0; i < nums.length; i++) {
        index.put(nums[i], i);
    }

    for (int i = 0; i < nums.length; i++) {
        int other = target - nums[i];
        if (index.containsKey(other) && index.get(other) != i) {
            return new int[] { i, index.get(other) };
        }
    }

    return new int[] { -1, -1 };

}
```

#### 雙指標 (單一解)

```
public static int[] twoSumI(int[] nums, int target) {

    int[] sortedNums = Arrays.copyOf(nums, nums.length);

    Arrays.sort(sortedNums);

    int left = 0, right = sortedNums.length - 1;

    while (left < right) {
        int sum = sortedNums[left] + sortedNums[right];

        if (sum == target) {
            int index1 = -1, index2 = -1;
            for (int i = 0; i < nums.length; i++) {
                if (nums[i] == sortedNums[left] && index1 == -1) {
                    index1 = i;
                } else if (nums[i] == sortedNums[right]) {
                    index2 = i;
                }
            }
            return new int[] { index1, index2 };
        } else if (sum < target) {
            left++;
        } else if (sum > target) {
            right--;
        }
    }

    return new int[] { -1, -1 };
}

public static void main(String[] args) {
    int target = 6;
    int[] nums = new int[] { 3, 1, 3, 5 };
    int[] res = twoSumI(nums, target);
    System.out.printf("%d[%d] + %d[%d] = %d", nums[res[0]], res[0], nums[res[1]], res[1], target);
}

```

#### 雙指標 (所有解)

```
public static List<int[]> twoSumII(int[] nums, int target) {

    // 紀錄原始陣列的值與索引位置
    Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        map.put(nums[i], i);
    }

    // 複製一份原始陣列，並且排序，以進行雙指標演算法
    int[] sortedNums = Arrays.copyOf(nums, nums.length);
    Arrays.sort(sortedNums);

    List<int[]> res = new ArrayList<int[]>();
    
    int left = 0, right = sortedNums.length - 1;

    while (left < right) {
        int leftVal = sortedNums[left];
        int rightVal = sortedNums[right];
        int sum = leftVal + rightVal;
        if (sum == target) {

            int index1 = map.get(leftVal);
            int index2 = map.get(rightVal);

            res.add(new int[] { index1, index2 });

            // 跳過重複的作法
            while (left < right && sortedNums[left] == leftVal)
                left++;
            while (left < right && sortedNums[right] == rightVal)
                right--;

        } else if (sum < target) {
            while (left < right && sortedNums[left] == leftVal)
                left++;
        } else if (sum > target) {
            while (left < right && sortedNums[right] == rightVal)
                right--;
        }
    }
    return res;
}

public static void main(String[] args) {
    int target = 4;
    int[] nums = new int[] { 1, 1, 1, 2, 2, 3, 3 };
    List<int[]> resList = twoSumII(nums, target);
    for (int[] res : resList)
        System.out.printf("%d[%d] + %d[%d] = %d\n", nums[res[0]], res[0], nums[res[1]], res[1], target);
}

```

### 3SUM 總和問題

(待續)