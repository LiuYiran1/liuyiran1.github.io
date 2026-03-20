##### Java 8+ 链式调用sort

```java
// 场景 C：基本排序（按分数从小到大）
list.sort(Comparator.comparingInt(Student::getScore));

// 场景 D：倒序（按分数从大到小）
list.sort(Comparator.comparingInt(Student::getScore).reversed());

// 场景 E：多级排序（先按分数从高到低；如果分数一样，再按年龄从小到大）
list.sort(
    Comparator.comparingInt(Student::getScore).reversed()
              .thenComparingInt(Student::getAge)
);
```

##### map.merge(key, value, remappingFunction)

1. **c (Key)**：你要操作的那个键（例如：一个字符或字符串）。

2. **1 (Value)**：如果键 c 在 Map 中**还不存在**，要插入的默认值。

3. **Integer::sum (Function)**：如果键 c **已经存在**，如何处理旧值（oldValue）和新值（1）。Integer::sum 是一个方法引用，表示 (oldValue, 1) -> oldValue + 1

##### java快速进行数组copy的办法

```java
int[] src = {1, 2, 3, 4, 5};
int[] dest = new int[5];

// 参数：源数组, 源起始位, 目标数组, 目标起始位, 长度
System.arraycopy(src, 0, dest, 0, src.length);
```

```java
int[] src = {1, 2, 3, 4, 5};

// 复制整个数组到一个新数组
int[] dest = Arrays.copyOf(src, src.length);

// 复制某个范围 [from, to)
int[] range = Arrays.copyOfRange(src, 1, 3); // 结果: {2, 3}
```

##### 简化写法

```java
	List<List<Integer>> ans = new LinkedList<>();
    List<Integer> tem = new ArrayList<>();
    tem.add(boxedArr[0]);
    tem.add(boxedArr[1]);
    ans.add(tem);
```

看着冗余

修改：
```java
List<List<Integer>> ans = new LinkedList<>();
ans.add(List.of(boxedArr[0], boxedArr[1]));
```

- List.of 生成的 list不可变

```java
ans.add(Arrays.asList(boxedArr[0], boxedArr[1]));
```

- Arrays.asList 长度不能变

```java
// 支持后续修改的写法
ans.add(new ArrayList<>(Arrays.asList(boxedArr[0], boxedArr[1])));
```



