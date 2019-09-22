---
layout: post
title: "SpringBoot集成邮件发送"
date: 2019-09-22
tag: 框架
---   

### 邮件集成文档
SpringBoot集成邮件模块官网文档描述十分简洁，如下图

![image.png](/images/20190922.png)
SpringBoot集成邮件链接：[https://docs.spring.io/spring-boot/docs/2.1.8.RELEASE/reference/html/boot-features-email.html](https://docs.spring.io/spring-boot/docs/2.1.8.RELEASE/reference/html/boot-features-email.html)
>文档中写明了SpringBoot是集成了Spring的邮件服务，我们直接参考Spring的邮件发送操作即可

Spring邮件链接：[https://docs.spring.io/spring/docs/5.1.9.RELEASE/spring-framework-reference/integration.html#mail](https://docs.spring.io/spring/docs/5.1.9.RELEASE/spring-framework-reference/integration.html#mail)

### 配置依赖和参数
在`pom.xml`加入依赖
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```
Spring-boot支持配置一个spring-boot的parent版本，里面有spring-boot的版本号，其他依赖可以不写版本号，spring-boot可以自动帮着配置合适的版本

在`application.properties`中配置邮箱服务器
```
spring.mail.host=smtp.qq.com
spring.mail.username=1010xxxxxxx@qq.com
spring.mail.password=xxxxxx //发送方的邮箱授权码
spring.mail.default-encoding=UTF-8
mail.fromMail=1010xxxxxx@qq.com
```
### 发送简单邮件
```
@Service
public class SimpleMail {

    @Autowired
    private MailSender mailSender;

    public void sendMail() {
        SimpleMailMessage msg = new SimpleMailMessage();
        msg.setFrom("1010xxxxxx@qq.com");
        msg.setTo("1010xxxxxx@qq.com");
        msg.setSubject("普通邮件主题");
        msg.setText("普通邮件正文");
        System.out.println(msg.toString());
        try{
            mailSender.send(msg);
        }catch (MailException ex) {
            System.err.println(ex.getMessage());
        }
    }
}
```
简单邮件使用的是MailSender接口，该接口直接配置成`@Autowired`spring-boot会自动根据配置文件初始化实现类注入，直接拿来send`SimpleMailMessage`即可
### Mime邮件
```
@Service
public class MimeMail {

    @Autowired
    private JavaMailSender javaMailSender;

    public void sendMail() {
        try {
            MimeMessage message = javaMailSender.createMimeMessage();
            MimeMessageHelper helper = new MimeMessageHelper(message);
            helper.setFrom("1010xxxxxx@qq.com");
            helper.setTo("1010xxxxxx@qq.com");
            helper.setSubject("Mime邮件主题");
            helper.setText("Mime邮件正文");
            javaMailSender.send(message);
        }catch (MessagingException e) {
            e.printStackTrace();
        }

    }
}
```
Mime可以发送富文本（附件，图片，html等），使用`MimeMessage`和`JavaMailSender`接口，由于·MimeMessage·比`SimpleMailMessage`内容复杂，所以一般用`MimeMessageHelper`来组织邮件内容，简化了set，get过程
### Mime邮件支持的附件
```
@Service
public class AttachmentsMail {

    @Autowired
    private JavaMailSender javaMailSender;

    public void sendMail() {
        try {
            MimeMessage message = javaMailSender.createMimeMessage();
            //是否发送的邮件是富文本（附件，图片，html等）
            MimeMessageHelper helper = new MimeMessageHelper(message,true);
            helper.setFrom("1010xxxxxx@qq.com");
            helper.setTo("1010xxxxxx@163.com");
            helper.setSubject("附件和Html邮件主题");
            helper.setText("有附件，注意查收！");
            //增加一个附件
            FileSystemResource file = new FileSystemResource(new File("C:\\Users\\a\\Desktop\\test.excel"));
            helper.addAttachment("test.excel", file);
            javaMailSender.send(message);
        }catch (MessagingException e) {
            e.printStackTrace();
        }

    }
}
```
### Mime邮件支持的内嵌内容
```
@Service
public class HtmlInlineMail {

    @Autowired
    private JavaMailSender javaMailSender;

    public void sendMail() {
        try {
            MimeMessage message = javaMailSender.createMimeMessage();
            //是否发送的邮件是富文本（附件，图片，html等）
            MimeMessageHelper helper = new MimeMessageHelper(message,true);
            helper.setFrom("1010xxxxxx@qq.com");
            helper.setTo("1010xxxxxx@163.com");
            helper.setSubject("附件和Html邮件主题");
            //第二个参数是否启用html解析
            helper.setText("欢迎进入<a href=\"https://granett.github.io/\">我的主页</a><img src='cid:identifier1234'>",true);
            //内嵌一张图片
            FileSystemResource file = new FileSystemResource(new File("C:\\Users\\a\\Desktop\\avatar.jpg"));
            helper.addInline("identifier1234", file);
            javaMailSender.send(message);
        }catch (MessagingException e) {
            e.printStackTrace();
        }
    }
}
```
完整代码：[https://github.com/granett/spring-boot](https://github.com/granett/spring-boot)









