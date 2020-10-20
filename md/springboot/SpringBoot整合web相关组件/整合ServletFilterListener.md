# Servlet Filter Listener

在SpringBoot 或者 SpringMVC中基本不使用 Servlet  Filter  Listener 但是某些请胯下如果需要使用

需要 @WebServlet  @WebFilter  @WebListener 

例如

```java
@WebServlet("/myservlet")
class MyServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doPost(req, resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("MyServlet  >> doPost..." + req.getParameter("name"));
    }
}


@WebFilter("/*")
class MyFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

        System.out.println("MyFilter  >>  init...");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        System.out.println("MyFilter  >> doFilter");
        chain.doFilter(request, response);

    }

    @Override
    public void destroy() {
        System.out.println("MyFilter  >> destroy...");
    }
}

@WebListener
class MyListener implements ServletRequestListener { //支持其他Listener

    @Override
    public void requestDestroyed(ServletRequestEvent sre) {
        System.out.println("MyListener >> requestDestroyed...");
    }

    @Override
    public void requestInitialized(ServletRequestEvent sre) {
        System.out.println("MyListener  >>  requestInitialized...");
    }
}
```

然后再启动类上使用  `@ServletComponentScan` 扫描到组件即可



上述例子在访问`http://localhost:8080/myservlet?name=123` 时的执行顺序

```
MyListener  >>  requestInitialized...
MyFilter  >> doFilter
MyServlet  >> doPost...123
MyListener >> requestDestroyed...

```

