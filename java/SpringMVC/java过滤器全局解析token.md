# java过滤器全局解析token
[toc]
## 使用过滤器定义一个全局的token解析器

在进行后端接口的开发过程中，一般涉及到人员用户，权限或者安全方面的考虑接口都会使用token来传递用户或者一些安全系数高的鉴权参数等。
### 一般接口定义
#### 全局AOP解析
使用AOP，对需要获取token信息的接口，进行方法增强，在进入controller之前，动态的为接口方法鉴权，或者为接口方法添加一个token相关的参数。
根据需求不同需要的参数也不一样，这个可以根据业务来，这方面代码也很多，原理就是在方法执行前，进行token鉴权，或者参数添加或提取。如果业务比较复杂，可以自己展开想象，自己动手搞。

#### 接口使用注解@RequestHeader
另一种方法是，每个接口自己去处理token的鉴权或者参数提取，在请求参数中声明一个从header里面获取的值，然后方法内部去解析
```java

@GetMapping("legend")
public Response getTheShy(@RequestHeader String token, @RequestParam() String top){
    //解析token鉴权。。
    //业务
	return null;
}

```
如果你不怕麻烦，或者只有很少的接口需要token这个方法可以考虑，对于微服务来说，token的鉴权可能是单独的一个服务，或者需要调用其他方法，把这个方法封装起来，其实也还好。

### 我的需求以及方法

#### 需求和现状
1. 业务几乎每个对外的而接口都需要用户的信息，比如用户ID或者用户名称
2. 解析和分发token的业务不是我的模块，我获得到的token是由网关分发到后台程序的token，解析需要依赖网关的服务
3. 我只需要token解析出来的用户某个信息
4. 我不想每个接口都去关心token是怎么解析的，怎么获取参数的

#### 实践
使用过滤器来控制请求参数的获取，控制请求参数的值

1. 控制层代码
具体代码：
controller:
```java
    @GetMapping("top")
    public Response<List<CustomerView>>  getAllView(String token){
   
      log.info(token);    
      return viewService.getAllView(token);
    }
```
在控制层接口中我直接声明我需要的而用户信息，即需要从token中解析的数据

2. 过滤器代码

具体代码：
```java
@Component
@WebFilter(filterName="token",urlPatterns= {"/*"})
public class FilterConfigController implements Filter {


    private static Logger log = LoggerFactory.getLogger(FilterConfigController.class);

    @Override
    public void init(FilterConfig filterConfig) {
        log.info("过滤器初始化");
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequestWrapper requestWrapper = new HttpServletRequestWrapper((HttpServletRequest) servletRequest){

            @Override
            public Map<String, String[]> getParameterMap() {
                return super.getParameterMap();
            }

            @Override
            public String getParameter(String name) {
                return super.getParameter(name);
            }

            @Override
            public String[] getParameterValues(String name) {
                if("token".equals(name) || "userId".equals(name)){
                    String token = this.getHeader("Token");
                    //鉴权或者获取token
                    if (StringUtils.isEmpty(token)){
                        log.error("缺少token");
                        throw new MyRuntimeException(ErrorCode.ERROR_AUTH_NULL_TOKEN);
                    }
                    if ("token".equals(name)){
                        //具体的解析token逻辑
                    }else if ("userId".equals(name)){
                        //具体的解析token逻辑
                    }
                    
                    return new String[]{token};
                }
                return super.getParameterValues(name);
            }

            @Override
            public String getQueryString() {
                return super.getQueryString();
            }
        };
        filterChain.doFilter(requestWrapper,servletResponse);

    }


}
```
1. 继承Filter，并实现其方法，主要对 doFilter方法中的getParameterValues方法重写
2. 该方法的作用就是，如果你得controller需要一个参数值时，会从这里获取，所以我们可以对此进行自定义的参数获取。
3. 规定好我们要获取的参数名称，可以叫做token,然后这边对这个参数名进行拦截，当需要获取我们指定参数名称的值时，由下面代码生成
4. 下面的代码就是从请求头获取token，然后鉴权和获取用户信息的具体实现
5. 如果请求头的token不满足我们的需求,或者解析失败，就可以直接返回前端，而不用进入controller
6.我采用的直接返回前端的方法是，直接在过滤器中抛出一个自定义的运行时异常（此方法不允许有异常，只能抛运行时异常）
7. 由全局的异常处理，来处理抛出的自定义运行时异常，并返回给前端
```java

/**
     * 处理运行时异常
     * @param e
     * @return
     */
    @ExceptionHandler
    public Response runtimeExceptionMatchHandle(RuntimeException e, HttpServletResponse response){

        if (e instanceof MyRuntimeException){
            e.printStackTrace();
            return Response.failure(((MyRuntimeException) e).getCode());
        }
        return Response.failure(ErrorCode.ERROR);
    }
```

自定义异常，抛出的时候也把要返回的信息返回
```java

/**
 * 自定义运行时异常
 */
public class MyRuntimeException extends RuntimeException{

    private ErrorCode code;
    public MyRuntimeException(ErrorCode code){
        super(code.getMsg());
        this.code = code;
    }

    public ErrorCode getCode() {
        return code;
    }
}
```

#### 注意点
1. 如果在controller的参数上加了@RequestParam注解，那么在获取这个参数值的时候不会进入到过滤器的方法中
    1. 在其他不需要进入自定义过滤的而参数加上注解，不加也可以，只要名称不一样就不会往下走
    2. 要获取的token相关参数，不要加此注解

2. 自定义的过滤器可以多个参数获取只需要在入口处，多加一个参数名称判断，且在返回参数值的时候，根据名称不同返回不同的值即可


如果有问题，欢迎指正，不明白的话可以私信博主