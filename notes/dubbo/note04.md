Dubbo SPI的实现
# 扩展点：
- SPI配置文件改为kev=value格式，key为改实现类的扩展名
- 支持多分区不同策略加载：
  - |-META-INF/dubbo/internal/ 内部的SPI
  - |-META-INF/dubbo/ dubbo用户spi
  - |-META-INF/services/ JDK原生支持的spi
  
# SPI:如何加载的扩展类
ExtensionLoader.getExtension()时,查看是否已经加载过，没有的话就createExtension并缓存。
创建的过程包
- 扩展类的类加载，查看是否已经缓存过这个类的class对象
  - 有：说明已经加载过了，直接返回
  - 没有：触发SPI加载机制：首先根据策略去不同META-INF路径下查找配置文件并加载，加载之后会判断
    - 是否有@Adpative注解，如果有说明这是一个适配类，需要在适配类缓存字段中添加该类。
    - 是否是一个包装类，如果是需要在包装类缓存中添加该类。
    - 以上都不是的话，则开始缓存这个扩展类的相关信息
      - 如果这个类使用了@Activate注解，则缓存这个类的name和注解信息的映射关系。
      - 在缓存这个扩展名和扩展类的class文件的映射关系
      - 在缓存这个扩展类的class文件和扩展名的映射关系（和上述反过来）

# @Adaptive 适配器：适配器怎么工作
适配器是一种特殊的扩展类，有一个属性value:String[].
适配器类会动态生成一个类实现，里面会接续URL里面的参数，找到注解中value对应key的value，这个value就是实际执行逻辑的扩展类的扩展名```
```java
public class Transporter$Adaptive implements Transporter { 

    public org.apache.dubbo.remoting.Client connect(URL arg0, ChannelHandler arg1) throws RemotingException { 

        // 必须传递URL参数 
        if (arg0 == null) throw new IllegalArgumentException("url == null"); 

        URL url = arg0; 
        //假设@Adaptive注解是@Adaptive(value = "client,transporter") 
        // SPI接口的default是@SPI(value = "netty")
        // 优先从URL中的client参数获取，其次是transporter参数,最后是@SPI注解中的默认值 
        String extName = url.getParameter("client",url.getParameter("transporter", "netty")); 
        if (extName == null) 
            throw new IllegalStateException("..."); 
        // 通过ExtensionLoader加载Transporter接口的指定扩展实现 
        Transporter extension = (Transporter) ExtensionLoader .getExtensionLoader(Transporter.class) .getExtension(extName); 
        然后调用实际的connect()方法
        return extension.connect(arg0, arg1); 

    } 
}
```
# 自动包装特性
在扩展类加载的时候，因为判断了该类是不是包装类，同时会把所有包装类对应的class文件缓存下来。
后续创建一个真正的扩展类时，会自动把所有包装类都包装上。
包装类的格式:
```java
public class ProtocolListenerWrapper implements Protocol {
    private final Protocol protocol;
    public ProtocolListenerWrapper(Protocol protocol) {
        this.protocol = protocol;
    }
}
```

# 自动装配特性
扩展类实例化成功之后，会检测他的属性是不是也是实现了某个SPI接口，如果是的话，然后实例化并注入进去。
获取扩展类实例的方式有多种 ExtensionFactory
- SpiExtensionFactory ：获取扩展类的ExtensionLoader 然后获取对应的适配类 loader.getAdaptiveExtension()
- 通过spring容器获取

# @Activate注解与自动激活特性
@Activate可以在扩展类上使用，用来控制该扩展类在什么时候生效举个例子：
- @Activate(value = "name") 表示URL只要有name这个参数就能生效
- @Activate(value = "name:zs") 表示URL中需要有name并且值为zs才能生效
如何实现?
假设一个url为：test://localhost/test?ext=order1,default，可以通过调用来获取这个接口可用的扩展类
```
    getActivateExtension(url, "ext", "default_group");
```
因为接口在SPI加载的时候，已经缓存了所有的加了@Activate注解的类class。所以在getActivateExtension方法会获取到ext参数的值:"order1,default"
该值表示扩展名为order1的扩展类顺序第一,default在后面，default表示的就是排除用户已经指定的扩展类之外，剩下的其余所有扩展类
```java
    /**
     *举个例子: "test:localhost?log=D,F,default,A"
     *实现类有[A,B,C,D,E,F] 用户url上配置的是[D,F,default,A]
     *第一个循环会找出default对应的值有哪些：[B,C,E]
     *第二个循环会遍历用户配置的，依次加入D->F->[B->C->E] ->A
     */
```