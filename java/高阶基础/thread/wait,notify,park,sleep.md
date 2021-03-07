1. 执行<font color='red'>sleep</font>方法前不需要获得锁，如果在synchronized同步块里执行也不会释放锁
2. 执行<font color='red'>wait,notify</font>方法前需要获得锁，也就是说必须在synchronized代码块里执行；执行完wait方法会立马释放锁，执行完notify要等到当前代码块执行完才会释放锁；注意二者逻辑顺序，否则线程不能正确唤醒
3. 执行<font color='red'>park</font>方法前不需要获得锁，在synchronized同步块里执行也不会释放。park是通过信号量实现的，不需要注意park、unpark的顺序