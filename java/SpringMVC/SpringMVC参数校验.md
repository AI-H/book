# SpringMVC参数校验
[toc]

### springMVC 的基本参数校验
### 使用@RequestParam 注解控制请求参数
1. 在写控制层的接口时，我们需要对前端或者接口访问者进行访问参数的校验，如果将参数的校验逻辑写到控制层的代码里，会造成代码重复，资源浪费，不美观。
2. 对于控制层接口的基本类型方法参数，我们可以用注解控制，一般都是GET方法居多
3. 相配合的还需要一个自定义的全局错误返回处理，这样就可以完全的自定义了

例如:
```java
	@GetMapping("/hello")
    public Response sayHello(@RequestParam String hello,long start) {
        return Response.success(hello);
    }

```

```java
   @ExceptionHandler
    public Response exceptionMatchHandle(Exception e){
            //参数绑定异常 SpringMVC异常
            if (e instanceof ServletRequestBindingException){
                log.error("发生了SpringMVC异常：{}",e.getMessage());
                if (e instanceof MissingRequestHeaderException){
                    return Response.failure(ErrorCode.ERROR_PARMS_NO_HEADER);
                }
                return Response.failure(ErrorCode.ERROR_PARMS_NO_PARM);
            }
    }
```

如上所示，在接口声明中可以将请求参数加上@RequestParam 注解，当访问接口没有传该参数的时候，springmvc会报异常然后可以使用自定义的异常处理统一捕获处理即可，

--- 在请求参数的声明中，尽量不要声明基本类型上图的**start**，因为如果声明了基本类型就相当于，该参数是必传的，如果没有就会报错，如果你希望**start**是非必须的，可以声明成引用类型Long。

### @RequestParam的使用

1. @RequestParam的可用属性

   ```java
   @java.lang.annotation.Target({java.lang.annotation.ElementType.PARAMETER})
   @java.lang.annotation.Retention(java.lang.annotation.RetentionPolicy.RUNTIME)
   @java.lang.annotation.Documented
   public @interface RequestParam {
       @org.springframework.core.annotation.AliasFor("name")
       java.lang.String value() default "";
   
       @org.springframework.core.annotation.AliasFor("value")
       java.lang.String name() default "";
   
       boolean required() default true;
   
       java.lang.String defaultValue() default "\n\t\t\n\t\t\n\uE000\uE001\uE002\n\t\t\t\t\n";
   }
   ```

   - 以上是@RequestParam注解的声明，可以看到由四个属性组成，name和value相当于同一个，也就是可用的有三个
   - name(value)：标识该参数的名称，也就是访问接口时需要传的参数名称，默认是接口定义时的参数名，对应上述的hello,但可以通过该属性覆盖重写参数名  **@RequestParam(name="hi") String hello**此时访问接口传参名得是hi
   - required:标识该参数是不是必须参数，默认是true就是必传，如果没有该参数就会报错，如果改为false,则该参数不是必须，可以不传
   - defaultValue() : 当访问接口时，该参数没有传的话，就给与一个默认的值

   ```java
   @GetMapping("/hello")
   //当hello参数没传的时候，默认给hello一个值，defaultValue 与required的值无关
       public String sayHello(@RequestParam(defaultValue = "asasasasa") 	String hello) {
           return hello;
       }
   ```

   - defaultValue 与required的值无关，当required为true时，配置了defaultValue，即便不传hello的值，依然可以访问成功，返回值为 asasasasa

## 使用JSR-303验证框架

spring boot 支持JSR-303、Bean验证框架，默认实现用的是 Hibernate validator。在Spring MVC中，只需要使用@Valid注解标注在方法参数商，Spring Boot即可对参数对象进行校验，校验结果会放在BindingResult对象中。

