---
title: Java之Builder模式
date: 2017-08-11 17:12:22
categories: Java
type: "categories"
comments: false
tags:
- Java

---
### Java Builder 模式
我们一帮在构造一个javaBean对象时,无非有三种写法:
1.直接通过构造函数传参的方式设置属性，这种方法如果属性过多的话会让构造函数十分臃肿，而且不能灵活的选择只设置某些参数。

2.采用重叠构造区模式，先写第一个只有必要参数的构造器，第二个构造器有一个可选参数，第三个构造器有两个可选参数，以此类推；如果参数比较多时，类里面会出现一堆构造方法，并且阅读困难，很容易就把两个属性参数写颠倒位置了，编译不会出错，但运行就会出错了

3.采用Javabean 的写法，写一堆属性的setter方法，通过生成对象，让后调用setter方法给属性赋值。  这种方法有个劣势就是构造的过程被分到几个调用中，在构造中可能处于不一致状态，无法保证一致性。

4.采用Builder 模式构造对象，当一个类的参数多的情况下，使用重叠构造器模式客户端代码会很难编写，并且可读性差；使用javabean模式，调用一个无参的构造器，然后调用setter方法来设置每个必要的参数。但是javabean自身有着严重的缺点，因为构造过程被分到几个调用中，在构造javabean可能处于不一致的状态，类无法仅仅通过检验构造器参数的有效性来保证一致性。另一点不足之处，javabean模式阻止了把类做成不可变的可能，这就需要程序员付出额外的努力来确保他的线程安全； build模式 既能保证像重叠构造器那样的安全，也能实现JavaBean模式那样的可读性。 

使用build模式的步骤： 
(1)不直接生成想要的对象，而是让客户端利用所有必要的参数调用构造器（或者静态工厂），得到一个build对象。 
(2)然后让客户端在build对象上调用类似的setter方法来设置每个相关的可选参数， 
(3)最后，客户端调用无参的build方法来生成不可变的对象。这个builder是它构建的静态成员类。

下面就说一下Builder模式使用,

``` java
package com.dyz.persist.util;

public class TestBuilder {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;
    public int getServingSize() {
        return servingSize;
    }

    public int getServings() {
        return servings;
    }

    public int getCalories() {
        return calories;
    }

    public int getFat() {
        return fat;
    }

    public int getSodium() {
        return sodium;
    }

    public int getCarbohydrate() {
        return carbohydrate;
    }


    public static class Builder {
        private int servingSize = 0;
        private int servings = 0;
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }

        public Builder sodium(int val) {
            sodium = val;
            return this;
        }

        public TestBuilder build() {
            return new TestBuilder(this);
        }
    }

        private TestBuilder(Builder builder){
            servingSize = builder.servingSize;
            servings = builder.servings;
            calories = builder.calories;
            fat = builder.fat;
            sodium = builder.sodium;
            carbohydrate = builder.carbohydrate;
    }

    public static void main(String[] args) {
        TestBuilder builder = new Builder(240,8)
		.calories(100).
		sodium(100).
		carbohydrate(27).
		build();
    }
}```

可以看出Builder是TestBuilder静态的内部类,并且Builder中的属性和TestBuilder中的属性是一致的,所有的属性都在Builder中,TestBuilder中只有获取属性的方法.
使用Builder构造一个TestBuilder对象的写法:
``` java
TestBuilder builder = new Builder(240,8)
		.calories(100).
		sodium(100).
		carbohydrate(27).
		build();
```

采用javaBean 写法的缺点就是， 一但调用 new TestBuilder() 构造函数后，对象就被创建了，以后在调用 set 方法设置属性的时候这里设置一下，其他地方又设置一下，无法保证对象的状态一致性，而且代码的可读性很差

1.Builder 方式创建的对象，在调用 build() 方法之前是不会创建TestBuilder 对象的，所有的属性设置都必须在 build() 方法之前，而且创建了 TestBuilder 对象后就不可以更改其属性了，这就保证了对象状态的唯一性，而且代码的可读性也提高了。
2.如果有些参数是必填的，可以加到 Builder 的构造函数中。
