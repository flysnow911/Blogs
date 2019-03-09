InvocationHandler 怎么老是你？   
今日小擼了下mybatis的插件功能，本着正本清源的精神，看了下plugin的源码，又看到了InvocationHandler，看来plugin功能也是站在了InvocationHandler的肩膀上！   
先说下Plugin的开发吧。 把大象装进冰箱有三步骤，开发Plugin也只有三步骤：   
1.编写自己的plugin类，实现Mybatis的Interceptor接口。   
2.复写intercept,plugin,setProperties方法，加上@Intercepts @Signature注解，告诉mybatis需要拦截那些类，那些方法。   
3.在mybatis-config.xml注册自己的plugin。   

好了，开始正本清源了。 XMLConfigBuilder.java line:110， 解析xml中 注册的plugins
`pluginElement(root.evalNode("plugins"));`

line:183,就是具体解析plugins
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

生成注册的interceptor实例，加入confgiuration的intercetorChain中。 这就完成了自定义插件的注册和实例化。
在intercetorChain中，生成所有这些动态代理的实例：
`

    public class InterceptorChain {
      private final List<Interceptor> interceptors = new ArrayList<>();
      public Object pluginAll(Object target) {
        for (Interceptor interceptor : interceptors) {
          target = interceptor.plugin(target);
        }
        return target;
      }`   
注意上面的方法中，熟悉的**责任链**，interceptor.plugin(target);是给target生成一个代理，循环的调用interceptor.plugin()的话，就是给target生成多个动态代理代理！如果我们创建两个plugin,就是给target创建两层的代理，也就是拦截了两层！ 
要拦截N次，就拦截N次！
[![插件拦截原理](https://github.com/flysnow911/Blogs/blob/master/Mybatis/images/plugins.png "插件拦截原理")](https://github.com/flysnow911/Blogs/blob/master/Mybatis/images/plugins.png "插件拦截原理")

再看下自定义插件覆写的几个方法：

	Object intercept(Invocation invocation)  //这里面写插件具体要干啥，比如“收个保护费”，“打印个sql”。    
	Object plugin(Object target);     // 这里面写的是何时触发intercept，这是核心！    
	void setProperties(Properteis p)  // 这个把配置的properties设置到插件实例中。
先来分析下plugin方法，一般的实现如下： 
	
	@Override   
	public Object plugin(Object target) {   
		return Plugin.wrap(target, this);   
	}

看下Plugin.wrap方法，就懂了一切：

`public class Plugin implements InvocationHandler { //实现INvocationHandler接口! `

覆写invoke方法!
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
用Proxy.newInstance生成代理实例！


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
	
原来还是动态代理那一套啊!!  
