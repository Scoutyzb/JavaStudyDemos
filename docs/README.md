<link href="http://cdn.bootcss.com/highlight.js/8.0/styles/monokai_sublime.min.css" rel="stylesheet">  
<script src="http://cdn.bootcss.com/highlight.js/8.0/highlight.min.js"></script>  
<script >hljs.initHighlightingOnLoad();</script>  
#Spring核心
---
Spring两个核心特性（DI,AOP），DI指Dependency Injection，依赖注入，AOP指Aspect oriented programming，面向切面编程。


##Spring之旅
### 1.1 简化JAVA开发 ###
主要使用以下四种关键方式： 
  
1.	基于 POJO 的轻量级和最小侵入性编程； 
2.	通过依赖注入和面向接口实现松耦合；
3.	基于切面和惯例进行声明式编程；
4.	通过切面和模板减少样板式代码。

#### 激发pojo潜能 ####
很多框架通过强迫应用继承它们的类或实现它们的接口从而导致应用与框架绑死。
POJO意味着Spring组件不要求其实现某些类或接口  

```java 
public class HelloWorldBean｛ 
	public String sayHello() {
	return "Hello World";
	}
}
```


#### 依赖注入 ####
依赖注入已变成一种设计模式  
**每个对象负责管理与自己相互协作的对象（即它所依赖的对象）的引用，这将会导致高度耦合和难以测试的代码。**

```java
package sia.knights;

public class DamselRescuingKnight implements Knight {

  private RescueDamselQuest quest;

  public DamselRescuingKnight() {
    this.quest = new RescueDamselQuest();
  }

  public void embarkOnQuest() {
    quest.embark();
  }
  
```  

`quest`对象由`DamselRescuingKnight`自行创建，如果需要更改quest的种类，将十分困难。

> 更糟糕的是，为这个 DamselRescuingKnight 编写单元测试将出奇地困难。在这样的一个测试中，你必须保证当骑士的 embarkOnQuest() 方法被调用的时候，探险的 embark() 方法也要被调用。但是没有一个简单明了的方式能够实现这一点。很遗憾，DamselRescuingKnight 将无法进行测试。

没看懂

```java
package sia.knights;
  
public class BraveKnight implements Knight {

  private Quest quest;

  public BraveKnight(Quest quest) {
    this.quest = quest;
  }

  public void embarkOnQuest() {
    quest.embark();
  }

}
``` 

这个骑士能完成所有任务，因为任务是构造器所传入的。除此之外还有setter构造器等方式注入  

即，松耦合。


```java
package sia.knights;
import static org.mockito.Mockito.*;

import org.junit.Test;

import sia.knights.BraveKnight;
import sia.knights.Quest;

public class BraveKnightTest {

  @Test
  public void knightShouldEmbarkOnQuest() {
    Quest mockQuest = mock(Quest.class);
    BraveKnight knight = new BraveKnight(mockQuest);
    knight.embarkOnQuest();
    verify(mockQuest, times(1)).embark();
  }

}
```

在上述强耦合过程中，必须实现RescueDamselQuest的embark方法才能进行测试，这非常困难，而实际单元测试中，应仅仅关注此类，而在Spring中，你可以要求 Mockito 框架验证 Quest 的 mock 实现的 embark() 方法仅仅被调用了一次。通过此种方式来证明Embark方法的使用。

如何将一个Quest交给一个Knight呢？创建应用组件之间协作的行为通常称为装配（wiring）
> Spring 有多种装配 bean 的方式，采用 XML 是很常见的一种装配方式。以下是一个简单的 Spring 配置文件：knights.xml，该配置文件将 BraveKnight、SlayDragonQuest 和 PrintStream 装配到了 一起。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="
    http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean id="knight" class="sia.knights.BraveKnight">
    <constructor-arg ref="quest" />
  </bean>

  <bean id="quest" class="sia.knights.SlayDragonQuest">
    <constructor-arg value="#{T(System).out}" />
  </bean>

</beans>
```

```java
package sia.knights.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import sia.knights.BraveKnight;
import sia.knights.Knight;
import sia.knights.Quest;
import sia.knights.SlayDragonQuest;

@Configuration
public class KnightConfig {

  @Bean
  public Knight knight() {
    return new BraveKnight(quest());
  }
  
  @Bean
  public Quest quest() {
    return new SlayDragonQuest(System.out);
  }

}
```
这是两种装配方式，一种`xml`一种`java`，`<constructor-arg ref="quest" />`意味着构造器注入。（也许也是set注入）
`SlayDragonQuest`在此的注入方法是一个stream。
**所以System.out是一个`stream`**

```java
程序清单 1.8 KnightMain.java 加载包含 Knight 的 Spring 上下文
package sia.knights;
​
import org.springframework.context.support.ClassPathXmlApplicationContext;
​
public class KnightMain {
​
  public static void main(String[] args) throws Exception {
    ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("META-INF/spring/knight.xml");
    Knight knight = context.getBean(Knight.class);
    knight.embarkOnQuest();
    context.close();
  }
​
}
```
启动knight，可以看到完全没有出现quest，因为根据它已经根据Context自动装配好了