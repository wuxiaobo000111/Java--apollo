转载于 https://blog.csdn.net/u013360850/article/details/78822860

# 概述

这篇文章介绍一下如何在spring boot项目中使用email发送的功能。使用的是spring-boot-starter-mail组件。如下先给出依赖。

maven依赖

```xml
<!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-mail -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
    <version>1.5.7.RELEASE</version>
</dependency>

```


gradle依赖

```gradle
compile group: 'org.springframework.boot', name: 'spring-boot-starter-mail', version: '1.5.7.RELEASE'

```


加入如下配置

```yml
spring:
  profiles: dev
  mail:
    host: smtp.exmail.qq.com
    # 发送人的邮箱地址
    username: 
    # 发送人的邮箱密码
    password: 
    properties:
      smtp:
        starttls:
          enable: true
          required: true
      mail:
        smtp:
          ssl:
            enable: true
```

配置完成之后需要引入一个新的bean

```java
@Component
public class MailUtil {

    private final Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private JavaMailSender javaMailSender;

    public void sendDidaEmail(String[] receiver, String subject, String content) throws Exception {
        String sendName = LoadPropertyUtil.getProperty("spring.mail.username");
        sendEmail(sendName, receiver, null, subject, content);
    }


    public void sendEmail(String deliver, String[] receiver, String[] carbonCopy, String subject, String content) throws Exception {
        SimpleMailMessage message = new SimpleMailMessage();
        try {
            // 发送人的邮箱地址
            message.setFrom(deliver);
            // 接收人的邮箱地址,是一个字符串数据。
            message.setTo(receiver);
            // 抄送人的邮箱地址
            if (CheckObjUtil.isNotEmpty(carbonCopy)) {
                message.setCc(carbonCopy);
            }
            message.setSubject(subject);
            message.setText(content);
            javaMailSender.send(message);
        } catch (MailException e) {
            logger.error("Send mail failed, error, message:{}", JsonMapper.toJson(message), e);
            throw new Exception(e.getMessage());
        }
    }

}

```


因为在项目中是使用最简单的发送邮箱的功能,所以其他的功能也没有进行多少研究,下面的内容基本上来自于https://blog.csdn.net/u013360850/article/details/78822860这个博客地址。


## 发送HTML邮件

```java
    import cn.com.hellowood.mail.util.ServiceException;
    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.mail.javamail.JavaMailSender;
    import org.springframework.mail.javamail.MimeMessageHelper;
    import org.springframework.stereotype.Component;
    
    import javax.mail.MessagingException;
    import javax.mail.internet.MimeMessage;
    
    @Component
    public class MailUtil {
    
        private final Logger logger = LoggerFactory.getLogger(getClass());
    
        @Autowired
        JavaMailSender mailSender;
    
        public void sendHtmlEmail(String deliver, String[] receiver, String[] carbonCopy, String subject, String content, boolean isHtml) throws ServiceException {
            long startTimestamp = System.currentTimeMillis();
            logger.info("Start send email ...");
    
            try {
                MimeMessage message = mailSender.createMimeMessage();
                MimeMessageHelper messageHelper = new MimeMessageHelper(message);
    
                messageHelper.setFrom(deliver);
                messageHelper.setTo(receiver);
                messageHelper.setCc(carbonCopy);
                messageHelper.setSubject(subject);
                messageHelper.setText(content, isHtml);
    
                mailSender.send(message);
                logger.info("Send email success, cost {} million seconds", System.currentTimeMillis() - startTimestamp);
            } catch (MessagingException e) {
                logger.error("Send email failed, error message is {} \n", e.getMessage());
                e.printStackTrace();
                throw new ServiceException(e.getMessage());
            }
        }
    
    }


```


## 发送带附件的邮件


```java
    
    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.mail.javamail.JavaMailSender;
    import org.springframework.mail.javamail.MimeMessageHelper;
    import org.springframework.stereotype.Component;
    
    import javax.mail.MessagingException;
    import javax.mail.internet.MimeMessage;
    import java.io.File;
    
    @Component
    public class MailUtil {
    
        private final Logger logger = LoggerFactory.getLogger(getClass());
    
        @Autowired
        JavaMailSender mailSender;
    
        public void sendAttachmentFileEmail(String deliver, String[] receiver, String[] carbonCopy, String subject, String content, boolean isHtml, String fileName, File file) throws ServiceException {
            long startTimestamp = System.currentTimeMillis();
            logger.info("Start send mail ...");
    
            try {
                MimeMessage message = mailSender.createMimeMessage();
                MimeMessageHelper messageHelper = new MimeMessageHelper(message, true);
    
                messageHelper.setFrom(deliver);
                messageHelper.setTo(receiver);
                messageHelper.setCc(carbonCopy);
                messageHelper.setSubject(subject);
                messageHelper.setText(content, isHtml);
                messageHelper.addAttachment(fileName, file);
    
                mailSender.send(message);
                logger.info("Send mail success, cost {} million seconds", System.currentTimeMillis() - startTimestamp);
            } catch (MessagingException e) {
                logger.error("Send mail failed, error message is {}\n", e.getMessage());
                e.printStackTrace();
                throw new ServiceException(e.getMessage());
            }
        }
        
    }


```