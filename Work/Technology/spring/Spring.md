## 个人认识

Spring 是 JAVA EE 中高内聚、低耦合的最佳实践



## 知识点

### BEAN 的生命周期

1. InstantiationAwareBeanPostProcessor.postProcessBeforeInstantiation()
2. new Instance
3. *.postProcessAfterInstantiation()
4. *.postProcessPropertyValues()
5. set Values
6. BeanNameAware.setBeanName()
7. BeanFactoryAware.setBeanFactory()
8. BeanPostProcessor.postProcessBeforeInstantiation(Object bean, String beanName)
9. init-method
10. *.postProcessAfterInstantiation()
11. return Bean(Prototype) / **input Spring IOC cache pool** and return Bean Ref (Singleton)
12. destory-method (Singleton)

#### Bean 自身的方法

1. Setter
2. init-method
3. destory-method

#### Bean 级生命周期的接口

1. BeanNameAware
2. BeanFactoryAware

#### 容器级生命周期的接口

1. InstantiationAwareBeanPostProcessor
2. BeanPostProcessor

### Spring MVC 的初始化过程

1. Servlet 容器初始化会产生 **ServletContext**
2. 在 web.xml 中配置的 ContextLoaderListener 监听 ServletContext 的创建
3. Spring 创建 WebApplicationContext ( XmlWebApplicationContext ) 为 根上下文 ( Root Context )
4. **并将根上下文以 WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE 为 key，设置到 SevletContext 中**
5. 加载配置文件中的 DispatcherServlet 并创建自己的 IOC 容器 (XmlWebApplicationContext)，**从 ServletContext 中获取 root context 并设置为 parent**
6. 初始化路径映射，视图解析等
7. 将**包含但不止** DispatcherServlet 的 Context 放到 ServletContext 中去

## 参考链接

[Spring IOC 原理解析](http://www.360doc.com/content/17/0917/07/2708086_687778967.shtml) ★★★

[Spring MVC 启动过程](https://www.cnblogs.com/zhangminghui/p/4912631.html) ★★★

[Spring MVC 工作流程](http://blog.csdn.net/james_shu/article/details/54616120) ★★★



