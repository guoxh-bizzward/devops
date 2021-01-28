# Condition

## synchronized



## Lock



## Condition

* Condition中的await方法相当于Object的await方法,Condition中的signal方法相当于Object的notify方法,Condition中的signalAll相当于Object的notifyAll方法.不同的是,Object中的这些方法是和同步锁捆绑使用的.而Condition是需要和互斥锁/共享锁绑定使用的.
* Condition更强大的地方在于:能够更加精确的控制多线程的休眠和唤醒.对于同一个锁,我们可以创建多个Condition,在不同情况下使用不同的Condition.

```
例如,在多线程读/写同一个缓冲区,当向缓冲区写入数据后,唤醒"读线程";当从缓冲区读出数据后,唤醒"写线程";当并且当缓存区满的时候,"写线程"需要等待,当缓冲区为空时,"读线程"需要等待;
如果采用Object类中的wait(), notify(), notifyAll()实现该缓冲区，当向缓冲区写入数据之后需要唤醒"读线程"时，不可能通过notify()或notifyAll()明确的指定唤醒"读线程"，而只能通过notifyAll唤醒所有线程(但是notifyAll无法区分唤醒的线程是读线程，还是写线程)。  但是，通过Condition，就能明确的指定唤醒读线程。
```

