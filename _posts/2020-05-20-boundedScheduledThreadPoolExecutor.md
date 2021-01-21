## 一个有边界的ScheduledThreadPoolExecutor，仅供参考
```java
import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.RejectedExecutionException;
import java.util.concurrent.ScheduledFuture;
import java.util.concurrent.ScheduledThreadPoolExecutor;
import java.util.concurrent.Semaphore;
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.TimeUnit;

@Slf4j
public class BoundedScheduledExecutorUtil extends ScheduledThreadPoolExecutor  {

    private String poolName;

    private Semaphore semaphore;

    public BoundedScheduledExecutorUtil(int corePoolSize, ThreadFactory threadFactory,int bound,String poolName) {
        super(corePoolSize, threadFactory);
        semaphore=new Semaphore(bound);
        this.poolName=poolName;
    }

    @Override
    public ScheduledFuture<?> schedule(Runnable command,
                                       long delay, TimeUnit unit) {
        //log.info("queue size {}, bound={}",super.getQueue().size(),semaphore.availablePermits());
        if (!semaphore.tryAcquire()) {
            throw new RejectedExecutionException("Task " + command.toString() +
                    " rejected from " +
                    this.toString());
        }
        return super.schedule(command, delay, unit);
    }

    @Override
    protected void afterExecute(Runnable r, Throwable t) {
        semaphore.release();
        log.info(String.format("%s-pool-monitor: AvailablePermits:  %d, PoolSize: %d, CorePoolSize: %d, Active: %d, Completed: %d, Task: %d, Queue: %d, " +
                        "LargestPoolSize: %d, MaximumPoolSize: %d,  KeepAliveTime: %d, isShutdown: %s, isTerminated: %s",
                poolName, this.getAvailablePermits(), this.getPoolSize(), this.getCorePoolSize(), this.getActiveCount(), this.getCompletedTaskCount(), this.getTaskCount(),
                this.getQueue().size(), this.getLargestPoolSize(), this.getMaximumPoolSize(), this.getKeepAliveTime(TimeUnit.MILLISECONDS),
                this.isShutdown(), this.isTerminated()));
    }

    public int getAvailablePermits(){
        return semaphore.availablePermits();
    }


}
```