### 如何使用验证框架
#### 相关注解
JSR-303定义了一系列的注解用来验证Bean的属性，常用的有以下几种，用于标注在被校验对象的类字段之上。全部注解可以参考[*JSR 303* – Bean Validation](https://beanvalidation.org/2.0/spec/#builtinconstraints)

- 对于空的检查
  - @Null：验证对象是否为空；
  - @NotNull：验证对象不为空；
  - @NotBlank：验证字符串不为空或者不是空字符串，比如""和“    ”都会验证失败；
  - @NotEmpty：验证对象部位null，或者集合不为空。
- 长度的检查：
  - @Size(min= ,max=)：验证对象长度，可支持字符串、集合；
  - @Length：字符串的大小。
- 数值检测：
  - @Min：验证数字是否大于等于指定的值；
  - @Max：验证数字是否小于等于指定的值；
  - @Digits：验证数字是否符合指定格式，如@Digits(integer=9,fraction=2)表示，数字要求必须是小数点前有9位，小数点后有2位;
  - @Range：验证数字是否在指定范围内，如@Range(min=1,max=1000)。
- 其他：
  - @Email：验证是否为邮件格式，为null则不做校验；
  - @Pattern：验证String对象是否符合正则表达式的规则。
- 注解依赖的包：

```java
		<dependency>
			<groupId>jakarta.validation</groupId>
			<artifactId>jakarta.validation-api</artifactId>
			<version>2.0.2</version>
		</dependency>
```



#### 如何使用

1. 首先在需要校验的类中字段上标注需要的注解

   ```java
   /**
    * @author 飞客不去
    */
   @Data
   public class Drink {
   
   
       @NotNull
       private String name;
   
       @Size(min = 1,max = 10000)
       private String describ;
   
       @Digits(integer = 2,fraction = 2)
       private Double price;
   }
   ```

2. 之后在请求参数处标注上校验注解：

```java
@PostMapping("/drink")
    public String addDrink(@Valid Drink drink, BindingResult result){
        //校验的结果
        System.out.println(result);
        return drink.getName();
    }
```

3. 如上所述，在drink类中需要校验的字段上标注对应的控制注解，之后在请求参数处标注注解@Valid 声明该参数需要校验即可
4. 在需要校验的接口中增加参数，**BindingResult result**，框架会将校验的结果放在该对象中，
   1. 该对象包含了所有验证结果
   2. 可以使用提供的两个方法查看
      1. hasErrors,判断验证是否通过；
      2. getAllErrors,得到所有错误信息，通常返回FieldError列表
   3. 如果在需要校验的接口参数处没有声明**BindingResult result**参数，则SpringMVC会抛出异常，可以使用全局异常处理。
   4. 对于验证的结果一般都会让框架抛异常出去，但是如果想要自己查看并解决校验结果，可以增加**BindingResult result**对象

### 框架的高级用法

1. 当我们接口的参数内部还引用其他对象时，需要对内部的对象也做校验应该如何？

   ```java
   @Data
   public class Food {
       @Valid
       private Drink drink;
       @NotNull
       private String type;
   }
   ```

   ```java
   @PostMapping("/food")
       public String addFood(@Valid Food food){
   
           return food.getDrink().getName();
       }
   ```

   只需要在参数对象内部的对象引用字段上加上@Valid注解即可，此时框架就会对Food 内部的Drink类也会做校验了，不加则不会做校验，即便Drink内部也有校验注解。

2. 对于同一个Drink类，可能不同的接口需要做的校验类型不同，那么总不能一个接口可以用，另一个接口就不能在用了吧。

   1. 对于这个情况，一般都会有解决方法，不会说只试用单个，那么就没什么扩展的意义。

   2. 这个情况可以类似于我们之前说的@Jsonview，这是可以分组的，不过要加上一个注解@Validate

   3. 具体实施如下

      1. 在参数类的内部新建接口用于分组，
      2. 在每个校验注解内部配置所属的组
      3. 在接口定义的地方使用@Validate注解并定义好校验需要的接口分组

      代码实现如下：

      1. 实体类（参数类）

      ```java
      @Data
      public class Drink {
          public interface Update{}
          public interface Add{}
      
      
          @NotNull(groups = {Update.class,Add.class})
          private String name;
      
          @Size(min = 1,max = 10000,groups = {Add.class})
          @NotNull(groups = {Update.class})
          private String describ;
      
          @Digits(integer = 2,fraction = 2)
          private Double price;
      }
      ```

      2. 接口定义：

      ```java
      @PostMapping("/updatedrink")
          public String updateDrink(@Validated(Drink.Add.class) Drink drink){
              return drink.getName();
          }
      ```

      



### 框架的扩展

1. 框架给出的注解是有限的，但是大部分情况下我们已经够用了，
2. 我们有时候自己的业务比较复杂，对校验也可能比较复杂。
3. 此时我们可以自己定义一个我们自己的注解，校验框架也是允许自定义注解的 。

#### 自定义校验

1. 举例说明自定义注解的实现：我们需要一个自定义注解来校验货物的库存

```java 
@Stock
private int weight;
```

2. 自定义注解如下格式

```java
@Target({ElementType.METHOD, ElementType.FIELD,  ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Constraint(validatedBy = {StockValidator.class})
public @interface Stock {

    String message() default "库存不能低于{min}个";
    int min() default 4;
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

- 自定义注解Stock跟其他注解差不多，作用就是如果属性值小于min的话，就会报错。

- 需要提供一个@Constraint来说明需要什么类来验证注解
- 验证注解必须要包含以下几个属性
  - message,用于创建错误细腻些，支持表达式，如库存不能低于{min}个
  - groups,验证规则分组
  - payload,定义了验证的有效负荷
  - 后两个可以自行琢磨怎么使用，也可以不使用

3. 新建一个StockValidator类来验证注解

```java
/**
 * Stock注解校验类
 */
public class StockValidator implements ConstraintValidator<Stock,Integer>{

    Stock stock;
    int min;
    @Override
    public void initialize(Stock stock) {
            this.stock =stock;
            this.min =stock.min();
    }

    @Override
    public boolean isValid(Integer integer, ConstraintValidatorContext constraintValidatorContext) {
        //校验逻辑
        if (integer == null) {
        return false;
        }
        return integer>min;
    }
}
```

- StockValidator类必须实现ConstraintValidator接口initialize方法以及验证方法isValid

- 具体的校验逻辑在isValid方法中做校验
- 其他参数可以自行琢磨，不在本篇的介绍中，这个程度至少自己就够用了



### 结语

写博客是为了记录自己的学习，也是分享给大家自己的问题解决，如果能帮上看到这个博客的人时最好的了，且写且珍惜。





