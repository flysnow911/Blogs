// 方法1：获取文件或流
this.getClass().getResource("/")+fileName;
this.getClass().getResourceAsStream(failName);
// 方法2：获取文件
File file = org.springframework.util.ResourceUtils.getFile("classpath:test.txt");
// 方法3：获取文件或流
ClassPathResource classPathResource = new ClassPathResource("test.txt");
classPathResource .getFile();
classPathResource .getInputStream();

// >>>>>>>>>>>>>>>> 下面方法可以读取jar包下文件

// 方法1

InputStream io = Thread.currentThread().getContextClassLoader().getResourceAsStream("test.txt");

// 方法2
InputStream io = ClassLoader.getSystemResourceAsStream("test.txt");
