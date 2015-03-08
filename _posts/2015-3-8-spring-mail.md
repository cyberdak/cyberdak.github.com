---
layout : post
title : 使用spring.mail来发送邮件
category : "java"
tags : [spring,mail,mail]
---

jaxax.mail提供了原生的邮件发送和接受api，spring.mail在此基础上提供了进一层的封装，使用起来更方便。它屏蔽了一些底层的实现和细节，使我们可以用起来可以更方便。

环境：
```
javamail 1.4.4
spring 3.2.3RELEASE
maven
```

首先，在pom.xml添加以下依赖：spring.mail包被整合在spriing-context-support中。
```xml
<!-- javamail API --> 
 <dependency>
      <groupId>javax.mail</groupId>
      <artifactId>mail</artifactId>
      <version>1.4.4</version>
</dependency>
<!-- spring framework -->
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-context-support</artifactId>
	<version>3.2.3.RELEASE</version>
</dependency>
```

在resrouces下面添加spring-mail.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop" xmlns:c="http://www.springframework.org/schema/c" xmlns:cache="http://www.springframework.org/schema/cache" xmlns:context="http://www.springframework.org/schema/context" xmlns:jdbc="http://www.springframework.org/schema/jdbc" xmlns:jee="http://www.springframework.org/schema/jee" xmlns:lang="http://www.springframework.org/schema/lang" xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:p="http://www.springframework.org/schema/p" xmlns:task="http://www.springframework.org/schema/task" xmlns:tx="http://www.springframework.org/schema/tx" xmlns:util="http://www.springframework.org/schema/util"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
		http://www.springframework.org/schema/cache http://www.springframework.org/schema/cache/spring-cache.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
		http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc.xsd
		http://www.springframework.org/schema/jee http://www.springframework.org/schema/jee/spring-jee.xsd
		http://www.springframework.org/schema/lang http://www.springframework.org/schema/lang/spring-lang.xsd
		http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
		http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task.xsd
		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
		http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd">

    <bean id="javaMailSender" class="org.springframework.mail.javamail.JavaMailSenderImpl">
        <!-- 服务器 -->
        <property name="host" value="${mail.smtp.host}" />
        <!-- 端口号 -->
        <property name="port" value="${mail.smtp.port}" />
        <!-- 用户名 -->
        <property name="username" value="${mail.smtp.username}" />
        <!--  密码   -->
        <property name="password" value="${mail.smtp.password}" />
        <!-- SMTP服务器验证 -->
        <property name="javaMailProperties">
            <props>
               <!-- 验证身份 -->
               <prop key="mail.smtp.auth">${mail.smtp.auth}</prop>
            </props>
        </property>
    </bean>
    
    <bean id="simpleMailMessage" class="org.springframework.mail.SimpleMailMessage">
        <!-- 发件人email -->
        <property name="from" value="${mail.smtp.from}" />
        <!-- 
         收件人email
        <property name="to" value="to@yeah.net" />
        email主题(标题)
        <property name="subject" value="Subject" />
        email主题内容
        <property name="text">
          <value>ContentText</value>
        </property>
        -->
    </bean>
    
    <bean id="mailService" class="com.cyberdak.mail.MailService">
        <property name="javaMailSender" ref="javaMailSender" />
        <property name="simpleMailMessage" ref="simpleMailMessage" />
    </bean>

</beans>
```

config.properties添加smtp属性
```properties
## mail
##smtp服务器地址
mail.smtp.host=smtp.qq.com
##smtp服务器端口
mail.smtp.port=25
##帐号
mail.smtp.username=21936354
##发件人
mail.smtp.from=21936354@qq.com
##密码
mail.smtp.password=*********
##是否需要认证
mail.smtp.auth=true
```

新建文件MailService.java.MailService就是直接最顶层的api，接收附件文件列表，标题，内容以及收件人列表。

```java
package com.cyberdak.mail;
import java.io.File;

import javax.mail.MessagingException;
import javax.mail.internet.MimeMessage;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.core.io.FileSystemResource;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;

/**
 * @author 亢伟楠
 *	时间:2015年3月4日下午9:38:55
 */
public class MailService {
    private JavaMailSender javaMailSender;
    private SimpleMailMessage simpleMailMessage;
    private static final Logger logger = LoggerFactory.getLogger(MailService.class);
	
	public void sendMail(String[] filePaths,String title,String content,String... receivers) throws MessagingException{
		// 建立邮件消息,发送简单邮件和html邮件的区别
		MimeMessage mailMessage = javaMailSender.createMimeMessage();
		// 注意这里的boolean,等于真的时候才能嵌套图片，在构建MimeMessageHelper时候，所给定的值是true表示启用，
		// multipart模式 为true时发送附件 可以设置html格式
		MimeMessageHelper messageHelper = new MimeMessageHelper(mailMessage,
				true, "utf-8");

		// 设置收件人，寄件人
		messageHelper.setTo(receivers);
		messageHelper.setFrom(simpleMailMessage.getFrom());
		messageHelper.setSubject(title);
		// true 表示启动HTML格式的邮件
		messageHelper.setText(
				"<html><head></head><body><h1>"+content+"</h1></body></html>",
				true);
		for(String filePath : filePaths){
			FileSystemResource file = new FileSystemResource(
					new File(filePath));//"e:/sample1422434681333.jpg"
			messageHelper.addAttachment(file.getFilename(), file);
		}
		
		// 发送邮件
		javaMailSender.send(mailMessage);

		logger.info("邮件发送成功..");
	}
	
	//getter & setter
	public JavaMailSender getJavaMailSender() {
		return javaMailSender;
	}
	public void setJavaMailSender(JavaMailSender javaMailSender) {
		this.javaMailSender = javaMailSender;
	}
	public SimpleMailMessage getSimpleMailMessage() {
		return simpleMailMessage;
	}
	public void setSimpleMailMessage(SimpleMailMessage simpleMailMessage) {
		this.simpleMailMessage = simpleMailMessage;
	}
	
	
}

```

使用:
```java

	@Autowired
	private MailService mailService;

	public void sendMail(){
    	mailService.sendMail(new String[]{"e:/sample1422434681333.jpg"}, "title", "test", "21936354@qq.com","kangweinan@huaxinzhi.com");
    }
```
