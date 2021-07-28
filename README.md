=====miaosha
这是一个基于SpringBoot的秒杀项目，能实现基本的秒杀功能。</br>
前端采用的是Thymeleaf模板,缓存采用的Redis,消息队列采用的是RabbitMQ。</br>
项目亮点</br>
1、使用分布式Session,可以实现让多台服务器同时响应。</br>
2、使用redis做缓存提高访问速度和并发量，减少数据库的压力；</br>
3、使用页面静态化，加快用户访问速度，提高QPS。</br>
4、使用消息队列异步下单，提升用户体验，削峰和降流。</br>
5、安全性优化：双重MD5密码校验，秒杀接口地址隐藏，接口限流防刷，数学公式验证码。</br>
一、环境搭建</br>
Redis，RabbitMQ安装在linux系统上，参考安装：https://www.cnblogs.com/xyuanzi/p/15038995.html</br>
二、登录功能的实现</br>
![image](https://user-images.githubusercontent.com/58498940/127273827-3c15dde4-bbd0-4b59-9069-c5faa728aa81.png)
输入用户名15700761667，密码123456，即可登录。

2.1 数据库的设计</br>
    不做注册，只做登录。在MYSQL中创建表，并加入用户的信息，注意数据库的密码是加密以后的密码，在前台访问时，输入的时加密前的密码。</br>
    用户表中包括id,nickname,pasword,salt,注册时间，上次登录时间，登录次数等字段。</br>
2.2 明文密码两次MD5处理</br>
    加密的目的是：第一次是因为Http是明文传输的，不安全；第二次是防止数据库被盗，从而密码被破解。</br>
2.3 JSR303参数检验+全局异常处理</br>
    通过对输入的参数LOginVOC加注解@Validated,然后在传入的参数mobile和password上加上注解判断，如@NotNull判断是否为空，也可以自定义全局异常处理。</br>
2.4 分布式session</br>
    使用redis缓存客户端的信息，生成token,放在cookie中。</br>
    客户端每次访问时携带cookie,向服务端发起请求。服务端就能从cookie中找出用户的信息了。</br>
三、实现秒杀功能
 主要内容
  3.1数据库的设计</br>
 商品表goods，秒杀用户表miaosha_user，秒杀商品表miaosha_goods，秒杀商品订单表miaosha_order,订单信息表order_info</br>
 3.2商品列表页</br>
 ![image](https://user-images.githubusercontent.com/58498940/127273682-ddb57f45-9992-4377-a047-50c17df156bc.png)

 为了展示秒杀商品。点击详情进入商品具体详情页
 3.3商品详情页</br>
 ![image](https://user-images.githubusercontent.com/58498940/127273933-5a56db76-e415-4ea7-9765-2faa25c66257.png)
 ![image](https://user-images.githubusercontent.com/58498940/127274268-7da7c575-1a86-4efb-bb0c-f0a0af4c2125.png)
 3.4订单详情页</br>
 ![image](https://user-images.githubusercontent.com/58498940/127274606-cbbf6911-4176-4451-8662-5ce565cf8f58.png)

 
 此时已经完成了秒杀的所有功能。

   
