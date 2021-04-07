# Springmvc基础用法

```text
Mybatis中内置有个内置分页插件：PageHelper
图片服务器（FastDFS）
```

## 入门案例

### 步骤

1. 搭建开发环境
   * 选用骨架并补全项目结构
   * 配置前端控制器（dispatcherServlet） `<url-pattern>/<url-pattern/>`拦截所有除jsp的请求
   * SpringMVC配置文件
   * tomcat部署项目
2. 编写入门的程序
   * 使用bean来管理该类（IOC）
   * 方法上注解@RequestMapping\(path="/xxx"\)请求路径为xxx时调用该方法
   * 在web.xml中配置

     ```markup
     <init-param>
     <name>
     <!--将bean.xml配置进来，默认初始化的时候就加载该文件-->
     <values>
     <init-param>
     ```
3. SpringMVC配置
   * 注解扫描包
   * 视图解析器对象
   * 开启SpringMVC框架注解的支持

## 请求参数绑定\(主要是页面怎么写，处理请求方法，直接写参数类型就行\)

1. 简单数据类型SpringMVC会自动封装
2. pojo类：页面参数名称（form表单name项）需要跟 pojo 的属性名一致
3. 包装pojo\(Account 类中有一个 User 类型的参数 （user）\)
   * 举例：user.username 可以把 username 的值绑定到 Account 类中的 user 属性的 username 属性上
4. 数组：在页面表单项中有多个name属性相同且等于数组的形参名或时pojo类中的数组属性，就可以
   * 页面：name : id
   * 后台：String\[\]  ids    或者   JavaBean 中有一个 String\[\] ids 也可以绑定
5. list集合
   * 页面名称：list\[索引\].泛型属性
   * 后台：`List<User> list`

     ```java
       JavaBean： Account
           List<User> users;
           //setter/getter方法
       页面：
           users[0].username
           users[0].age
           users[1].username
           users[1].age
       Controller 中
           Account account
       会把 users 映射到 Account 中 users 上
     ```
6. Map集合：
   * 页面名称:map\['key'\].泛型属性

     ```java
       JavaBean： Account
           Map<String,User> userMap;
           //setter/getter方法
       页面：
           userMap['key1'].username
           userMap['key1'].age
           userMap['key2'].username
           userMap['key2'].age
       Controller 中
           Account account
       会把 userMap 映射到 Account 中 userMap 上
     ```

## 注解

1. RequestMapping
   * 可以标记在类和方法上，如果类和方法上都标记有该注解，则访问时需要   /类注解url/方法注解url
2. RequestParam
   * @RequstParam\(value="页面参数名",required=true,defaultValue="0"\) 数据类型  形参名随意
   * value：参数名称
   * requried：指定是否必须
   * defaultValue：指定默认值
3. RequestBody
   * 映射请求体，只支持 post 方法
   * 会把表单中的参数映射成   参数名1=值1&参数名2=值2&........
   * 后期一般情况下，会把  参数名1=值1&参数名2=值2&........   解析成 json 串
4. ResponseBody
5. PathVariable
   * 获取 url 中绑定的参数

     ```java
     @RequestMapping("/hello/{id}")
     public String hello(@PathVariable Integer id){
       system.out.println(id)
       return "success";
     }
     ```
6. ModelAttribute
   * 该注解标记的方法会在目标 Handler 执行之前执行，可以为 model 准备数据
   * 当表单提交数据不是完整的实体类数据时，保证没有提交数据的字段使用数据库对象原来的数据。

