# Spring事务传播与UnexpectedRollbackException

```Java
ServiceA {
    void methodA()  {
//。      insert or update
        serviceB.methodB();    
    }
}
ServiceB {
    void methodB() {
//        insert or update 
    }    
}
```

拿上面的代码举例：

1. 如果serviceB.methodB的事务传播级别为@Transactional(propagation = Propagation.REQUIRED)
   1. 如果serviceA.methodA没有事务管理，methodB则会给自己创建一个新的事务，如果methodB异常回滚，methodA并不会回滚，因为不受事务控制；
   2. 如果methodA有事务管理，则methodB延续使用methodA的事务，如果methodB异常，methodB回滚，methodA会报org.springframework.transaction.UnexpectedRollbackException: Transaction rolled back because it has been marked as rollback-only，同时也会回滚。
2. 讨论1.b的解决方案，假设methodA的事务传播级别为@Transactional(propagation = Propagation.REQUIRED)
   1. 如果methodB的事务传播级别为@Transactional(propagation = Propagation.REQUIRES_NEW)，执行到methodB，methodA的事务挂起，methodB创建新的事务
      1. 如果methodB发生异常，同时methodA没有catch异常，则methodA，methodB都回滚；
      2. 如果methodB发生异常，methodA catch处理了这个异常，并没有再往上抛异常，methodB回滚，methodA正常提交；
      3. 如果methodB正常提交，但是methodA在调用methodB之后发生异常，methodB并不会回滚，methodA回滚。
   2. 如果如果methodB的事务传播级别为@Transactional(propagation = Propagation.NESTED)，执行到methodB，methodA的事务挂起，methodB创建子事务并保存savepoint
      1. 如果methodB发生异常，同时methodA没有catch异常，则methodA，methodB都回滚；
      2. 如果methodB发生异常，methodA catch处理了这个异常，并没有再往上抛异常，methodB回滚，methodA正常提交；
      3. 如果methodB正常提交，但是methodA在调用methodB之后发生异常，methodB，methodA都回滚。