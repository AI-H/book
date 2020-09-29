# 抽象工厂模式

[toc]

## 什么是抽象工厂

1. 提供一个接口，用于创建相关或依赖对象的家族，而不需要明确指定具体类。
2. 之前分享学习了工厂模式，这两种模式经常被拿来一起说明，其实用途差不多，只不过抽象工厂和工厂模式的作用范围不一样，在文章最后会将这两个模式坐下对比。

## 如何使用抽象工厂

1. 抽象工厂用于创建一类产品，针对的是一个家族
2. 比如我有两大类产品，这两种产品是可以相互结合起来使用的，比如车和油，应该是一一对应，所以我们就可以用抽象工厂将创建这个产品的家族抽象出来
3. 代码实现：



顶层的抽象工厂：

```java
/**
 * 抽象工厂
 */
public abstract class AbstractFactory {

    public abstract Oil createOil();

    public abstract Car createCar();
}
```



需要创建的产品家族：

```java
/**
 * 油
 */
@Data
public class Oil {
    private String name;
    private Double price;
}
/**
 * 煤油
 */
public class Kerosene extends Oil{
}
/**
 * 润滑油
 */
public class LubricatingOil extends Oil{
}

```

```java
/**
 * 车的总称
 */
@Data
public class Car {
    private String name;
    private Integer wheelNumber;
    private String  brand;
    private Double price;
}
/**
 * 自行车
 */
public class Bike extends Car{
}
/**
 * 火车
 */
public class Train extends Car{
}
```

抽象工厂的子类实现：

```java
/**
 * 火车相关的建造工厂
 */
public class TrainFactory extends AbstractFactory {

    @Override
    public Oil createOil() {
        return new Kerosene();
    }

    @Override
    public Car createCar() {
        return new Train();
    }
}

/**
 * 自行车相关制造工厂
 */
public class BikeFactory extends AbstractFactory{

    @Override
    public Oil createOil() {
        return new LubricatingOil();
    }

    @Override
    public Car createCar() {
        return new Bike();
    }
}
```



使用抽象工厂：

```java
public class CarShop {
    public static void main(String[] args) {
        //获得抽象工厂的实现，也可以抽离成工厂模式，创造一个获取抽象工厂实现的工厂
        AbstractFactory factory =new BikeFactory();
        factory.createCar();
        factory.createOil();
    }
}
```



## 实现抽象工厂的必要条件

1. 顶层的抽象工厂（可以是接口也可以是抽象类），定义了一系列创造产品的接口
2. 一系列抽象产品，最顶层的产品类是平行的，（比如上面的Oil和Car），顶层产品可以是抽象类也可以是接口也可以是正常的类，抽象模式定义为接口。
3. 抽象工厂的子类，每个子类都创建一个家族的产品，比如上面的BikeFactory,只生产自行车相关的产品，只生产自行车家族的产品，而TrainFactory，只生产火车家族的产品。
4. 抽象工厂的子类的实现为工厂方法的实现，我理解的，抽象工厂也可以是多个工厂模式的抽象，但是多个工厂模式聚合抽象更灵活。

## 抽象工厂的其他用法

1. 抽象工厂模式，必要的条件上述已经列出，顶层工厂，工厂实现子类，一系列产品。
2. 除了生产一系列的家族产品，也可以选择实现，比如对上面的抽象工厂进行改动

```java
/**
 * 抽象工厂
 */
public abstract class AbstractFactory {

    public abstract Oil createOil(String name);

    public abstract Car createCar(String name);
}
```

抽象类的实现：

```java
/**
 * 只实现生产油的方法，另外一个默认实现返回null;
 */
public class OilFactory extends AbstractFactory {
    @Override
    public Oil createOil(String name) {
        if ("Kerosene".equals(name)) {
            return new Kerosene();
        }else if ("LubricatingOil".equals(name)){
            return new LubricatingOil();
        }
        return null;
    }
    @Override
    public Car createCar() {
        return null;
    }
}
```

3. 以上也是抽象工厂的一种实现方法，根据情况选择不同的实现方法，因为抽象工厂制造出来就是为了提供一个接口，用于创建相关或依赖对象的**家族**，这个也是一个家族。

## 抽象工厂和工厂模式的区别

1. 抽象工厂的子类实现是工厂模式，即只看实现的子类的话，就是简单工厂方法。类似上个所说的只实现某一个创建方法，创建一个家族对象，仅看这一个的话，这个就是工厂模式。
2. 工厂模式使用继承：把对象的创建委托给子类，子类实现工厂方法来创建对象；抽象工厂使用对象的组合：对象的创建被实现在工厂接口所暴漏的方法中。
3. 工厂模式和抽象工厂的使用区别就是，工厂模式创建一类产品，抽象工厂创建一个产品家族

### 抽象工厂的优缺点

1. 优点：
   1. 更符合抽象原则
   2. 跟符合依赖倒置原则，使用抽象工厂创建相关的产品家族，可以不依赖于产品具体对象
2. 缺点：
   1. 对开闭原则有倾斜性
      - 对已经有的产品，修改只需要修改具体的实现类即可
      - 对未实现的产品家族，需要添加的时候，就需要对整个实现体系进行修改，也是很不方便的地方

### 总结

设计模式主要传达的是一种设计思想，我们并不应该拘泥于固定的实现形式，毕竟代码是死的，人是活的，我们也需要根据不同的场景，不同的业务，使用这种思想去做最符合自己的实现。抽象工厂模式告诉我们的就是，需要床建的产品很多的时候，可以把这种创建抽象到最顶层的实现，由子类去实现产品家族，有可能是一系列的平行产品，也可能是具体实现一类产品。