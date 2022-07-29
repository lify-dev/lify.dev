---
title: Spring 中的事件机制
date: 2021-05-12
tags: [sprngboot,event]
categories: [spring]
---


事件机制在一些大型项目中被经常使用，于是 Spring 专门提供了一套事件机制的接口，方便我们运用。本文来说说 ApplicationEventPublisher 的使用。

在设计模式中，观察者模式可以算得上是一个非常经典的行为型设计模式，猫叫了，主人醒了，老鼠跑了，这一经典的例子，是事件驱动模型在设计层面的体现。

另一模式，发布订阅模式往往被人们等同于观察者模式，但我的理解是两者唯一区别，是发布订阅模式需要有一个调度中心，而观察者模式不需要，例如观察者的列表可以直接由被观察者维护。不过两者即使被混用，互相替代，通常不影响表达。

java 和 spring 中都拥有 Event 的抽象，分别代表了语言级别和三方框架级别对事件的支持。

**Spring 的文档对 Event 的支持翻译之后描述如下：**

> ApplicationContext 通过 ApplicationEvent 类和 ApplicationListener 接口进行事件处理。 如果将实现 ApplicationListener 接口的 bean 注入到上下文中，则每次使用 ApplicationContext 发布 ApplicationEvent 时，都会通知该 bean。本质上，这是标准的观察者设计模式。

下面通过 demo，看一个电商系统中对 ApplicationEventPublisher 的使用。

我们的系统要求，当用户注册后，给他发送一封邮件通知他注册成功了。

然后给他初始化积分，发放一张新用户注册优惠券等。

**定义一个用户注册事件:**

```java
public class UserRegisterEvent extends ApplicationEvent{
    public UserRegisterEvent(String name) { //name即source
        super(name);
    }
}
复制代码
```

ApplicationEvent 是由 Spring 提供的所有 Event 类的基类，为了简单起见，注册事件只传递了 name（可以复杂的对象，但注意要了解清楚序列化机制）。

**再定义一个用户注册服务(事件发布者):**

```java
@Service
public class UserService implements ApplicationEventPublisherAware {
    public void register(String name) {
        System.out.println("用户：" + name + " 已注册！");
        applicationEventPublisher.publishEvent(new UserRegisterEvent(name));
    }
    private ApplicationEventPublisher applicationEventPublisher;
    @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
        this.applicationEventPublisher = applicationEventPublisher;
    }
}
复制代码
```

需要注意的是，服务必须交给 Spring 容器托管。ApplicationEventPublisherAware 是由 Spring 提供的用于为 Service 注入 ApplicationEventPublisher 事件发布器的接口，使用这个接口，我们自己的 Service 就拥有了发布事件的能力。用户注册后，不再是显示调用其他的业务 Service，而是发布一个用户注册事件。

**创建邮件服务，积分服务，其他服务(事件订阅者)等:**

```java
@Service
public class EmailService implements ApplicationListener<UserRegisterEvent> {
    @Override
    public void onApplicationEvent(UserRegisterEvent userRegisterEvent) {
        System.out.println("邮件服务接到通知，给 " + userRegisterEvent.getSource() + " 发送邮件...");
    }
}
复制代码
```

事件订阅者的服务同样需要托管于 Spring 容器，ApplicationListener 接口是由 Spring 提供的事件订阅者必须实现的接口，我们一般把该 Service 关心的事件类型作为泛型传入。处理事件，通过 event.getSource() 即可拿到事件的具体内容，在本例中便是用户的姓名。 其他两个 Service，也同样编写，实际的业务操作仅仅是打印一句内容即可，篇幅限制，这里省略。

最后我们使用 Springboot 编写一个启动类。

```java
@SpringBootApplication
@RestController
public class EventDemoApp {
    public static void main(String[] args) {
        SpringApplication.run(EventDemoApp.class, args);
    }
    @Autowired
    UserService userService;
    @RequestMapping("/register")
    public String register(){
        userService.register("xttblog.com");
        return "success";
    }
}
复制代码
```

至此，一个电商系统中简单的事件发布订阅就完成了，后面无论如何扩展，我们只需要添加相应的事件订阅者即可。

