# Spring线程池配置

```java
@Configuration
@EnableAsync
public class TaskExecutorConfig {

    @Bean("taskExecutor")
    public TaskExecutor taskExecutor(){
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(30);
        executor.setQueueCapacity(Integer.MAX_VALUE);
        executor.setKeepAliveSeconds(60);
        executor.setThreadNamePrefix("member-");
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        //等待所有任务结束后再关闭线程池
        executor.setWaitForTasksToCompleteOnShutdown(true);
        return executor;
    }
}
```

