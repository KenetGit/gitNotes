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

   

2. @RequestBody是作用在形参列表上，用于将前台发送过来固定格式的数据【xml 格式或者 json等】封装为对应的 JavaBean 对象，封装时使用到的一个对象是系统默认配置的 HttpMessageConverter进行解析，然后封装到形参上。（理解转换器如何将一个字符串转换为一个Java对象）

   @requestBody注解常用来处理content-type不是默认的application/x-www-form-urlcoded编码的内容，比如说：application/json或者是application/xml等。一般情况下来说常用其来处理application/json类型。

   通过@requestBody可以将**请求体中的JSON字符串**绑定到相应的bean上，当然，也可以将其分别绑定到对应的字符串上。

     例如说以下情况：

   ```javascript
   　　　　$.ajax({
   　　　　　　　　url:"/login",
   　　　　　　　　type:"POST",
   　　　　　　　　data:'{"userName":"admin","pwd","admin123"}',
   　　　　　　　　content-type:"application/json charset=utf-8",
   　　　　　　　　success:function(data){
   　　　　　　　　　　alert("request success ! ");
   　　　　　　　　}
   　　　　});
   
   ```

   ```java
   @requestMapping("/login")
   public void login(@requestBody String userName,@requestBody String pwd){
   　　　System.out.println(userName+" ："+pwd);
   }  
   ```

   这种情况是将JSON字符串中的两个变量的值分别赋予了两个字符串，但是呢假如我有一个User类，拥有如下字段：

   ```java
   String userName;
   String pwd;
   ```


   那么上述参数可以改为以下形式：@requestBody User user 这种形式会将JSON字符串中的值赋予user中对应的属性上。需要注意的是，**JSON字符串中的key必须对应user中的属性名，否则是转换不了的**。

   在一些特殊情况@requestBody也可以用来处理content-type类型为application/x-www-form-urlcoded的内容，只不过这种方式不是很常用，在处理这类请求的时候，**@requestBody会将处理结果放到一个MultiValueMap<String,String>中**，这种情况一般在特殊情况下才会使用，例如jQuery easyUI的datagrid请求数据的时候需要使用到这种方式、小型项目只创建一个POJO类的话也可以使用这种接受方式。

3. 总结

   (1) @ResponseBody

   该注解用于将Controller的方法返回的对象，通过适当的HttpMessageConverter转换为指定格式后，写**入到Response对象的body数据区**。配合@ResponseBody注解，以及**HTTP Request Header中的Accept属性**，Controller返回的Java对象可以自动被转换成对应的XML或者JSON数据。

   这个过程是通过HttpMessageConverter即消息转换器机制实现的。实现对象转xml的类为Jaxb2RootElementHttpMessageConverter，转换成功的条件：

   a) 返回对象的类具有XmlRootElement注解。
   b) 请求头中的Accept属性包含application/xml。

   **对象转换成json数据时需要把Jackson2或者GSON加入工程的class path**，Spring就会**自动**把GsonHttpMessageConverter加进来，这样Spring就会选择MappingJackson2HttpMessageConverter或者GsonHttpMessageConverter来进行数据转换。

   (2) @RequestBody

   该注解用于读取Request请求的body部分数据，使用系统默认配置的HttpMessageConverter进行解析，然后把相应的数据绑定到要返回的对象上 ,再把HttpMessageConverter返回的对象数据绑定到 controller中方法的参数上。配合@RequestBody注解，以及HTTP Request Header中的Content-Type属性，HTTP Request Body中包含的XML或者JSON数据可以自动被转换成对应的Java对象。

   原理参考：https://blog.csdn.net/geyuezhen/article/details/52853675

4. <mvc:annotation-driven />有三个可选配置项

```xml
<mvc:annotation-driven  message-codes-resolver ="bean ref" 
	validator="" conversion-service="">
 
     <mvc:return-value-handlers>
        <bean></bean>
    </mvc:return-value-handlers>
 
 	<!--允许注册实现了HandlerMethodArgumentResolver接口的bean，
		来对handlerMethod中的用户自定义的参数或annotation进行解析 -->
    <mvc:argument-resolvers>

    </mvc:argument-resolvers>
 
    <!--HttpMessageConverter主要是用来转换request的内容到一定的格式，
		转换输出的内容的到response。即：controller方法返回的类型 -->
    <!--MappingJacksonHttpMessageConverter是其中一个默认的配置 -->
    <mvc:message-converters>
 
    </mvc:message-converters>
 
</mvc:annotation-driven>
```



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

