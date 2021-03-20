5. @Transactional注解只能作用在public方法上，原因：private/protected方法对代理类不可见

6. spring事务 非运行时异常能否回滚？可以，配置@Transactional(rollbackFor=Exception.class)

7. service类内部调用事务注解不生效解决（强制使用AspectJ代理）

8. 事务传播性  以下说的上文指的是调用方

    #### l **PROPAGATION_REQUIRED**

    <font color='red'>默认的Spring事务传播级别</font>，如果上文中已经存在事务，那么就加入到事务中执行；如果上文中不存在事务，则新建事务执行。

    #### l **PROPAGATION_SUPPORTS**

    如果上文存在事务，则支持事务加入事务，如果没有事务，则使用非事务的方式执行。

    #### l **PROPAGATION_MANDATORY**

    要求上文中必须要存在事务，否则就会抛出异常！

    #### l **PROPAGATION_REQUIRES_NEW**

    每次都会新建一个事务，并且同时将上下文中的事务挂起，执行当前新建事务完成以后，上下文事务恢复再执行。

    应用场景：现在有一个发送100个红包的操作，在发送之前，要做一些系统的初始化、验证、数据记录操作，然后发送100封红包，然后再记录发送日志，发送日志要求100%的准确，如果日志不准确，那么整个父事务逻辑需要回滚。

    怎么处理整个业务需求呢？就是通过这个PROPAGATION_REQUIRES_NEW级别的事务传播控制就可以完成。发送红包的子事务不会直接影响到父事务的提交和回滚。

    #### l **PROPAGATION_NOT_SUPPORTED**

    若上文中存在事务，则挂起事务，执行当前逻辑，结束后恢复上下文的事务。

    这个级别有什么好处？可以帮助你将事务尽可能的缩小。我们知道一个事务越大，它存在的风险也就越多。所以在处理事务的过程中，要保证尽可能的缩小范围。比如一段代码，是每次逻辑操作都必须调用的，比如循环1000次的某个非核心业务逻辑操作。这样的代码如果包在事务中，势必造成事务太大，导致出现一些难以考虑周全的异常情况，所以事务的这个传播级别就派上用场了。用当前级别的事务模板包含起来就可以了。

    #### l **PROPAGATION_NEVER**

    如果上文有事务，就抛出runtime异常。

    #### l **PROPAGATION_NESTED**
    
    如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作。<font color='orange'>只支持jdbc，且jdbc3.0版本以上，数据库要支持savepoint 保存点。</font>
    
9. 事务传播标志性案例

   #### l **PROPAGATION_NESTED与PROPAGATION_REQUIRED_NEW的区别**
   ```cpp
   class A {
       public void invoke() {
           try {
               new B().invoke();
           catch (Exception e) {
               new C().invoke();
           }        
       };
   }
   ```
   
   不论使用`REQUIRES_NEW`或是`NESTED`，在调用B的invoke时如果发生异常，都能正确完成业务逻辑
   
   - `REQUIRES_NEW`执行到B时，A事物被挂起，B会新开了一个事务进行执行，B发生异常后，B中的修改都会回滚，然后外部事物继续执行
   - `NESTED`执行到B时，会创建一个savePoint，如果B中执行失败，会将数据回滚到这个savePoint
   
   <font color='red'>重点来了，如果B处正常执行，就会产生区别了</font>
   
   - `REQUIRES_NEW`如果B正常执行，则B中的数据在A提交之前已经完成提交，其他线程已经可见其修改，这就意味着可能有脏数据的产生；同时，如果接下来A的其他逻辑发生了异常，A回滚，但是B已经完成提交，不会回滚了。当然，如果A接下来的逻辑没有相关要求，那就无所谓了
   - `NESTED`如果B正常执行，此时B中的修改并不会立即提交，而是在A提交时一并提交，如果A下面的逻辑中发生异常，A回滚时，B中的修改也会回滚，就可以避免上述情况的发生。
   #### l **try-catch处理同一事务问题**
   
   接以上，如果B().invoke()是`REQUIRES`或是`SUPPORTS`或者`MANDATORY`等这种，A().invoke()与B().invoke()为同一个事务，只要B().invoke()发生异常，<font color='red'>即使A中对B做了异常捕获，也会导致A回滚</font>，因为B报错报纸事务已经标记为回滚，异常如下<font color='red'>`org.springframework.transaction.UnexpectedRollbackException: Transaction rolled back because it has been marked as rollback-only`</font>