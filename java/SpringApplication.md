SpringApplication

`SpringApplication` spring Boot驱动Spring应用上下文的引导类

`ConfigFileApplicationListener` 监听`ApplicaitionEnvironmentPreparedEvent`事件

从而加载`application.properties`或者`application.yaml`文件

`SpringApplication`利用

- Spring应用上下文（`ApplicationContext`）生命周期控制  注解驱动Bean
- Spring事件/监听（`ApplicationEventMulticaster`）机制加载或者初始化组件

