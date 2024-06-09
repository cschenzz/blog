## CompletableFuture使用

```java
@DisplayName("java线程测试")
@Test
void testThread01() {
    System.out.println("main start...");
    Thread t = new Thread(() -> {
        System.out.println("thread run...");
        System.out.println("thread end.");
    });
    // t.setDaemon(true); //设置守护线程
    t.start(); // 启动新线程
    System.out.println(t.getState());
    // t.join();
    System.out.println("main end...");
}

@DisplayName("java线程测试")
@Test
void testThread02() throws InterruptedException {
    Counter c1 = new Counter();
    Counter c2 = new Counter();
    // 对c1进行操作的线程:
    new Thread(() -> {
        c1.add(1);
    }).start();
    new Thread(() -> {
        c1.dec(1);
    }).start();
    // 对c2进行操作的线程:
    new Thread(() -> {
        c2.add(1);
    }).start();
    new Thread(() -> {
        c2.dec(1);
    }).start();
    Thread.sleep(200);
    System.out.println(c1.get());
    System.out.println(c2.get());
}

@DisplayName("CompletableFuture测试")
@Test
void test07() {
    // CompletableFuture cf = CompletableFuture.completedFuture("message");
    // assertTrue(cf.isDone());
    // assertEquals("message", cf.getNow(null));
    CompletableFuture cf = CompletableFuture.runAsync(() -> {
        assertTrue(Thread.currentThread().isDaemon());
        // randomSleep();
    });
    assertFalse(cf.isDone());
    // sleepEnough();
    assertTrue(cf.isDone());
}

@DisplayName("CompletableFuture测试")
@Test
void test01() throws InterruptedException {
    // 创建异步执行任务:
    CompletableFuture<Double> cf = CompletableFuture.supplyAsync(() -> {
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
        }
        if (Math.random() < 0.3) {
            throw new RuntimeException("fetch price failed!");
        }
        return 5 + Math.random() * 20;
    });
    // 如果执行成功:
    cf.thenAccept((result) -> {
        System.out.println("price: " + result);
    });
    // 如果执行异常:
    cf.exceptionally((e) -> {
        e.printStackTrace();
        return null;
    });
    // 主线程不要立刻结束，否则CompletableFuture默认使用的线程池会立刻关闭:
    Thread.sleep(200);
}

@DisplayName("多个CompletableFuture可以串行执行")
@Test
void test02() throws InterruptedException {
    // 定义两个CompletableFuture，第一个CompletableFuture根据证券名称查询证券代码，第二个CompletableFuture根据证券代码查询证券价格，这两个CompletableFuture实现串行操作
    // 第一个任务:
    CompletableFuture<String> cfQuery = CompletableFuture.supplyAsync(() -> {
        return queryCode("中国石油");
    });
    // cfQuery成功后继续执行下一个任务:
    CompletableFuture<Double> cfFetch = cfQuery.thenApplyAsync((code) -> {
        return fetchPrice(code);
    });
    // cfFetch成功后打印结果:
    cfFetch.thenAccept((result) -> {
        System.out.println("price: " + result);
    });
    // 主线程不要立刻结束，否则CompletableFuture默认使用的线程池会立刻关闭:
    Thread.sleep(2000);
}

static String queryCode(String name) {
    try {
        Thread.sleep(100);
    } catch (InterruptedException e) {
    }
    return "601857";
}
static Double fetchPrice(String code) {
    try {
        Thread.sleep(100);
    } catch (InterruptedException e) {
    }
    return 5 + Math.random() * 20;
}

@DisplayName("多个CompletableFuture还可以并行执行")
@Test
void test03() throws InterruptedException {
    // CompletableFuture可以指定异步处理流程：
    //
    // thenAccept()处理正常结果；
    // exceptional()处理异常结果；
    // thenApplyAsync()用于串行化另一个CompletableFuture；
    // anyOf()和allOf()用于并行化多个CompletableFuture。
    // 同时从新浪和网易查询证券代码，只要任意一个返回结果，就进行下一步查询价格，查询价格也同时从新浪和网易查询，只要任意一个返回结果，就完成操作
    // 两个CompletableFuture执行异步查询:
    CompletableFuture<String> cfQueryFromSina = CompletableFuture.supplyAsync(() -> {
        return queryCode("中国石油", "https://finance.sina.com.cn/code/");
    });
    CompletableFuture<String> cfQueryFrom163 = CompletableFuture.supplyAsync(() -> {
        return queryCode("中国石油", "https://money.163.com/code/");
    });
    // 用anyOf合并为一个新的CompletableFuture:
    CompletableFuture<Object> cfQuery = CompletableFuture.anyOf(cfQueryFromSina, cfQueryFrom163);
    // 两个CompletableFuture执行异步查询:
    CompletableFuture<Double> cfFetchFromSina = cfQuery.thenApplyAsync((code) -> {
        return fetchPrice((String) code, "https://finance.sina.com.cn/price/");
    });
    CompletableFuture<Double> cfFetchFrom163 = cfQuery.thenApplyAsync((code) -> {
        return fetchPrice((String) code, "https://money.163.com/price/");
    });
    // 用anyOf合并为一个新的CompletableFuture:
    CompletableFuture<Object> cfFetch = CompletableFuture.anyOf(cfFetchFromSina, cfFetchFrom163);
    // 最终结果:
    cfFetch.thenAccept((result) -> {
        System.out.println("price: " + result);
    });
    // 主线程不要立刻结束，否则CompletableFuture默认使用的线程池会立刻关闭:
    Thread.sleep(200);
}

static String queryCode(String name, String url) {
    System.out.println("query code from " + url + "...");
    try {
        Thread.sleep((long) (Math.random() * 100));
    } catch (InterruptedException e) {
    }
    return "601857";
}

static Double fetchPrice(String code, String url) {
    System.out.println("query price from " + url + "...");
    try {
        Thread.sleep((long) (Math.random() * 100));
    } catch (InterruptedException e) {
    }
    return 5 + Math.random() * 20;
}

//-------------------------------
@DisplayName("Future测试")
@Test
void test11() throws ExecutionException, InterruptedException {
    ExecutorService service = Executors.newSingleThreadExecutor();
    Future<Integer> future = service.submit(() -> {
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return 1;
    });
    System.out.println("---------------");
    System.out.println(future.get());
    System.out.println("finish!!!");
}
```

---------------------
- [JAVA基于CompletableFuture的流水线并行处理深度实践](https://juejin.cn/post/7124124854747398175)
