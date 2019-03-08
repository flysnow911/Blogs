InvocationHandler ��ô�����㣿   
����С�]����mybatis�Ĳ�����ܣ�����������Դ�ľ��񣬿�����plugin��Դ�룬�ֿ�����InvocationHandler������plugin����Ҳ��վ����InvocationHandler�ļ���ϣ�   
��˵��Plugin�Ŀ����ɡ� �Ѵ���װ�������������裬����PluginҲֻ�������裺   
1.��д�Լ���plugin�࣬ʵ��Mybatis��Interceptor�ӿڡ�   
2.��дintercept,plugin,setProperties����������@Intercepts @Signatureע�⣬����mybatis��Ҫ������Щ�࣬��Щ������   
3.��mybatis-config.xmlע���Լ���plugin��   

���ˣ���ʼ������Դ�ˡ� XMLConfigBuilder.java line:110�� ����xml�� ע���plugins
`pluginElement(root.evalNode("plugins"));`

line:183,���Ǿ������plugins
`     
	
	private void pluginElement(XNode parent) throws Exception {
   	 if (parent != null) {   
     	 for (XNode child : parent.getChildren()) {   
        		String interceptor = child.getStringAttribute("interceptor");   
        		Properties properties = child.getChildrenAsProperties();   
        		Interceptor interceptorInstance =    
					(Interceptor)  resolveClass(interceptor).newInstance();     
       			 interceptorInstance.setProperties(properties);    
        		configuration.addInterceptor(interceptorInstance);   
     	 }
    }  

����ע���interceptorʵ��������confgiuration��intercetorChain�С� ���������Զ�������ע���ʵ������
��intercetorChain�У�����������Щ��̬�����ʵ����
`

    public class InterceptorChain {
      private final List<Interceptor> interceptors = new ArrayList<>();
      public Object pluginAll(Object target) {
        for (Interceptor interceptor : interceptors) {
          target = interceptor.plugin(target);
        }
        return target;
      }`   
ע������ķ����У���Ϥ��**������**��interceptor.plugin(target);�Ǹ�target����һ������ѭ���ĵ���interceptor.plugin()�Ļ������Ǹ�target���ɶ����̬�������������Ǵ�������plugin,���Ǹ�target��������Ĵ���Ҳ�������������㣡 


�ٿ����Զ�������д�ļ���������

	Object intercept(Invocation invocation)  //������д�������Ҫ��ɶ�����硰�ո������ѡ�������ӡ��sql����    
	Object plugin(Object target);     // ������д���Ǻ�ʱ����intercept�����Ǻ��ģ�    
	void setProperties(Properteis p)  // ��������õ�properties���õ����ʵ���С�
����������plugin������һ���ʵ�����£� 
	
	@Override   
	public Object plugin(Object target) {   
		return Plugin.wrap(target, this);   
	}

����Plugin.wrap�������Ͷ���һ�У�

`public class Plugin implements InvocationHandler { //ʵ��INvocationHandler�ӿ�! `

��дinvoke����!
`   

	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {  
		try {    
 			 Set<Method> methods = signatureMap.get(method.getDeclaringClass());   
		  if (methods != null && methods.contains(method)) {   
			return interceptor.intercept(new Invocation(target, method, args));   
		  }   
		  return method.invoke(target, args);  
		  } catch (Exception e) {  
		  throw ExceptionUtil.unwrapThrowable(e);  }
		  }`
��Proxy.newInstance���ɴ���ʵ����


	public static Object wrap(Object target, Interceptor interceptor) {
		Map<Class<?>, Set<Method>> signatureMap = getSignatureMap(interceptor);   
		Class<?> type = target.getClass();  
		Class<?>[] interfaces = getAllInterfaces(type, signatureMap);  
		if (interfaces.length > 0) {   
 	 		return Proxy.newProxyInstance(  
  	    	type.getClassLoader(),  
	 	     interfaces,  
		      new Plugin(target, interceptor, signatureMap));  
		}   
	return target;}   
	
ԭ�����Ƕ�̬������һ�װ�!! 