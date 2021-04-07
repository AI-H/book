# 工厂方法模式

\[toc\]

## 工厂模式是什么

1. 在JAVA中，创建一个新的实例，最直接的方法就是new一个，没有对象怎么办，new一个就有了。除了new一个实例外，还有很多种创建实例的方法，比如之前所说的，单例模式，原型模式，一个是固定从一个方法中获取一个实例，一个是通过已有克隆出一个实例，这两个应用场景完全相反，但是便利之处可以学习，由此便有了工厂模式，从固定的一个方法中获取实例，由方法内部根据你每次的请求，帮你创建克隆或者什么地方给你一个实例。
2. 工厂模式，定义了一个创建对象的接口，但由子类决定要实例化的类是哪一个。工厂方法让类把实例化推迟到子类。

## 工厂模式的好处

1. 对于有很多派生类的实例，可以通过工厂模式，固定一个方法，根据参数不同来获取不同功能的对象
2. 对于使用者来说，只需要控制传入工厂的参数类型就可以，不必关心他需要的实例的创建过程
3. 就像是一个大型超市，把所有的物品的进货渠道生产过程都隐藏掉，而顾客只需要进入这个超市就可以买到所有超市里面有的东西，每次只需带着买的东西清单就行。

## 如何实现

1. 工厂模式的实现也很简单，只要符合通过一个方法获取实例即可

```java
/**
工厂类
*/
public class CommodityFactory {

    public Commodity sellComodityByName(String name){
        Commodity commodity = null;
        if ("hongniu".equals(name)){
            commodity =  new HongNiu();
        } else if ("lvcha".equals(name)) {
            commodity = new LvCha();
        }
        return commodity;
    }
}
```

```java
/**
* 工厂客户类
*/
public class SuperMarket {

    private CommodityFactory commodityFactory;

    public SuperMarket (CommodityFactory commodityFactory) {
        this.commodityFactory = commodityFactory;
    }

    public Commoodity sellCommodity(String name){
        Commodity commodity = commodityFactory.sellComodityByName(name);
        System.out.println(commodity.dabao());
        return commodity;
    }

    public static void main(String[] args) {
        SuperMarket superMarket = new SuperMarket(new CommodityFactory());
        superMarket.sellCommodity("hongniu");
    }

}
```

以上代码就能算是一个简单的工厂，通过商品名称，给一个商品。当然是比较简单的，设计模式其实离我们很近。

1. 可能有人就会觉得这就是工厂模式了？不会是假的吧。其实这个就算是个工厂模式了，只不过生产方式比较简单，种类也不多，功能也不多。
2. 工厂模式一般会有以下几个类型
   1. 工厂的“产品”，工厂需要生产的产品（大名词，范围比较广的描述，类似上面的Commoodity），可以定义成一个抽象类，有一些有用实现，但这些实现可以被重写，
   2. 工厂具体生产的产品，继承或实现于上面的产品，必须是一个具体类，相当于上面的HongNiu和LvCha
   3. 用来创建产品的工厂，唯一可以实例化产品的地方，一般实例化的方法都是静态的，也就是静态工厂
   4. 工厂的客户，需要工厂创建实例的需求者，需要工厂创建产品的使用者，相当于上面SuperMarket

## 工厂模式的优化和拓展

1. 静态工厂模式 1. 一般工厂用来创建产品的方法都是静态的 2. 好处是，随用随建，不需要使用者（超市）在实例化的时候，传入一个工厂实例 3. 不好的地方是，永远只能用这一个工厂，因为工厂的方法是静态的，不能被重写，所以也就无法被继承，拓展出其他的工厂，就相当于方圆百里只有一家商品工厂，别人想加盟都不行。 4. 如果不需要对工厂的方法进行扩展，那静态工厂模式是一个好选择。

   ```java
   /**
   工厂类
   */
   public class CommodityFactory {

       public static Commodity sellComodityByName(String name){
           Commodity commodity = null;
           if ("hongniu".equals(name)){
               commodity =  new HongNiu();
           } else if ("lvcha".equals(name)) {
               commodity = new LvCha();
           }
           return commodity;
       }
   }
   ```

2. 加盟工厂（分工厂）模式

   1. 工厂类可以被继承，工厂创建方法可以被重写
   2. 工厂类的子类，可以分别定义创建物品的具体实例
   3. 加盟工厂模式，可以将工厂的创建能力细分化
   4. 客户只需要选择对应的工厂即可创建出对应功能的实例

   ```java
   /**
    顶层工厂类
   */
   public class CommodityFactory {

       public abstract Commodity sellComodityByName(String name);
   }
   ```

   ```java
   **
    * @author 飞客不去
    * 子类工厂，加盟商
    */
   public class DrinkCommodityFactory extends CommodityFactory{
       /**
        * 根据名称来获取饮料商品
        *
        * @param name
        * @return
        */
       @Override
       Commodity getComodityByName(String name) {
           Commodity commodity = null;
           if ("hongniu".equals(name)){
               commodity =  new HongNiu();
           } else if ("lvcha".equals(name)) {
               commodity = new LvCha();
           }
           return commodity;
       }
   }
   ```

   ```java
   /**
   使用工厂创建产品
   */
   public static void main(String[] args) {
           CommodityFactory drinkCommodityFactory = new DrinkCommodityFactory();
           Commodity hongniu = drinkCommodityFactory.getComodityByName("hongniu");
           hongniu.dabao();
    }
   ```

### 注意点和区别

1. 工厂模式就是，定义一个创建对象的接口，让其子类自己决定实例化哪一个工厂类，工厂模式使其创建过程延迟到子类进行，就是我们所优化的加盟工厂模式，这个其实是工厂模式的基本形态，用的很多。但是跟抽象工厂模式还是不一样的。
2. 静态工厂模式，一般会将工厂做成单例的，也叫单例静态工厂模式，也很常见，一般见于只需要一个工厂就能完成实例创建，且希望的效果就是，将代码解耦，将实例的创建单独抽取。
3. 工厂模式的好处就是将实例的创建与使用解耦，不必关心实例是怎么创建的，顶级工厂可以将创建实例方法抽象，由子类去指定如何创建对象以及，创建具体什么对象。

