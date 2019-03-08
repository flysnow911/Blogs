### 从Mybatis中sql执行流程能看出，sqlsession是sql执行的关键对象，并且他管理着数据访问的会话相关数据。既然与会话相关，那如何保证SqlSession的线程安全呢？
### 一般我们coding,最简单的线程安全方法是什么？
>ThreadLocal.java。

### 那Mybatis如何保证SQLSession的线程安全？
>答案：SqlSessionManager.java可以保证线程安全。
>line40: private final ThreadLocal<SqlSession> localSqlSession = new ThreadLocal<>();  
>line42: private SqlSessionManager(SqlSessionFactory sqlSessionFactory )
>看到这两行就秒懂了，首先SqlSessionManager是单例的，其次他利用threadLocal来管sqlSession。  
>但是，SqlSessionManager.java已经被废弃。现在生产sqlSession的类是DefaultSessionFactory生产defaultSession，但并不是线程安全
>的。所以，原生的Mybatis是不能提供线程安全的sqlsession的，只能自己动手。  

### 我们平时coding，并没有刻意去维护session的线程安全，是怎么回事呢？
>因为我们有spring-mybatis.jar的SqlSessionTemplate。
>localSqlSession包含一个私有内部类SqlSessionInteceptor.java，实现对SqlsessionFactory.java的动态代理，在invoke方法中生成提供>sqlsession的实例。SqlSessionTemplate.java由TransactionSyncnazationManager.java依据spring线程的上下文生成线程安全 
>sqlSession，放在事务同步管理类的资源中，并且由SqlSessionUtils通过SqlSession的引用计数来管理sqlSession，如session的打开和关闭操>作。这样做的好处是，SqlSessionTemplate有着更好的与spring的配合，并且管理管理sqlsessiong更高效。  

### 原生mybatis中,SqlSessionManager.java可以生产线程安全的sqlsession。为何spring不用？
>对比了SqlSessionManager和SqlSessionTemplate两个类，在管理sqlSession的功能中，两个类极其相似，都包含一个私有内部类>SqlSessionInteceptor.java，实现对SqlsessionFactory.java的动态代理，在invoke方法中生成提供sqlsession的实例。不同的是，>SqlSessionManager直接管理着成员变量localSqlSession， 而SqlSessionTemplate由则TransactionSyncnazationManager.java生成依据>spring线程的上下文线程安全的sqlSession，放在事务同步管理类的资源中，并且由SqlSessionUtils通过SqlSession的引用计数来管理>sqlSession，如session的打开和关闭操作。这样做的好处是，SqlSessionTemplate有着更好的与spring的配合，并且管理管理sqlsessiong更高>效。  
