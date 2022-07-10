网络上已经有很多文章就synchronize机制做过详细的论述了，这篇文章想要简要总结那些文章的内容，集中于用文字描述并尽可能删去粘贴的代码。[openjdk wiki](https://wiki.openjdk.org/display/HotSpot/Synchronization)中的状态转换图是文字版内容的进一步总结。

https://lankong.seiun.me/static/IY0lyd7HJjQ07Fla/8b7bcde372e9a4a9805b7a9f7b290a2c.webp

偏向锁有个启动时间，这段时间内如果有线程正在尝试加锁那么首先申请轻量级锁，想申请偏向锁要么让线程先休眠，要么设置JVM参数。

### 偏向锁

对于开启了偏向锁优化的程序，如果程序处在安全点，意味着多个线程都运行到安全点，虚拟机可以开始执行GC，撤销偏向锁。如果不处在安全点，意味着虚拟机无法开始GC，可能有多个线程处在临界区。程序不处于安全点时尝试获取偏向锁，获取失败意味着有多个线程在竞争资源，升级为轻量级锁。如果没开启偏向锁优化，直接进入轻量级锁的获取。revoke_and_rebias方法的执行流程总结：对象处在匿名偏向状态下，意味着可以为对象添加偏向锁，CAS设置偏向锁；如果对象已经被偏向锁锁定，线程会尝试让对象进入未偏向状态，为对象添加偏向锁，这些步骤通过CAS完成，如果这其中哪步失败了就表示有其他线程也在尝试添加偏向锁，那么这些线程竞争轻量级锁。

```
void ObjectSynchronizer::fast_enter(Handle obj, BasicLock* lock, bool attempt_rebias, TRAPS) {
 //判断是否开启了偏向锁
 if (UseBiasedLocking) { 
     //如果不处于全局安全点
    if (!SafepointSynchronize::is_at_safepoint()) {
      //通过`revoke_and_rebias`这个函数尝试获取偏向锁
      BiasedLocking::Condition cond = BiasedLocking::revoke_and_rebias(obj, attempt_rebias, THREAD);
      //如果是撤销与重偏向直接返回
      if (cond == BiasedLocking::BIAS_REVOKED_AND_REBIASED) {
        return;
      }
    } else {//如果在安全点，撤销偏向锁
      assert(!attempt_rebias, "can not rebias toward VM thread");
      BiasedLocking::revoke_at_safepoint(obj);
    }
    assert(!obj->mark()->has_bias_pattern(), "biases should be revoked by now");
 }

 slow_enter (obj, lock, THREAD) ;
}
```

### 轻量级锁

如果程序未开启偏向锁优化，对象的初始状态是无锁态。线程尝试添加轻量级锁，添加失败时分两种情况：对象处于加锁状态，且对象中的lock record指针指向当前线程，表示线程尝试再次加锁（这个过程也叫重入操作），放弃此次加锁；多个线程竞争轻量级锁，开始尝试获取重量级锁。

### 重量级锁

这种锁需要向操作系统发起软中断获取，只能在内核态完成锁的申请，这也是重量级锁让性能低下的原因，上面两个锁只在用户态就可申请成功。获取重量级锁的过程也叫锁膨胀，每个线程通过自旋获取重量级锁，持有重量级锁的线程竞争加锁，没有添加成功的线程进入阻塞状态等待唤醒。`Object.wait`会让偏向锁升级为重量级锁，线程进入阻塞状态等待唤醒。
