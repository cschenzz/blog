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

    // -------启用异步线程执行耗时任务--------
    // 打印 1, 4, 2, 3
    Console.log("1");
    CompletableFuture.runAsync(() -> {
        Console.log("2.开始执行异步线程");
        cn.hutool.core.thread.ThreadUtil.safeSleep(5000);
        Console.log("3.线程任务结束");
    });
    Console.log("4");
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


## CompletableFuture单元测试
```java
package com.example.spring.test;

import cn.hutool.core.lang.Console;
import cn.hutool.core.thread.ThreadUtil;
import cn.hutool.core.util.RandomUtil;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

import java.util.concurrent.CompletableFuture;

/**
 * CompletableFuture测试
 */
public class CompletableFutureTests {

    @DisplayName("CompletableFuture测试")
    @Test
    void testX01() {
        CompletableFuture<Void> completableFuture = CompletableFuture.runAsync(() -> {
            System.out.println("当前线程" + Thread.currentThread().getId());
            int i = 10 / 2;
            System.out.println("运行结果：" + i);
        });

        // supplyAsync有返回值
        // handle能拿到返回结果，也能得到异常信息，也能修改返回值
        // CompletableFuture<String> future =
        CompletableFuture.supplyAsync(() -> {
            System.out.println("当前线程" + Thread.currentThread().getId());
            int i = 10 / 4;
            System.out.println("运行结果：" + i);
            return i;
        }).handle((res, exception) -> {
            if (exception != null) {
                return "--";
            } else {
                return "xx-" + res * 2;
            }
        });
    }

    @DisplayName("多个CompletableFuture可以串行执行")
    @Test
    void testX02() {
        // 异步串行执行, 1-2-3依次执行, 1的结果为2的输入, 2的结果为3的输入
        CompletableFuture.supplyAsync(() -> {
            Console.log("step.1");
            ThreadUtil.sleep(100);
            return "中国石油";
        }).thenApplyAsync((company) -> {
            Console.log("step.2, {}", company);
            ThreadUtil.sleep(100);
            return 5 + Math.random() * 20;
        }).thenAccept((result) -> {
            System.out.println("3.price: " + result);
        });

        // 主线程不要立刻结束，否则CompletableFuture默认使用的线程池会立刻关闭
        ThreadUtil.sleep(2000);
    }

    @DisplayName("多个CompletableFuture还可以并行执行")
    @Test
    void testX03() {
        // 模拟分别使用2个线程分别从新浪, 网易获取中国石油的股票代码, 然后那个先执行完就使用那个返回的股票代码去查询当前股票价格, 最后打印股票价格
        CompletableFuture.anyOf(CompletableFuture.supplyAsync(() -> {
            Console.log("通过sina新浪查询中国石油代码");
            long sleepMs = RandomUtil.randomLong(200, 400);
            ThreadUtil.sleep(sleepMs);
            Console.log("-----查询成功(耗时{}ms), from sina------", sleepMs);
            return "601857-sina";
        }), CompletableFuture.supplyAsync(() -> {
            Console.log("通过163网易查询中国石油代码");
            long sleepMs = RandomUtil.randomLong(200, 400);
            ThreadUtil.sleep(sleepMs);
            Console.log("-----查询成功(耗时{}ms), from 163------", sleepMs);
            return "601857-163";
        })).thenApplyAsync((code) -> {
            Console.log("查询[{}]股票价格", code);
            ThreadUtil.sleep(RandomUtil.randomLong(400, 500));
            return RandomUtil.randomDouble(5, 200);
        }).thenAccept((result) -> {
            System.out.println("股票价格price: " + result);
        });

        // 主线程不要立刻结束，否则CompletableFuture默认使用的线程池会立刻关闭
        ThreadUtil.sleep(2000);
    }

    @DisplayName("多个CompletableFuture还可以并行执行-2")
    @Test
    void testX04() {
        // 模拟分别使用2个线程分别从新浪, 网易获取中国石油的股票代码, 然后那个先执行完就使用那个返回的股票代码分别通过新浪和网易去查询当前股票价格, 那个先执行完先打印对应的打印股票价格
        CompletableFuture<Object> cfQuery = CompletableFuture.anyOf(CompletableFuture.supplyAsync(() -> {
            Console.log("通过sina新浪查询中国石油代码");
            long sleepMs = RandomUtil.randomLong(200, 400);
            ThreadUtil.sleep(sleepMs);
            Console.log("-----1.查询成功(耗时{}ms), from sina------", sleepMs);
            return "601857-sina";
        }), CompletableFuture.supplyAsync(() -> {
            Console.log("通过163网易查询中国石油代码");
            long sleepMs = RandomUtil.randomLong(200, 400);
            ThreadUtil.sleep(sleepMs);
            Console.log("-----1.查询成功(耗时{}ms), from 163------", sleepMs);
            return "601857-163";
        }));

        CompletableFuture.anyOf(cfQuery.thenApplyAsync((code) -> {
            Console.log("通过sina查询股票价格");
            long sleepMs = RandomUtil.randomLong(100, 300);
            ThreadUtil.sleep(sleepMs);
            Console.log("-----2.查询成功(耗时{}ms), from sina------", sleepMs);
            return "99元-sina";
        }), cfQuery.thenApplyAsync((code) -> {
            Console.log("通过163查询股票价格");
            long sleepMs = RandomUtil.randomLong(100, 300);
            ThreadUtil.sleep(sleepMs);
            Console.log("-----2.查询成功(耗时{}ms), from 163------", sleepMs);
            return "99元-163.com";
        })).thenAccept((result) -> {
            System.out.println("股票价格price: " + result);
        });

        // 主线程不要立刻结束，否则CompletableFuture默认使用的线程池会立刻关闭
        ThreadUtil.sleep(2000);
    }

}
```

##  ScheduledExecutorService延时任务

`ScheduledExecutorService` 是 Java 并发工具包中用于定时任务调度的接口，它扩展了 `ExecutorService` 接口，提供了延迟执行和周期性执行任务的能力。
```java
// 1.执行异步延时任务, 输出1,2,9秒...
Console.log("---1.start---");
java.util.concurrent.Executors.newSingleThreadScheduledExecutor().schedule(() -> {
    // ----------------------------
    Console.log("9秒异步延时任务执行完成");
    // ----------------------------
}, 9, TimeUnit.SECONDS);
Console.log("---2.end---");


// 2.返回ScheduledExecutorService, 用法同上
public ScheduledExecutorService getExecutor() {
    // 创建一个固定大小的线程池
    // java.util.concurrent.Executors.newScheduledThreadPool(5);
    // ----------------------------------
    return new ScheduledThreadPoolExecutor(5,
            new BasicThreadFactory.Builder().namingPattern("schedule-pool-%d").daemon(true).build(),
            new ThreadPoolExecutor.CallerRunsPolicy()) {
        @Override
        public void afterExecute(Runnable r, Throwable t) {
            super.afterExecute(r, t);
            // printException(r, t);
        }
    };
}
```

---------------------
- [JAVA基于CompletableFuture的流水线并行处理深度实践](https://juejin.cn/post/7124124854747398175)
