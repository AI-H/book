# Springmvc上传文件

## SpringMVC的控制器响应数据和结果视图

### 响应返回值类型分类

1. String类型
   * 方法返回值为字符串类型，可以指定逻辑视图名，被视图解析器加前缀和后缀，得到物理视图，返回给客户端
2. void类型
   * 方法返回值为void时，如果方法体内没有请求转发，或重定向的操作，会报404错误
   * 可以在方法参数处添加`HttpServletRequest request, HttpServletResponse`参数，用request请求转发方式给出响应，需要的数据可以添加到request域中
   * 也可以用response来给出重定向，或者是返回Ajax请求的结果
3. ModelAndView类型
   * 该对象可以作为控制器的返回值
   * 第一个方法：`mav.addObject(String attributeName,Object attributeValue)`设置返回给页面的数据，k-v格式，可以在jsp页面上用`${attributeName}`获得值（跟request域存值一样）
   * 返回ModelAndView类型，浏览器只是请求转发，要想jsp可以用El表达式，需要在xml头中设置：`isELIgnored="false"`
   * 第二个方法：`mv.setViewName("success");`用于设置逻辑视图名称，视图解析器会根据名称前往指定的视图（相当于返回String类型的字符串）

### SpingMVC结果视图

1. 视图跳转方式
   * 控制器提供了返回String类型值时，默认就是请求转发
   * 不过还提供两个类似命令的关键字，必须在String类型的返回值前面
   * return:"forward:/WEB-INF/page/success.jsp"代表请求转发到success.jsp界面,forward后面只能写实际视图，不能写成逻辑视图
   * return:"redirect:hello2"代表重定向到hello2这个路径下，如果重定向到jsp界面，则jsp不能写在WEB-INF下面，否则找不到
2. 过滤静态资源
   * 页面请求一些静态资源时不需要拦截下来，则需要在Springmvc.xml下配置，当客户端请求一些静态资源时，不拦截

     ```markup
       <!--前端控制器，哪些静态资源不拦截-->
       <mvc:resources location="/css/" mapping="/css/**"/>
       <mvc:resources location="/images/" mapping="/images/**"/>
       <mvc:resources location="/js/" mapping="/js/**"/>
     ```
3. Ajax请求响应json数据格式
   * 异步请求返回json数据，，在方法参数上加`@RequestBody`在方法返回值上面加@ResponseBody注解,SpringMVC可以自动将返回值类型转换成json数据类型。
   * @ResponseBody也可以加到该方法上，一样的效果
   * 必须导入jackson相关的三个jar包，否则会报415异常

## 文件上传

### 上传的三个要求

1. method方法必须是POST
2. form表单的enctype取值必须是multipart/form-data,enctype:表单请求正文的请求
3. 提供一个表单域：`<input type="file"/>`
4. 传统方式

   ```java
        /**
     * 文件上传
     * @return
     */
    @RequestMapping("/fileupload1")
    public String fileuoload1(HttpServletRequest request) throws Exception {
        System.out.println("文件上传...");

        // 使用fileupload组件完成文件上传
        // 上传的位置
        String path = request.getSession().getServletContext().getRealPath("/uploads/");
        // 判断，该路径是否存在
        File file = new File(path);
        if(!file.exists()){
            // 创建该文件夹
            file.mkdirs();
        }
        // 解析request对象，获取上传文件项
        DiskFileItemFactory factory = new DiskFileItemFactory();
        ServletFileUpload upload = new ServletFileUpload(factory);
        // 解析request
        List<FileItem> items = upload.parseRequest(request);
        // 遍历
        for(FileItem item:items){
            // 进行判断，当前item对象是否是上传文件项
            if(item.isFormField()){
                // 说明普通表单向
            }else{
                // 说明上传文件项
                // 获取上传文件的名称
                String filename = item.getName();
                // 把文件的名称设置唯一值，uuid
                String uuid = UUID.randomUUID().toString().replace("-", "");
                filename = uuid+"_"+filename;
                // 完成文件上传
                item.write(new File(path,filename));
                // 删除临时文件
                item.delete();
            }
        }

        return "success";
    }
   ```

5. springMVC上传文件原理

   ```java
        /**
     * SpringMVC文件上传
     * @return
     */
    @RequestMapping("/fileupload2")
    public String fileuoload2(HttpServletRequest request, MultipartFile upload) throws Exception {
        System.out.println("springmvc文件上传...");

        // 使用fileupload组件完成文件上传
        // 上传的位置
        String path = request.getSession().getServletContext().getRealPath("/uploads/");
        // 判断，该路径是否存在
        File file = new File(path);
        if(!file.exists()){
            // 创建该文件夹
            file.mkdirs();
        }

        // 说明上传文件项
        // 获取上传文件的名称
        String filename = upload.getOriginalFilename();
        // 把文件的名称设置唯一值，uuid
        String uuid = UUID.randomUUID().toString().replace("-", "");
        filename = uuid+"_"+filename;
        // 完成文件上传
        upload.transferTo(new File(path,filename));

        return "success";
    }
   ```

   * `MultipartFile`:SpringMVC文件上传核心接口对象
   * 需要在`springmvc.xml`中配置

     ```markup
       <!--配置文件解析器对象
           id 属性 必须是下面的这个名称    
       -->
       <bean id="multipartResolver"         
           class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
           <property name="maxUploadSize" value="10485760" />
       </bean>
     ```

   * 方法中MultipartFile  名称（需要跟前台 file 域的name属性一致）（upload）
   * MultipartFile  有一个方法`getOriginalFilename()`获取文件名称

6. 跨服务器上传文件

   ```java
        /**
     * 跨服务器文件上传
     * @return
     */
    @RequestMapping("/fileupload3")
    public String fileuoload3(MultipartFile upload) throws Exception {
        System.out.println("跨服务器文件上传...");
        // 定义上传文件服务器路径
        String path = "http://localhost:9090/uploads/";
        // 说明上传文件项
        // 获取上传文件的名称
        String filename = upload.getOriginalFilename();
        // 把文件的名称设置唯一值，uuid
        String uuid = UUID.randomUUID().toString().replace("-", "");
        filename = uuid+"_"+filename;
        // 创建客户端的对象
        Client client = Client.create();
        // 和图片服务器进行连接
        WebResource webResource = client.resource(path + filename);
        // 上传文件
        webResource.put(upload.getBytes());
        return "success";
    }
   ```

## 异常处理机制

1. 自己定义一个异常类SysException
2. 自定义一个异常处理器实现`HandlerExceptionResolver`接口，重写抽象方法

   ```java
        /**
        * 异常处理器
        */
        public class SysExceptionResolver implements HandlerExceptionResolver{

            /**
            * 处理异常业务逻辑
            * @param request
            * @param response
            * @param handler
            * @param ex
            * @return
            */
            public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
                // 获取到异常对象
                SysException e = null;
                if(ex instanceof SysException){
                    e = (SysException)ex;
                }else{
                    e = new SysException("系统正在维护....");
                }
                // 创建ModelAndView对象
                ModelAndView mv = new ModelAndView();
                mv.addObject("errorMsg",e.getMessage());
                mv.setViewName("error");
                return mv;
            }

        }
   ```

3. 在 springmvc.xml 配置文件中配置异常处理器bean

   ```markup
    <bean class="自定义异常处理器类的全类名"/>
   ```

4. 需要使用的时候，在catch代码块里 throw new 自定义异常（信息）

