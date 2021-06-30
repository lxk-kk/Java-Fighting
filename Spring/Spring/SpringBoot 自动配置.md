#### [SpringBoot 自动配置](https://juejin.im/post/6844903812788912141)

##### 1. 启动类

```java
@SpringBootApplication
public class ServiceConsumerApplication {
	public static void main(String[] args) {
		SpringApplication.run(ServiceConsumerApplication.class, args);
	}
}
```

##### 2. 注解 SpringBootApplication

```java
@SpringBootConfiguration /*表明：被注解的类用来提供一个 Spring Boot 应用*/
@EnableAutoConfiguration /*启动自动配置*/
@ComponentScan(			 /*扫描包*/
    excludeFilters = {
      	@Filter( type = FilterType.CUSTOM, classes = {TypeExcludeFilter.class} ), 
      	@Filter( type = FilterType.CUSTOM, classes = {AutoConfigurationExcludeFilter.class})
    }
)
public @interface SpringBootApplication {...}
```

##### 3. 注解 EnableAutoConfiguration

```
启动 Spring 容器的自动配置，默认配置 Spring 应用程序中可能需要的 bean 对象，并注入到 Spring 容器中。
如果开发者在程序中自定义了配置项，SpringBoot 将优先选择 开发者自定义的配置，而不会采用默认的自动配置。
可以通过 @exclude 注解手动排除 不需要的配置，若无法直接处理，则可以使用 @excludeName 指定配置名进行排除
```

```java
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class}) /*自动配置：导入选择器*/
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";
    Class<?>[] exclude() default {};
    String[] excludeName() default {};
}
```

##### 4. AutoConfigurationImportSelector 选择器

```
往 Spring 容器总导入 组件
```

```java
// 为 Spring 容器导入组件
@Override
public String[] selectImports(AnnotationMetadata annotationMetadata) {
  if (!isEnabled(annotationMetadata)) {
    return NO_IMPORTS;
  }
  AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);
  return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
}
```





