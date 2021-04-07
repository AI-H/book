# 生成器（建造者）模式

[toc]

## 什么是生成器模式

1. 生成器模式：封装了一个复杂产品的构建过程，并允许按照步骤构造。
2. 生成器模式将一个对象的复杂构建，拆分一个一个的步骤，可以根据需求按照步骤来建造这个产品。
3. 对于一些构建方式，和装配方式相互独立的时候，也可以用建造者模式。例如你想要建造一个房子，房子的构建，可能都需要混凝土，玻璃，涂料，钢筋等，但是装配方式不一样，产生的对象也就不一样。
4. 在建造着模式中，可以自定义每个组件的构建或者装配过程。

## 生成器模式有什么好处

1. 将复杂的创建对象拆解成步骤，自己指定步骤的组合。
2. 将一个庞大的实例，封装到一个独立的对象中去完成，就好像我们将初号机的生产造了个工厂。
3. 将对象创建的步骤灵活化，并且可以根据建造者的实现，生成符合我们不同需求的对象。

## 如何使用生成器模式

### 生成器的四个必要对象

1. 一个相对比较灵活的，可以有弹性结构的产品对象，就是一个具体要生产的东西，根据需求灵活配置。
2. 抽象的产品建造者，内部封装了构建这个产品所需要的步骤，以及一个固定的获得该对象的方法。
3. 对抽象产品建造者的各个实现，实现了抽象建造者的方法，具体定义了产品每个步骤的细节。
4. 客户，需要定制化生成产品的对象，就是使用生成器来生成产品的对象，有时候也会被封装成建造者工厂，内部流水线式建造产品，固定化生产的步骤。

### 代码实现

1. 我们需要将电脑的生产拆分成一个一个步骤，并封装起来，首先是电脑对象：

```java
/**
 * 产品类,使用lombok来节省setter,getter,和构造方法
 */
@Data
public class Computer {
    private String cpu;
    private String mem;
    private String storage;
    private String screen;
}
```

2. 抽象的建造者

```java
**
 * 抽象的生成器接口
 */
public abstract class AbstratBuilder {

    public abstract void addCpu();
    public abstract void addMem();
    public abstract void addStorage();
    public abstract void addScreen();
    public abstract void buildNewComputer();
    public abstract Computer getComputer();
}
```

3. 抽象建造者的实现类：

```java
/**
 * 具体的建造者--intel
 */
public class IntelBuilder extends AbstratBuilder{

    private Computer computer;
    
    @Override
    public void addCpu() { computer.setCpu("I7-9750H");}
    @Override
    public void addMem() {  computer.setMem("三星");}

    @Override
    public void addStorage() { computer.setStorage("SSD-1T");}

    @Override
    public void addScreen() {computer.setScreen("三星-oleds");}

    @Override
    public void buildNewComputer() {computer = new Computer();}

    @Override
    public Computer getComputer() { return computer;}
}
```

```java
/**
 * 具体建造者-AMD
 */
public class AMDBuilder extends AbstratBuilder{
    private Computer computer;

    @Override
    public void addCpu() { computer.setCpu("AMD-R7-3800X");}

    @Override
    public void addMem() { computer.setMem("海盗船");}

    @Override
    public void addStorage() {computer.setStorage("westdata-ssg-550g"); }

    @Override
    public void addScreen() { computer.setScreen("acer-23");}

    @Override
    public void buildNewComputer() {computer = new Computer();}

    @Override
    public Computer getComputer() {return computer;}
}
```

4. 客户使用类：

```java
/**
 * 使用客户
 */
public class Client {

    public static void main(String[] args) {
        AbstratBuilder builder = new IntelBuilder();
        builder.buildNewComputer();
        builder.addCpu();
        builder.addMem();
        builder.addScreen();
        builder.addStorage();
        Computer computer = builder.getComputer();
    }
}
```

有时候也会被封装成一个固定流程：

```jav
**
 * 固定的建造商，类似于你买了电脑组件后，交给别人去实际组装，步骤对使用者不可见
 */
public class MustBuilder {

    private AbstratBuilder builder;

    public MustBuilder(AbstratBuilder builder){
        this.builder = builder;
    }

    public void bulidComputer(){
        builder.addStorage();
        builder.addScreen();
        builder.addMem();
        builder.addCpu();
    }

    public Computer getComputer() {
        return builder.getComputer();
    }
}

/**
 * 交给别人组装
 */
public class MustClient {

    public static void main(String[] args) {
        MustBuilder mustBuilder = new MustBuilder(new IntelBuilder());
        mustBuilder.bulidComputer();
        Computer computer = mustBuilder.getComputer();
    }
}
```



## 生成器模式使用问题

### 构建一个使用抽象生成器的对象（MustBuilder）是否必要

1. 是否要有固定的组装步骤，也就是上面所说的是不是需要交给专门的人去组装

2. 对于构建者来说，只需要必要的三个就已经完成了需求了，抽象的建造者，具体的建造者们，要建造的产品；那么还需不需要在造一个对象，MustBuilder,定义好固定的构建步骤，只需要传入一个实际的建造者，就直接出产品。
3. 针对这个问题有一个判断条件，就是，你用这个建造者模式为了解决什么问题
   1. 如果使用建造者模式仅仅是为了将构建对象的过程拆开，且组装对象的步骤固定，那就加上MustBuilder
   2. 如果你需要根据步骤的不同产生不同功能的产品，那就不要使用。
4. 结合之前用到的工厂模式，如果你有很多组抽象生成器和对象，那你就可以写一个MustBuilder，根据不同的产品以及需求，来获取不同的生成器。

### 对于产品实例的引用位置

1. 生成器模式中，产品需要在生成器中有一个引用，不然也没法构建出来一个产品
2. 以上代码中，产品的引用是在具体的实现建造者中，有的地方（wiki）把产品的引用放到，抽象的生成器中，并使用**protected **修饰。
3. 建议放到具体的实现建造者中
4. 抽象的建造者可以设计成接口，也可以设计成抽象类。

### 总结

生成器模式主要解决的事情就是，将一个复杂的对象创建过程，拆解成一个个步骤，并封装到一个对象中，并可以根据步骤的不同或者实现者不同获取功能不同的实例产品对象。

学习模式应该学习思想，而不是固定的代码，符合自己的要求，没有什么性能安全问题，代码按什么方式写都可以，代码也不应该是死板的。

