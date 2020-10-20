# 自定义错误页面和json

使用`@ControllerAdvice` 注解可以处理应用异常,但是一些容器级别的异常就处理不了,`@ControllerAdvice` 本身就是通过容器来处理的

## 简单的配置错误页面

springboot 可以根据用户的不同请求,智能的返回一段html 错误页面或者一段json,  springboot 中的错误默认是由`BasicErrorController`

源码:

```java
@Controller
@RequestMapping("${server.error.path:${error.path:/error}}")
public class BasicErrorController extends AbstractErrorController {

	private final ErrorProperties errorProperties;
    // ....
    
    @RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
	public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {
		HttpStatus status = getStatus(request);
		Map<String, Object> model = Collections
				.unmodifiableMap(getErrorAttributes(request, getErrorAttributeOptions(request, MediaType.TEXT_HTML)));
		response.setStatus(status.value());
		ModelAndView modelAndView = resolveErrorView(request, response, status, model);
		return (modelAndView != null) ? modelAndView : new ModelAndView("error", model);
	}

	@RequestMapping
	public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
		HttpStatus status = getStatus(request);
		if (status == HttpStatus.NO_CONTENT) {
			return new ResponseEntity<>(status);
		}
		Map<String, Object> body = getErrorAttributes(request, getErrorAttributeOptions(request, MediaType.ALL));
		return new ResponseEntity<>(body, status);
	}

```

相关配置位于 `ErrorProperties` 

其中两个 Mapping   一个 `errorHTMl` 是在 produces = MediaType.TEXT_HTML_VALUE (依赖于Accept请求头)访问页面的时候 , 另一个是其他请求的时候采用的路由.

errorHTML 请求通过 

```java
ModelAndView modelAndView = resolveErrorView(request, response, status, model);
```

获取视图  `resolveErrorView` 又通过 `ErrorViewResolver`类获取视图 , 通过 在`ErrorMvcAutoConfiguration` 类中的 `DefaultErrorViewResolverConfiguration` 静态类 发现 注入到容器中的是 `DefaultErrorViewResolver` 类  

来到 `DefaultErrorViewResolver` 类 

```java
public class DefaultErrorViewResolver implements ErrorViewResolver, Ordered {

	private static final Map<Series, String> SERIES_VIEWS;

	static {
		Map<Series, String> views = new EnumMap<>(Series.class);
		views.put(Series.CLIENT_ERROR, "4xx");
		views.put(Series.SERVER_ERROR, "5xx");
		SERIES_VIEWS = Collections.unmodifiableMap(views);
	}
	
	//....
    
    private ModelAndView resolve(String viewName, Map<String, Object> model) {
		String errorViewName = "error/" + viewName;
		TemplateAvailabilityProvider provider = this.templateAvailabilityProviders.getProvider(errorViewName,
				this.applicationContext);
		if (provider != null) {
			return new ModelAndView(errorViewName, model);
		}
		return resolveResource(errorViewName, model);
	}
	
	@Override
	public ModelAndView resolveErrorView(HttpServletRequest request, HttpStatus status, Map<String, Object> model) {
		ModelAndView modelAndView = resolve(String.valueOf(status.value()), model);
		if (modelAndView == null && SERIES_VIEWS.containsKey(status.series())) {
			modelAndView = resolve(SERIES_VIEWS.get(status.series()), model);
		}
		return modelAndView;
	}
	
```

其在 error 路径下面查找 4xx , 5xx 文件作为错误视图,所以要简单配置的只需要在static文件夹下创建error文件夹在创建对应的html文件 

>   默认的  4xx.html  和 404.html 会最大程度匹配

### 动态的错误页面

在 templates  下创建 error 文件夹在创建对应的html文件 使用模板引擎渲染页面即可

在模板文件中可以用到的error信息有 `timestamp` , `status` , `error` , `message` , `path`  



## 复杂配置

简单配置只能对 HTML 页面进行配置 , 想要对错误JSON 进行配置 ,就要从三方面进行

### 自定义error数据   `ErrorAttributes` 

上文说到 error信息有 `timestamp` , `status` , `error` , `message` , `path`  这些信息都是通过 

```java
getErrorAttributes(request, getErrorAttributeOptions(request, MediaType.TEXT_HTML));
```

方法进行获取的,在其中使用了 `DefaultErrorAttributes` 类获取error数据

通过 `ErrorMvcAutoConfiguration` 配置类知道在没有 `ErrorAttributes `类的之后才会注入 `DefaultErrorAttributes`

```java
@Bean
@ConditionalOnMissingBean(value = ErrorAttributes.class, search = SearchStrategy.CURRENT)
public DefaultErrorAttributes errorAttributes() {
   return new DefaultErrorAttributes();
}
```

所以只需要我们自己提供一个 `ErrorAttributes` 类 即可 ,可以继承 `DefaultErrorAttributes`类 或者 `ErrorAttributes`接口

 `DefaultErrorAttributes`: `getErrorAttributes方法`源码:

