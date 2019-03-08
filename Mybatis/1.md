### 什么是一级缓存？  
原生mybatis默认打开一级缓存，且缓存范围是SESSION,意思是同一个session，执行相同的sql（参数也相同），第一次执行后，后面执行都能从缓存中读取。这里就不写demo了，可以参考以下blog的实验一：  
https://tech.meituan.com/2018/01/19/mybatis-cache.html  
### 可是我在spring项目中集成mybatis后，为何缓存失效？  
源码追踪了下，发现虽然是同一个线程，但每次执行访问数据库，都生成新的SqlSessionTemplate，既然生成sqlSession不同，自然缓存也失效了。  
### 为什么会每次都生成新的SqlSessionTemplate?
具体代码参考如下SqlSessionUtils.java：   
public static SqlSession getSqlSession(SqlSessionFactory sessionFactory, ExecutorType executorType,    PersistenceExceptionTranslator exceptionTranslator) {   
notNull(sessionFactory, NO_SQL_SESSION_FACTORY_SPECIFIED);   
    notNull(executorType, NO_EXECUTOR_TYPE_SPECIFIED);   
    SqlSessionHolder holder = (SqlSessionHolder) TransactionSynchronizationManager.getResource(sessionFactory);   
     // 首先从SqlSessionHolder里取出session   
    SqlSession session = sessionHolder(executorType, holder);   
    if (session != null) {   
      return session;   
    }   
    if (LOGGER.isDebugEnabled()) {   
      LOGGER.debug("Creating a new SqlSession");   
    }   
    session = sessionFactory.openSession(executorType);   
    registerSessionHolder(sessionFactory, executorType, exceptionTranslator, session);   
    return session;   
  }   

代码逻辑是，事务管理类TransactionSynchronizationManager中维护了与事务相关的资源sessionholder,sessionholder负责保存session，如果在不同事务中，事务的资源不同，取出来的session也不同，一级缓存能失效。  
### 加上事务控制，同一个事务内缓存就能生效？  
是的。同一个事务内取出来的session就是相同的，所以一级缓存能生效。  
**这样解释了当年在苏宁时期出现的一个问题：一个事务中，多次从DB中取出同一个sequence。**
当时的解释是事务未提交导致的。当时把业务拆分到不同事务，解决了问题。但其实只是找到解决办法，但没有找到真正原因。  
真正原因就是mybatis的一级缓存。



