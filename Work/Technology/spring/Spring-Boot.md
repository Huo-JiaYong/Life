## 知识点

1. 默认的 java.version = 1.6 （为保证最低版本都能运行）

   ```xml
   <properties>
   	<java.version>1.8</java.version>
   </properties>
   ```

2. 如何不使用 spring-boot-partent 实现 spring-boot-parent 的依赖管理功能（因为 parent 是唯一的，可能项目需要指定其他 parent 标签）

   ```xml
   <dependencyManagement>
   	<dependencies>
         <dependency>
         	<groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-dependencies</artifactId>
           <version>2.0.0.RELEASE</version>
           <scope>import</scope>
         </dependency>
     	</dependencies>
   </dependencyManagement>
   ```

   若需要指定 spring-boot-dependencies 中某个依赖的版本，需要在 spring-boot-dependencies 之前定义

   ```xml
   <!-- 指定 spring-boot-redis 版本为  1.5.0.RELEASE -->
   <dependencyManagement>
     <dependency>
     	<groupId>org.springframework.data</groupId>
       <artifactId>spring-boot-redis</artifactId>
       <version>1.5.0.RELEASE</version>
       <scope>import</scope>
     </dependency>
     <!-- 同上 -->
   </dependencyManagement>
   ```

3. Spring Boot自动配置（auto-configuration）尝试根据添加的jar依赖自动配置你的Spring应用。

   实现自动配置有两种可选方式，

   1. `@EnableAutoConfiguration`
   2. `@SpringBootApplication`注解到`@Configuration`类上

   **注**：你应该只添加一个`@EnableAutoConfiguration`注解，通常建议将它添加到主配置类（primary `@Configuration`)

4. 