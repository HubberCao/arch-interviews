## ReentrantLock

### 是如何实现可重入性的？

```
为每个锁关联一个获取计数器和一个所有者线程,当计数值为0的时候,这个所就没有被任何线程只有.当线程请求一个未被持有的锁时,
JVM将记下锁的持有者,并且将获取计数值置为1,如果同一个线程再次获取这个锁,技术值将递增,退出一次同步代码块,计算值递减,
当计数值为0时,这个锁就被释放.ReentrantLock里面有实现
```
[可重入锁ReentrantLock详解](https://www.iteye.com/blog/donald-draper-2360411)
[Java可重入锁学习笔记](https://www.shiyanlou.com/questions/2460/)
