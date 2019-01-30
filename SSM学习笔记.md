### SSM学习笔记

(1) 关于配置视图解析器

SpringMVC 视图解析器InternalResourceViewResolver：https://blog.csdn.net/sinat_35821285/article/details/79094925。理由是返回视图的真实路径和逻辑路径的区别。

(2) SpringMVC容器的配置文件applicationContext.xml就是配置各种bean，web.xml则用于配置web容器

(3) 关于容器的加载顺序及冲突避免：https://segmentfault.com/a/1190000012972619

(4) 关于@ResponseBody和@RequestBody

1. @ResponseBody是作用在方法上的，@ResponseBody 表示该方法的返回结果直接写入 HTTP response body 中，一般在异步获取数据时使用【也就是AJAX】，在使用 @RequestMapping后，返回值通常解析为跳转路径，但是加上 @ResponseBody 后返回结果不会被解析为跳转路径，而是直接写入 HTTP response body 中。 

   比如异步获取 json 数据，加上 @ResponseBody 后，会直接返回 json 数据。@RequestBody 将 HTTP 请求正文插入方法中，使用适合的 HttpMessageConverter 将请求体写入某个对象。

   在使用此注解之后不会再走试图处理器，而是直接将数据写入到输入流中，他的效果等同于通过response对象输出指定格式的数据

   @ResponseBody这个注解通常使用在控制层（controller）的方法上，其作用是将方法的返回值以特定的格式写入到response的body区域，进而将数据返回给客户端。当方法上面没有写ResponseBody,底层会将方法的返回值封装为ModelAndView对象。

   **假如是字符串则直接将字符串写到客户端,假如是一个对象，此时会将对象转化为json串然后写到客户端**。这里需要注意的是，**如果返回对象,按utf-8编码。如果返回String，默认按iso8859-1编码，页面可能出现乱码**。因此在注解中我们可以手动修改编码格式，例如@RequestMapping(value="/cat/query",produces="text/html;charset=utf-8")，前面是请求的路径，后面是编码格式。

   那么，控制层方法的返回值是如何转化为json格式的字符串的呢？其实是通过HttpMessageConverter中的方法实现的，因为它是一个接口，因此由其实现类完成转换。如果是**bean对象**，会调用对象的getXXX（）方法获取属性值并且以键值对的形式进行封装，进而转化为json串。如果是map集合，采用get(key)方式获取value值，然后进行封装。

   若返回是一个bean对象，也就是POJO对象，必须添加setter和getter方法

   ```java
   　　@RequestMapping("/login")
   　　@ResponseBody
   　　public User login(User user){
   　　　　return user;
   　　}
   　//　User字段：userName pwd
   　//　那么在前台接收到的数据为：'{"userName":"xxx","pwd":"xxx"}'
   
   　　// 效果等同于如下代码：
   　　@RequestMapping("/login")
   　　public void login(User user, HttpServletResponse response){
   　　　　response.getWriter.write(JSONObject.fromObject(user).toString());
   　　}
   ```

   原理：

   当一个处理请求的方法标记为@ResponseBody时，就说明该方法需要输出其他视图（json、xml），SpringMVC通过已定义的转化器做转化输出，默认输出json。

   其实是spring-mvc注解驱动帮忙做了这个事。

   ![img](https://images2017.cnblogs.com/blog/667897/201707/667897-20170729145804160-65046341.png)

   相关源码：

   ![img](https://images2017.cnblogs.com/blog/667897/201707/667897-20170729150018832-711523873.png)

   

   为此需要添加依赖：

   ![img](https://images2017.cnblogs.com/blog/667897/201707/667897-20170729143817300-1067172651.png)

   所以也就不需要在spring-mvc配置文件中显式配置：

   ```xml
   <!-- 用于将对象转换为 JSON  -->  
       <bean id="stringConverter"  
           class="org.springframework.http.converter.StringHttpMessageConverter">  
           <property name="supportedMediaTypes">  
               <list>  
                   <value>text/plain;charset=UTF-8</value>  
               </list>
           </property>
       </bean>  
    
   <bean id="jsonConverter"       class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter"/>  
   
   <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">  
           <property name="messageConverters">  
               <list>  
                   <ref bean="stringConverter" />
                   <ref bean="jsonConverter" />  
               </list>  
           </property>  
       </bean>
   ```

   

2. @RequestBody是作用在形参列表上，用于将前台发送过来固定格式的数据【xml 格式或者 json等】封装为对应的 JavaBean 对象，封装时使用到的一个对象是系统默认配置的 HttpMessageConverter进行解析，然后封装到形参上。一般是post请求的时候才会使用这个请求，把参数丢在requestbody里面。

(5) Spring-MVC 4.x 对跨域问题的支持

1. 注解

```java
@CrossOrigin("http://test.com")
@CrossOrigin(origins="http://test.com",maxAge=3600)
```

2. 手动配置Filter，在Response中添加对跨域的支持

```java
public class CORSFilter implements Filter {
    public void doFilter(ServletRequest req, ServletResponse res, 
    FilterChain chain) throws IOException, ServletException {
        HttpServletResponse response = (HttpServletResponse) res;
        response.setHeader("Access-Control-Allow-Origin", "*");
        response.setHeader("Access-Control-Allow-Methods", 
        	"POST, GET, OPTIONS, DELETE");
        response.setHeader("Access-Control-Max-Age", "3600");
        response.addHeader("Access-Control-Allow-Headers", 
        	"Origin, X-Requested-With, Content-Type, Accept");
        chain.doFilter(req, res);
    }
    public void init(FilterConfig filterConfig) {}
    public void destroy() {}
}

```

web.xml中

```xml
<filter>
    <filter-name>cors</filter-name>
    <filter-class>xxxx.CORSFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>cors</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

(6) 关于@RequestParam和@PathVariable

@RequestParam

使用@RequestParam接收前段参数比较方便，前端传参的URL：

>  url = “${ctx}/main/mm/am/edit?Id=${Id}&name=${name}”

后端使用集合来接受参数，灵活性较好，如果url中没有对参数赋key值，后端在接收时，会根据参数值的类型附，赋一个初始key（String、long ……）

> ```java
> @RequestMapping("/edit")
>     public String edit(Model model, @RequestParam Map<String, Object> paramMap ) {
>         long id = Long.parseLong(paramMap.get("id").toString());
>         String name = paramMap.get("name").toString;
>         return page("edit");
>     }
> ```

@PathVariable

使用@PathVariable接收参数，参数值需要在url进行占位，前端传参的URL：

> url = “${ctx}/main/mm/am/edit/${Id}/${name}”

```java
@RequestMapping("/edit/{id}/{name}")
    public String edit(Model model, @PathVariable long id,@PathVariable String name) {
        
        return page("edit");
    }
```

前端传参的URL于后端@RequestMapping的URL必须相同且参数位置一一对应，否则前端会找不到后端地址

(7) SpringMVC 的 RequestMethod.PUT 和 RequestMethod.DELETE

