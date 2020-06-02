## Java 多线程

## 1. 创建方式

创建多线程的方式有3种：

	- 继承线程类
	- 实现Runnable接口
	- 匿名类

## 2. 运用示例

### 2.1. 继承线程类

使用多线程，就可以做到盖伦在攻击提莫的**同时**，赏金猎人也在攻击盲僧
设计一个类KillThread **继承`Thread`**，**并且重写run方法**
启动线程办法： 1. 实例化一个KillThread对象。	2.  调用其**start**方法
就可以观察到 赏金猎人攻击盲僧的**同时**，盖伦也在攻击提莫

```java
/* 继承自Thread的KillThread */
package multiplethread;
 
import charactor.Hero;
 
public class KillThread extends Thread{	//继承了线程类
     
    private Hero h1;
    private Hero h2;
 
    public KillThread(Hero h1, Hero h2){	//传入的构造参数
        this.h1 = h1;
        this.h2 = h2;
    }
 
    public void run(){	//重写Run方法
        while(!h2.isDead()){
            h1.attackHero(h2);
        }
    }
}
```

```java
package multiplethread;
 
import charactor.Hero;
 
public class TestThread {
 
    public static void main(String[] args) {
         
        Hero gareen = new Hero();
        gareen.name = "盖伦";
        gareen.hp = 616;
        gareen.damage = 50;
 
        Hero teemo = new Hero();
        teemo.name = "提莫";
        teemo.hp = 300;
        teemo.damage = 30;
         
        Hero bh = new Hero();
        bh.name = "赏金猎人";
        bh.hp = 500;
        bh.damage = 65;
         
        Hero leesin = new Hero();
        leesin.name = "盲僧";
        leesin.hp = 455;
        leesin.damage = 80;
         
        //实例化后，调用start方法
        KillThread killThread1 = new KillThread(gareen,teemo);
        killThread1.start();
        KillThread killThread2 = new KillThread(bh,leesin);
        killThread2.start();
         
    }
     
}
```

### 2.2. 实现Runnable接口

创建类Battle，实现`Runnable`接口
启动的时候，首先创建一个Battle对象，然后再根据该battle对象创建一个线程对象，调用线程的`start()`启动

```java
Battle battle1 = new Battle(gareen,teemo);	//实现了Runnable接口的Battle类
new Thread(battle1).start();	//通过创建线程调用
```

battle1 对象实现了Runnable接口，所以有run方法，但是直接调用run方法，并不会启动一个新的线程。
必须，借助一个线程对象的start()方法，才会启动一个新的线程。
所以，在创建Thread对象的时候，把battle1作为构造方法的参数传递进去，这个线程启动的时候，就会去执行battle1.run()方法了。

```java
/* 实现了Runnable接口的Battle类 */
package multiplethread;
 
import charactor.Hero;
 
public class Battle implements Runnable{
     
    private Hero h1;
    private Hero h2;
 
    public Battle(Hero h1, Hero h2){
        this.h1 = h1;
        this.h2 = h2;
    }
 
    public void run(){
        while(!h2.isDead()){
            h1.attackHero(h2);
        }
    }
}
```

```java
package multiplethread;
 
import charactor.Hero;
 
public class TestThread {
 
    public static void main(String[] args) {
         
        Hero gareen = new Hero();
        gareen.name = "盖伦";
        gareen.hp = 616;
        gareen.damage = 50;
 
        Hero teemo = new Hero();
        teemo.name = "提莫";
        teemo.hp = 300;
        teemo.damage = 30;
         
        Hero bh = new Hero();
        bh.name = "赏金猎人";
        bh.hp = 500;
        bh.damage = 65;
         
        Hero leesin = new Hero();
        leesin.name = "盲僧";
        leesin.hp = 455;
        leesin.damage = 80;
         
        //创建实现了Runnable的类后创建线程对象并调用 start() 
        Battle battle1 = new Battle(gareen,teemo);
        new Thread(battle1).start();
 
        Battle battle2 = new Battle(bh,leesin);
        new Thread(battle2).start();
 
    }
     
}
```

## 2.3. 匿名类实现

使用[匿名类](https://how2j.cn/k/interface-inheritance/interface-inheritance-inner-class/322.html#step687)，继承Thread,重写run方法，直接在run方法中写业务代码
匿名类的一个好处是可以很方便的访问外部的局部变量。
前提是外部的局部变量需要被声明为final。(JDK7以后就不需要了)

```java
package multiplethread;
  
import charactor.Hero;
  
public class TestThread {
  
    public static void main(String[] args) {
          
        Hero gareen = new Hero();
        gareen.name = "盖伦";
        gareen.hp = 616;
        gareen.damage = 50;
  
        Hero teemo = new Hero();
        teemo.name = "提莫";
        teemo.hp = 300;
        teemo.damage = 30;
          
        Hero bh = new Hero();
        bh.name = "赏金猎人";
        bh.hp = 500;
        bh.damage = 65;
          
        Hero leesin = new Hero();
        leesin.name = "盲僧";
        leesin.hp = 455;
        leesin.damage = 80;
          
        //匿名类
        Thread t1= new Thread(){
            public void run(){
                //匿名类中用到外部的局部变量teemo，必须把teemo声明为final
                //但是在JDK7以后，就不是必须加final的了
                while(!teemo.isDead()){
                    gareen.attackHero(teemo);
                }              
            }
        };
         
        t1.start();
          
        Thread t2= new Thread(){
            public void run(){
                while(!leesin.isDead()){
                    bh.attackHero(leesin);
                }              
            }
        };
        t2.start();
         
    }
      
}
```

