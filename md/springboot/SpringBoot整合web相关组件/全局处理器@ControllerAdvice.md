# 全局处理器@ControllerAdvice

@ControllerAdvice 就是@Controller 的增强版   @ControllerAdvice 主要用来处理全局数据,一般搭配@ExceptionHandler , @ModelAttribute , @InitBinder 使用

## @ExceptionHandler 全局异常处理

`@ControllerAdvice` 结合 `@ExceptionHandler` 可以做到全局异常捕获代码如下

```java
@ControllerAdvice
@ResponseBody
public class TestAdviceController {


    @ExceptionHandler(Exception.class)
    @ResponseBody
    public Object testException(Exception e,
                                      HttpServletRequest req,
                                      HttpServletResponse rep) throws IOException {
//        rep.setContentType("text/json;charset=utf-8");
//        PrintWriter w = rep.getWriter();
//        w.write("{message:}");
        Map<String,String> map = new HashMap<>();
        map.put("message",e.getMessage());
        return map;
    }
```

controller

```java
@RequestMapping("/hello")
    public Object hello() throws Exception {

        throw new Exception("错误提示信息");
        //return "aaaaaa";
    }
```

@ExceptionHandler 标注的方法参数可以有   `HttpServletRequest` `HttpServletResponse ` `Exception` `Model` `ModelAndView`   返回值可以是 `JSON`   `ModelAndView`   `视图逻辑名` 等

##  @ModelAttribute(value="key")  全局model 

 @ModelAttribute 可以将任意参数绑定到所有@Controller 方法中的Model 对象中,在Controller 中可以使用他们

@ModelAttribute  中的value 属性就是Model 中属性的key  而方法返回值就是Model key 所对应的值

```java
@ControllerAdvice
public class TestAdviceController {
	@ModelAttribute("info")
    public Map<String,String> testModelAttribute(){
        Map<String,String> map = new HashMap<>();
        map.put("全局数据1","全局值1");
        map.put("全局数据2","全局值2");
        map.put("全局数据3","全局值3");
        return map;
    }
```

controller:

```java
@RequestMapping("/modelattribute")
    public ModelAndView ModelAttribute(ModelAndView mv, Model model) {

        Map<String, Object> mm = model.asMap();

        Iterator<String> itor = mm.keySet().iterator();
        while (itor.hasNext()){
            String key = itor.next();
            Object val = mm.get(key);
            System.out.println(key+" : "+val);
        }
        mv.setViewName("ModelAttribute");
        //return "aaaaaa";
        return mv;
    }
```

>   <span style="color:red" >注意</span> ModelAndView 和 model 的区别在   ModelAndView  最终会储存在model内

模板:

```html
<!DOCTYPE html>
<html lang="ch" xmlns:th="http://www.thymeleaf.org" >
<head>
    <meta charset="UTF-8">
    <title>ModelAttribute</title>
</head>
<body>

<ul>
    <li th:each="obj,iterStat:${info}">
        <span th:text="'iterStat.index'+${iterStat.index}"></span>
        <span th:text="'key'+${obj.key}"></span>
        <span th:text="'value'+${obj.value}"></span>
    </li>
</ul>
</body>
</html>
```

@ModelAttribute 还可以用在 controller中的参数上,用于指明向该参数注入相应的全局数据



