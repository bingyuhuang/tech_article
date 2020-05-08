---
title: Java源码-锁
tags: java源码
renderNumberedHeading: true
grammar_cjkRuby: true
---

## 概念

多线程：提升性能，也增加程序复杂度

锁：多线程访问一组资源时保证数据一致性。排序获取锁资源。

死锁：两个线程分别占用锁不释放，且等待获取对方的锁释放的过程。

```
public Class DeadLockExamlple {
	public static void mian(String[] args) {
		Object lock1 = new Object();
		Object lock2 = new Object();
		new Thread(()->{
			synchronized(lock1) {
				try{
					Thread.sleep(3);
				} catch(InterruptedException e) {
					e.printStackTrace();
				}
			}
			synchronized(lock2) {
				System.out.println(Thread.currentThread().getName());
			}
		}).start();
		new Thread(()->{
			synchronized(lock2) {
				try{
					Thread.sleep(3);
				} catch(InterruptedException e) {
					e.printStackTrace();
				}
			}
			synchronized(lock1) {
				System.out.println(Thread.currentThread().getName());
			}
		}).start();
	}
}
```

悲观锁&乐观锁

> 悲观锁：保守策略，认为数据在访问过程中一定会被修改，访问资源的整个过程都加锁。synchronized
>
> 乐观锁：认为数据在访问过程中不一定被修改，在访问之前不加锁，在数据提交的时候才加锁。使用CAS(内存值，预期值，新值)，在数据修改时，判断 内存值==预期值，才会修改成新值。会存在ABA问题，修复是使用版本号。

可重入锁

> 同一个线程可以多次获取同一把锁。锁内部存在线程标识和计数器，计数器为0时锁空闲，线程获取锁，计数器加1，线程释放锁，计数器减1

独占锁和共享锁

>独占锁：只能由一个线程获取锁。是悲观锁，如synchronized
>
>共享锁：可以有多个线程同时获取，是乐观锁，如ReadWriteLoack，可以多个线程同时获取读锁。