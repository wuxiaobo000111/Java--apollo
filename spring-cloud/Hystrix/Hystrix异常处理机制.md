# Hystrix异常处理

>&nbsp;&nbsp;&nbsp;&nbsp;Hystrix的异常中有5种情况会被fallback捕获。

```text
    1.FAILURE: 执行失败,抛出异常。
    2.TIMEOUT: 执行超时。
    3.SHORT_CIRCUITED: 断路器打开。
    4.THREAD_POOL_REJECTED: 线程池拒绝。
    5.SEMAPHORE_REJECTED: 信号量拒绝。 
 需要注意的是BAD_REQUEST并不出触发fallback。
```


>&nbsp;&nbsp;&nbsp;&nbsp;如果要想再降级方法中获取异常，只是需要在降级方法参数中加入一个Throwable参数即可。

```java
    @GetMapping("/getFallbackMethodTest")
    @HystrixCommand
    public String getFallbackMethodTest(String id){
        throw new RuntimeException("getFallbackMethodTest failed");
    }

    public String fallback(String id, Throwable throwable) {
        logger.error(throwable.getMessage());
        return "this is fallback message";
    }

```