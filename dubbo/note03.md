JDK SPI的实现机制：
服务方定义接口，例如：com.test.Log
第三方jar可以在自己的 META-INF/services/目录下添加接口实现的类
|-META-INF/services
    |-com.test.Log(文件),内容为:
        |-com.impl.MyLog1
        |-com.impl.MyLog2

在加载某个接口的SPI时，遍历文件的所有内容行，使用类加载器去加载对应类，然后实例化
ps:在JDBC中 第三方jar会将接口实现的对象的实例化放到静态代码块中，这样在SPI加载的时候会触发代码块，从而触发实例化
例如：
```java
String url = "jdbc:xxx://xxx:xxx/xxx"; 
Connection conn = DriverManager.getConnection(url, username, pwd);
//调用DriverManager这个类的静态方法getConnection时，会触发DriverManager的类加载
```

````java
/**
 *调用DriverManager这个类的中有静态代码块，加载的时候被触发loadInitialDrivers，这个loadInitialDrivers会使用JDK的SPI加载对应的
 * Driver.
 */
static {
    loadInitialDrivers();
    println("JDBC DriverManager initialized");
}

/**
 * 而对应的Driver的实现类中又会存在这样的代码块,当他们被SPI的类加载时，他们会通过这块代码往DriverManager中注册自己的Driver实例
 */
static {
    java.sql.DriverManager.registerDriver(new Driver());
}

````