```java
@Override
	public Map<String, Object> getErrorAttributes(WebRequest webRequest, ErrorAttributeOptions options) {
		Map<String, Object> errorAttributes = getErrorAttributes(webRequest, options.isIncluded(Include.STACK_TRACE));
		if (this.includeException != null) {
			options = options.including(Include.EXCEPTION);
		}
		if (!options.isIncluded(Include.EXCEPTION)) {
			errorAttributes.remove("exception");
		}
		if (!options.isIncluded(Include.STACK_TRACE)) {
			errorAttributes.remove("trace");
		}
		if (!options.isIncluded(Include.MESSAGE) && errorAttributes.get("message") != null) {
			errorAttributes.put("message", "");
		}
		if (!options.isIncluded(Include.BINDING_ERRORS)) {
			errorAttributes.remove("errors");
		}
		return errorAttributes;
	}

@Override
	@Deprecated
	public Map<String, Object> getErrorAttributes(WebRequest webRequest, boolean includeStackTrace) {
		Map<String, Object> errorAttributes = new LinkedHashMap<>();
		errorAttributes.put("timestamp", new Date());
		addStatus(errorAttributes, webRequest);
		addErrorDetails(errorAttributes, webRequest, includeStackTrace);
		addPath(errorAttributes, webRequest);
		return errorAttributes;
	}
```

继承并实现这两个方法注入到容器中即可

```java
public class MyErrorAttributes extends DefaultErrorAttributes {

    @Override
    public Map<String, Object> getErrorAttributes(WebRequest webRequest, ErrorAttributeOptions options) {
        Map<String, Object> map = super.getErrorAttributes(webRequest, options);

        map.put("mymessage","出错了");
        map.remove("error");
        return map;
    }
}
```

测验:

![image-20200810113712190](%E8%87%AA%E5%AE%9A%E4%B9%89%E9%94%99%E8%AF%AF%E9%A1%B5%E9%9D%A2%E5%92%8Cjson.assets/image-20200810113712190.png)

使用postman:

![image-20200810113841763](%E8%87%AA%E5%AE%9A%E4%B9%89%E9%94%99%E8%AF%AF%E9%A1%B5%E9%9D%A2%E5%92%8Cjson.assets/image-20200810113841763.png)



### 自定义error视图  `errorViewResolvers`

如 自定义error数据 自定义error 视图 是通过继承 `errorViewResolvers` 接口来实现的 spring boot 默认提供

`DefaultErrorViewResolvers` 类 ,当用户没有注入 `errorViewResolvers` 接口的时候提供默认实现

所以直接继承`DefaultErrorViewResolvers`  或者 `errorViewResolvers`  接口  重写resolveErrorView 

### 完全自定义 `BasicErrorController` 

BasicErrorController 使用  `ErrorAttributes`  和 `errorViewResolvers` 提供视图和数据 ,我们也可以提供自己的 `ErrorController` 类

```java
@Controller
public class MyErrorController extends BasicErrorController {

    @Autowired
    public MyErrorController(ErrorAttributes errorAttributes, ErrorProperties errorProperties, List<ErrorViewResolver> errorViewResolvers) {
        super(errorAttributes, errorProperties, errorViewResolvers);
    }

    @Override
    public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {
        HttpStatus httpStatus = getStatus(request);
        Map<String, Object> model = getErrorAttributes(request, ErrorAttributeOptions.defaults());
        model.put("key","自定义错误信息");
        //指定视图
        ModelAndView mv = new ModelAndView("errorpage",model,httpStatus);
        return mv;
    }

    @Override
    public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
        Map<String, Object> model = getErrorAttributes(request, ErrorAttributeOptions.defaults());
        model.put("key","自定义错误信息");
        HttpStatus status = getStatus(request);
        return new ResponseEntity<>(model,status);
    }
}

```

`MyErrorController` 类

```java
@Controller
public class MyErrorController extends BasicErrorController {


    @Autowired
    public MyErrorController(ErrorAttributes errorAttributes,
                             ServerProperties serverProperties,
                             List<ErrorViewResolver> errorViewResolvers) {
            super(errorAttributes,serverProperties.getError(),errorViewResolvers);
    }


    @Override
    public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {
        HttpStatus httpStatus = getStatus(request);
        Map<String, Object> model = getErrorAttributes(request, ErrorAttributeOptions.defaults());
        model.put("key","自定义错误信息");
        //指定视图   寻找 errorpage.html 作为错误视图
        ModelAndView mv = new ModelAndView("errorpage",model,httpStatus);
        return mv;
    }

    @Override
    public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
        Map<String, Object> model = getErrorAttributes(request, ErrorAttributeOptions.defaults());
        model.put("key","自定义错误信息");
        HttpStatus status = getStatus(request);
        return new ResponseEntity<>(model,status);
    }
}

```

<span style="color:red">注意</span> 因为BasicErrorController 没有无参构造函数 ,而且BasicErrorController的有参构造函数需要 ErrorProperties 对象没有在容器中,所以我们参考 `ErrorMvcAutoConfiguration` 类中的 basicErrorController 方法,新增一个有参构造函数并注解@autowired 

