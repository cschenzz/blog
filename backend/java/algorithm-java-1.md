## 使用java8输出斐波那契数列
```java
@Test
void test01() {
    class FibonacciSupplier implements Supplier<Long> {
        long a = 1;
        long b = 1;
        public Long get() {
            long c = a + b;
            a = b;
            b = c;
            return c;
        }
    }
    FibonacciSupplier supplier = new FibonacciSupplier();
    LongStream natural = Stream.generate(supplier).mapToLong(oo -> Convert.toLong(oo));
    // 注意：无限序列必须先变成有限序列再打印:
    natural.limit(50).forEach(System.out::println);
}
```

## 递归示例(斐波那契数列和计算二进制数)
```java
@Test
void testX02() {
    // 打印20个fibonacci数
    IntStream.rangeClosed(1, 20).forEach(t -> log.info("{}: {}", t, fibonacci(t)));
    // 111011
    log.info("二进制数: {}", getBinaryStr(59));
}

/**
 * 返回第n个斐波那契数
 * 递归实现
 * 1, 1, 2, 3, 5, 8, 13, 21, 34
 *
 * @param n
 * @return
 */
private int fibonacci(int n) {
    // 如果n项为1或者2,则直接返回1
    if (n == 1 || n == 2) {
        return 1;
    }
    return fibonacci(n - 1) + fibonacci(n - 2);
}

/**
 * 求一个正整数num的二进制数
 * 递归实现
 *
 * @param num
 * @return 二进制文本
 */
private String getBinaryStr(int num) {
    log.info("num={}", num);
    // 判断除以2商是否为0,为0就是最后一次计算, 返回
    if (num / 2 == 0) {
        return "" + num % 2;
    }
    // 递归
    return "" + getBinaryStr(num / 2) + num % 2;
}
```
