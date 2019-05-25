---
title: "@Bean的用法"
date: 2019-05-20 11:29:21
tags: [编程，感悟]
categories: 开发中常用注解
---

@Bean是一个方法级别上的注解，主要用在@Configuration注解的类里，也可以用在@Component注解的类里，添加的bean的id为方法名。<!--more-->

### 案例
下面是@Configuration里的一个例子。

```java
	@Configuration
	public class Configure {
	
	    @Bean
	    public FilterRegistrationBean registLog() {
	        FilterRegistrationBean loggingFilter = new FilterRegistrationBean(new LoggingFilter());
	        loggingFilter.addUrlPatterns("/*");
	        loggingFilter.setName("LoggingFilter");
	        loggingFilter.setOrder(5);
	        return loggingFilter;
	    }
	
	}
```

这个配置就等同于之前在xml里的配置

```xml
<beans>
	<bean id="registLog" class="org.springframework.boot.web.servlet.FilterRegistrationBean"/>
</beans>
```

