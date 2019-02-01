# Servlet 工作原理

(1) redirect 与 forware

1. redirect

response.sendRedirect("/a.jsp")；页面的路径是相对路径。sendRedirect**可以将页面跳转到任何页面**，不一定局限于本web应用中，如：response.sendRedirect("http://www.ycul.com")；**跳转后浏览器地址栏变化**, 这种方式要传值出去的话，只能在url中带parameter或者放在session中，无法使用request.setAttribute来传递 。

这种方式是**在客户端作的重定向处理**。该方法通过修改HTTP协议的HEADER部分,对浏览器下达重定向指令的，让浏览器对在location中指定的URL提出请求，使浏览器显示重定向网页的内容。该方法可以接受绝对的或相对的URLs，如果传递到该方法的参数是一个相对的URL，那么Webcontainer在将它发送到客户端前会把它转换成一个绝对的URL。

```java
public voiddoPost(HttpServletRequest request,HttpServletResponseresponse)  throws ServletException,IOException
{
       response.setContentType("text/html; charset=UTF-8");
       response.sendRedirect("/index.jsp");
}
```

2. forware

```java
RequestDispatcher dispatcher =request.getRequestDispatcher("/a.jsp");
dispatcher .forward(request, response);
```

页面的路径是相对路径。forward方式只能跳转到本web应用中的页面上。**跳转后浏览器地址栏不会变化**。
使用这种方式跳转，传值可以使用三种方法：url中带parameter，session，request.setAttribute。

区别：

1. forward重定向是在容器内部实现的同一个Web应用程序的重定向，所以forward方法只能重定向到同一个Web应用程序中的一个资源，重定向后浏览器地址栏URL不变；而sendRedirect方法可以重定向到任何URL，因为这种方法是**修改http头部信息来实现**的，URL没什么限制，重定向后浏览器地址栏URL改变。

2. forward重定向将原始的HTTP请求对象（request）从一个servlet实例传递到另一个实例，而采用sendRedirect方式两者不是同一个application。

3. 基于第二点，参数的传递方式不一样。forward的form参数跟着传递，所以在第二个实例中可以取得HTTP请求的参数。sendRedirect只能通过链接传递参数，response.sendRedirect(“login.jsp?param1=a”)。

4. sendRedirect能够处理相对URL，自动把它们转换成绝对URL，如果地址是相对的，没有一个‘/’，那么Webcontainer就认为它是相对于当前的请求URI的。

   比如，如果为response.sendRedirect("login.jsp")，则会从当前servlet 的URL路径下找login.jsp：http://10.1.18.8:8081/dms/servlet/Servlet 重定向的URL:http://10.1.18.8:8081/dms/servlet/login.jsp。

   如果为response.sendRedirect("/login.jsp")则会从当前应用径下查找[url:http://10.1.18.8:8081/login.jsp](http://10.1.18.8:8081/login.jsp)。而forward不能这样处理相对路径。

5. response.sendRedirect是向**客户端浏览器**发送页面重定向指令，浏览器接收后将向web服务器重新发送页面请求，所以执行完后浏览器的url显示的是跳转后的页面。跳转页面可以是一个任意的url（本服务器的和其他服务器的均可）。

   RequestDispatcher.forward则是直接在**服务器**中进行处理，将**处理完**后的信息发送给浏览器进行显示,所以完成后在url中显示的是跳转前的页面。在forward的时候将上一页面中传送的request和response信息一同发送给下一页面（而response.sendRedirect不能将上一页面的request和response信息发送到下一页面）。由于forward是直接在服务器中进行处理，所以forward的页面只能是本服务器的。

