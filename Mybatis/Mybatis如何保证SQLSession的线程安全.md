### ��Mybatis��sqlִ�������ܿ�����sqlsession��sqlִ�еĹؼ����󣬲��������������ݷ��ʵĻỰ������ݡ���Ȼ��Ự��أ�����α�֤SqlSession���̰߳�ȫ�أ�
### һ������coding,��򵥵��̰߳�ȫ������ʲô��
>ThreadLocal.java��

### ��Mybatis��α�֤SQLSession���̰߳�ȫ��
>�𰸣�SqlSessionManager.java���Ա�֤�̰߳�ȫ��
>line40: private final ThreadLocal<SqlSession> localSqlSession = new ThreadLocal<>();  
>line42: private SqlSessionManager(SqlSessionFactory sqlSessionFactory )
>���������о��붮�ˣ�����SqlSessionManager�ǵ����ģ����������threadLocal����sqlSession��  
>���ǣ�SqlSessionManager.java�Ѿ�����������������sqlSession������DefaultSessionFactory����defaultSession�����������̰߳�ȫ
>�ġ����ԣ�ԭ����Mybatis�ǲ����ṩ�̰߳�ȫ��sqlsession�ģ�ֻ���Լ����֡�  

### ����ƽʱcoding����û�п���ȥά��session���̰߳�ȫ������ô�����أ�
>��Ϊ������spring-mybatis.jar��SqlSessionTemplate��
>localSqlSession����һ��˽���ڲ���SqlSessionInteceptor.java��ʵ�ֶ�SqlsessionFactory.java�Ķ�̬������invoke�����������ṩ>sqlsession��ʵ����SqlSessionTemplate.java��TransactionSyncnazationManager.java����spring�̵߳������������̰߳�ȫ 
>sqlSession����������ͬ�����������Դ�У�������SqlSessionUtilsͨ��SqlSession�����ü���������sqlSession����session�Ĵ򿪺͹رղ�>�����������ĺô��ǣ�SqlSessionTemplate���Ÿ��õ���spring����ϣ����ҹ������sqlsessiong����Ч��  

### ԭ��mybatis��,SqlSessionManager.java���������̰߳�ȫ��sqlsession��Ϊ��spring���ã�
>�Ա���SqlSessionManager��SqlSessionTemplate�����࣬�ڹ���sqlSession�Ĺ����У������༫�����ƣ�������һ��˽���ڲ���>SqlSessionInteceptor.java��ʵ�ֶ�SqlsessionFactory.java�Ķ�̬������invoke�����������ṩsqlsession��ʵ������ͬ���ǣ�>SqlSessionManagerֱ�ӹ����ų�Ա����localSqlSession�� ��SqlSessionTemplate����TransactionSyncnazationManager.java��������>spring�̵߳��������̰߳�ȫ��sqlSession����������ͬ�����������Դ�У�������SqlSessionUtilsͨ��SqlSession�����ü���������>sqlSession����session�Ĵ򿪺͹رղ������������ĺô��ǣ�SqlSessionTemplate���Ÿ��õ���spring����ϣ����ҹ������sqlsessiong����>Ч��
