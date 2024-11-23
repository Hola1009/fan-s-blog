> Lock锁, 可以得到和 synchronized 一样的效果, 即实现[[Sychronized 的 原子性 有序性 可见性]]
> 相较于synchronized, Lock锁可手动获取锁和释放锁, 可中断的获取锁, 超时获取锁
> Lock 是一个接口, 两个实现类: ReentrantLock (重入锁), ReentrantReadWriteLock (读写锁)

