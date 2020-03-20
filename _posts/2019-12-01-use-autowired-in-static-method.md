---
layout: post
title: "Java 中静态方法调用 Autowired"
subtitle: "Use autowired in static method with Autowired"
author: "qingshan"
header-img: "img/about-bg-walle.jpg"
header-mask: 0.4
tags:

- 工作
- Java

---
某个 Java 项目中使用 JPA 作为数据库操作层，因此在业务逻辑中不可避免要使用`@Autowired`来引入`Repository`来操作数据库。但是近期，有个简单的需求，即希望通过接口触发数据库数据的实时读取和写入。这个需求本来没什么困难的，直接新建一个 Web 路由，调用已经写好的`Repository`相关业务逻辑即可。但是实际上行不通。原因是，如果这么做了，那么`Repository`这里语句一定返回的是空，并触发程序报错。经过查找资料，发现这个问题被称为“静态方法中直接使用注入的 bean 对象”的问题。

# 1 根因

静态方法是属于类的，普通方法才属于对象，Spring 注入是在容器中实例化变量的，并且静态是优先于对象存在的，所以直接在静态方法中调用注入的静态变量其实是为 null 的。

出错的代码示例：

```java
@Component
public class BigTaskManager {

    @Autowired
    private ApplicationRepository applicationRepository;

    private Integer getActionId(String appId) {
        List<Application> items = applicationRepository.findByAppid(appId);
        if (items.isEmpty()) {
            return 0;
        } else {
            return items.get(0).getAction_id();
        }
    }
}


@RestController
@EnableAutoConfiguration
@RequestMapping("/tasks")
public class TaskManagerController {
    private static final Logger logger = LoggerFactory.getLogger(TaskManagerController.class);

    @RequestMapping("/yarn/running")
    public List<Map<String, Object>> listTasks() {
        BigTaskManager bm = new BigTaskManager();
        List<Map<String, Object>> TasksDetail = bm.collectTaskInfo();
        return TasksDetail;
    }
}
```

# 2 解决方法一

@PostConstruct 是 Java EE 5 引入来影响 Servlet 生命周期的注解，被用来修饰非静态的 void()方法，@PostConstruct 在构造函数之后执行,init()方法之前执行。

改造后的代码

```java
@Component
public class BigTaskManager {

    @Autowired
    private ApplicationRepository applicationRepository;

    @PostConstruct
    public void init() {
        bigTaskManager = this;
        bigTaskManager.applicationRepository = this.applicationRepository;
    }

    public Integer getActionId(String appId) {
        List<Application> items = bigTaskManager.applicationRepository.findByAppid(appId);
        if (items.isEmpty()) {
            return 0;
        } else {
            return items.get(0).getAction_id();
        }
    }
}
```

# 3 解决方法二 （实践没有成功）

@Autowired 用在构造函数上
@Autowired 注释，可以对类成员变量、方法及构造函数进行标注，完成自动装配的工作，此种方式就是在构造函数上使用@Autowired。

```java
@Component
public class BigTaskManager {

    private ApplicationRepository applicationRepository;

    @Autowired
    public BigTaskManager(ApplicationRepository applicationRepository) {
       BigTaskManager.applicationRepository = applicationRepository; 
    }

    public Integer getActionId(String appId) {
        List<Application> items = bigTaskManager.applicationRepository.findByAppid(appId);
        if (items.isEmpty()) {
            return 0;
        } else {
            return items.get(0).getAction_id();
        }
    }
}
```