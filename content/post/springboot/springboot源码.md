---
title: "Springboot源码"
date: 2021-07-27T19:59:52+08:00
lastmod: 2021-07-27T19:59:52+08:00
draft: false
keywords: ["springboot","springboot 源码"]
description: "springboot 源码学习"
tags: ["springboot"]
categories: ["springboot"]
author: "Kyle Zhao"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: false
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: true
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

---

Springboot 源码阅读

springboot 最大的特点 就是自动配置，我们引入 对应的 jar 包，springboot 帮助我们进行自动配置，可以省去很多繁琐的配置步骤。

<!--more-->

> 启动类

```java
/**
 * @author kyle
 */
@SpringBootApplication
public class SpringbootRestDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringbootRestDemoApplication.class, args);
    }
}
```

> @SpringBootApplication

该注解由 `SpringBootConfiguration`, `EnableAutoConfiguration`,`ComponentScan` 三个注解组成

@SpringBootConfiguration 

标明这是 springboot application 配置类, 以下几种方法排除自动配置类。

```
// 排除自动配置的类
1. @SpringBootConfiguration (exclude = {DataSourceAutoConfiguration.class})
2. @SpringBootApplication(excludeName = {"org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration"})
3. @EnableAutoConfiguration
(exclude = {DataSourceAutoConfiguration.class})
4. @EnableAutoConfiguration
(excludeName = {"org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration"})
5. 配置文件
spring:     
  autoconfigure:
    exclude:
      - org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
      - org.springframework.boot.autoconfigure.mail.MailSenderAutoConfiguration
```

@ComponentScan

扫描该路径下的 类，注入的容器中，默认扫描 SpringbootRestDemoApplication 该包路径下的所有类

@EnableAutoConfiguration

启动自动配置

```
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration
```

@AutoConfigurationPackage

表明包含该注解的类包应该注册

@Import(AutoConfigurationImportSelector.class)

```java
META-INF/spring-autoconfigure-metadata.properties
AutoConfigurationImportSelector.class

	protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
		List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),
				getBeanClassLoader());
		Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you "
				+ "are using a custom packaging, make sure that file is correct.");
		return configurations;
	}

	protected Class<?> getSpringFactoriesLoaderFactoryClass() {
		return EnableAutoConfiguration.class;
	}

SpringFactoriesLoader.class
    
	public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
		String factoryTypeName = factoryType.getName();
		return loadSpringFactories(classLoader).getOrDefault(factoryTypeName, Collections.emptyList());
	}

	// 
	private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
		MultiValueMap<String, String> result = cache.get(classLoader);
		if (result != null) {
			return result;
		}

		try {
            // public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories"
			Enumeration<URL> urls = (classLoader != null ?
					classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
					ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
			result = new LinkedMultiValueMap<>();
			while (urls.hasMoreElements()) {
				URL url = urls.nextElement();
				UrlResource resource = new UrlResource(url);
				Properties properties = PropertiesLoaderUtils.loadProperties(resource);
				for (Map.Entry<?, ?> entry : properties.entrySet()) {
					String factoryTypeName = ((String) entry.getKey()).trim();
					for (String factoryImplementationName : StringUtils.commaDelimitedListToStringArray((String) entry.getValue())) {
						result.add(factoryTypeName, factoryImplementationName.trim());
					}
				}
			}
			cache.put(classLoader, result);
			return result;
		}
		catch (IOException ex) {
			throw new IllegalArgumentException("Unable to load factories from location [" +
					FACTORIES_RESOURCE_LOCATION + "]", ex);
		}
	}

META-INF/spring.factories
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
```