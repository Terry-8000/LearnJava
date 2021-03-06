# 学习UML图

### 1. 什么是UML图

UML（Unified Modeling Language）统一建模语言，是一种开放的方法，用于说明、可视化、构建和编写一个正在开发的、面向对象的、软件密集系统的制品的开放方法。

### 2. UML类图的组成

UML类图由三部分组成：类名、属性和方法。

eg：

```java
class Person {

    private String name = "personName";
    private int age;

    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }

}
```

![](../../assets/类图1.png)

**类名：**类名就是类的名称，如果是抽象类的话就在右下角加一个（Abstract）标识。

**属性：**就是类的成员变量，可见性表示为public（+）、protected（\#）、private（-）。在UML类图中可以表示属性的可见性、名称、类型和默认值，格式如下（\[\] 里面的表示可选，可以有可以没有）：

```
可见性  名称:类型 = 默认值
```

**方法：**可见性同属性，格式如下：

```
可见性  名称(参数列表) : 返回类型
```

### 3. UML类图关系

UML类图之间的主要关系有6种：继承关系，实现关系，依赖关系，关联关系，聚合关系，组合关系。

**1. 继承关系：**子类和父类之间的关系，可以用带空心三角形的实线表示。

如果一个Teacher类继承上面的Person类：

```java
class Teacher extends Person {

    private int teacherNum;

    public String teach() {

        return "teaching";

    }
}
```

UML图可以表示为：

![](../../assets/类图2.png)

**2. 实现关系：**类和接口之间的关系，可以用带空心三角形的虚线表示。

如下一个Dog类和一个Cat类实现了Animal接口：

```java
public interface Animal {
	public void eat();
}
```

```java
public class Dog implements Animal {
	@Override
	public void eat() {
		System.out.println("狗狗吃狗粮");
	}
}
```

```java
public class Cat implements Animal {
	@Override
	public void eat() {
		System.out.println("咪咪吃猫粮");
	}
}
```

UML可以表示为：

![](../../assets/UML继承.png)

**3. 依赖关系：**



**4. 关联关系：**



**5. 聚合关系：**



**6. 组合关系：**





https://blog.csdn.net/garfielder007/article/details/54427742

