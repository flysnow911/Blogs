Mybatis算是简单而非常重要的开源框架了。话不多说，先来看下代码量。   
[![Source count](https://github.com/flysnow911/Blogs/blob/master/Mybatis/images/sourcecount.png "Source count")](https://github.com/flysnow911/Blogs/blob/master/Mybatis/images/sourcecount.png "Source count")
Java代码也就2.1w。算是个小工程了。   
再看下结构，平铺了20个packages,也就是有20个模块。从packages名字能看出，各个模块是什么功能，具体就不说了。   
[![struct](https://github.com/flysnow911/Blogs/blob/master/Mybatis/images/Struct.png "struct")](https://github.com/flysnow911/Blogs/blob/master/Mybatis/images/Struct.png "struct")   

大概说一下工程各模块！
##### 反射模块 reflection  
Mybatis有大量的类元数据的配置，依赖这些元数据创建类的实例，是这个模块的功能。这个模块对原生java反射进行了封装，向上提供了很多简洁高性能的API。平时工作也可以对这个api实行拿来主义，直接copy过来用。
[![reflection](https://github.com/flysnow911/Blogs/blob/master/Mybatis/images/reflection.png "reflection")](https://github.com/flysnow911/Blogs/blob/master/Mybatis/images/reflection.png "reflection")   

##### Type 模块  
(https://github.com/flysnow911/Blogs/blob/master/Mybatis/images/type.png "type")](https://github.com/flysnow911/Blogs/blob/master/Mybatis/images/type.png "type")      
该模块定义了若干个TypeHandler，主要功能是java数据类型和jdbc数据类型相互转换。  
给SQL设置参数时，java -> jdbc。  
映射结果集时，jdbc->java。  

##### 日志模块   Logging
主要功能是集成外部日志框架。   

##### IO模块  IO
资源加载模块，主要是对类加载器进行封装，确定类加载器的使用顺序，并提供了加载类文件以及其他资源文件的功能 。   

##### 解析器模块 parsing   
封装xpath解析代码，提供两个功能：
1. 解析xml文件，如：mapper.xml, mybatis-config.xml等。
2. 动态sql占位符支持。    

##### 数据源模块datasource    
封装了自身数据源的实现和第三方数据源实现的接口。

##### transaction模块   
提供简单的事务控制的接口和实现，但因为一般都是和spring集成，用了spring的事务。   

##### 缓存cache模块    
提供缓存接口和不同策略的缓存实现。   

##### Binding模块   
用户通过bingding模块，实现dao定义的方法和配置文件中sql的绑定。











