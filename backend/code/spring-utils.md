## 全局Spring工具类
```java
package com.example.utils;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.ListableBeanFactory;
import org.springframework.beans.factory.config.BeanFactoryPostProcessor;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

/**
 * spring工具类 方便在非spring管理环境中获取bean
 * <p>
 * DataSource ds = SpringUtils.getBean("masterDataSource");
 *
 * @author chenzz
 */
@Component
public final class SpringUtils implements BeanFactoryPostProcessor, ApplicationContextAware {

    private static ConfigurableListableBeanFactory beanFactory;

    private static ApplicationContext applicationContext;

    @SuppressWarnings("NullableProblems")
    public static <T> T getBean(String name) throws BeansException {
        return (T) beanFactory.getBean(name);
    }

    public static <T> T getBean(Class<T> clz) throws BeansException {
        return beanFactory.getBean(clz);
    }

    /**
	 * 获取{@link ListableBeanFactory}，可能为{@link ConfigurableListableBeanFactory} 或 {@link ApplicationContextAware}
	 *
	 * @return {@link ListableBeanFactory}
	 */
	public static ListableBeanFactory getBeanFactory() {
		final ListableBeanFactory factory =  null == beanFactory ? applicationContext : beanFactory;
		if(null == factory){
            // Can be modified to UtilException
			throw new RuntimeException("No ConfigurableListableBeanFactory or ApplicationContext injected, maybe not in the Spring environment?");
		}
		return factory;
	}

    /**
	 * 获取配置文件配置项的值
	 *
	 * @param key 配置项key
	 * @return 属性值
	 */
	public static String getProperty(String key) {
		if (null == applicationContext) {
			return null;
		}
		return applicationContext.getEnvironment().getProperty(key);
	}

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        SpringUtils.beanFactory = beanFactory;
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        SpringUtils.applicationContext = applicationContext;
    }

}
```

## spring boot打印访问地址, 端口
```java
import org.springframework.core.env.Environment;
import java.net.InetAddress;

@Autowired
private ApplicationContext context;

// ----------------------
Environment environment = context.getBean(Environment.class);
// 应用的上下文路径，也可以称为项目路径
String path = environment.getProperty("server.servlet.context-path");
log.info("\n访问地址: http://{}:{}{}", InetAddress.getLocalHost().getHostAddress(), environment.getProperty("server.port"), path);
```

## spring boot打印所有bean
```java
// 方法1
@Bean
public CommandLineRunner commandLineRunner(ApplicationContext ctx) {
    return args -> {
        System.out.println("Let's inspect the beans provided by Spring Boot:");
        String[] beanNames = ctx.getBeanDefinitionNames();
        Arrays.sort(beanNames);
        for (String beanName : beanNames) {
            System.out.println(beanName);
        }
        System.out.println("----------end-------------");
    };
}

// 方法2
var ctx = SpringApplication.run(CcApplication.class, args);
for (var name : ctx.getBeanDefinitionNames()) {
    System.out.println(name);
}
```

## spring boot获取配置
```java
// 方法1
var ctx = SpringApplication.run(SpringDemoApplication.class, args);
var env = ctx.getEnvironment();
String port = env.getProperty("server.port", "");

// 方法2
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.core.env.Environment;

import java.time.LocalDateTime;

private final Logger log = LoggerFactory.getLogger(this.getClass());

@Autowired
private ApplicationContext context;

@Test
void testEnvironment() {
    Environment env = context.getBean(Environment.class);
    log.debug("----env:{}----", env.getProperty("spring.application.name"));
    log.debug("------{}------", LocalDateTime.now());

    // Environment env = SpringUtils.getBean(Environment.class);
    // env.getProperty("spring.profiles.active");
}
```

## spring事务注解
```java
// import javax.transaction.Transactional;
// 建议用spring的注解
import org.springframework.transaction.annotation.Transactional;

@Transactional(rollbackFor = Exception.class)
```

> AopContext
```java
import org.springframework.aop.framework.AopContext;

/**
 * 获取aop代理对象
 */
@SuppressWarnings("unchecked")
public static <T> T getAopProxy(T invoker) {
    return (T) AopContext.currentProxy();
}
```