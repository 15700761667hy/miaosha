********miaosha*********
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
    通过对输入的参数LoginVo加注解@Valid,然后在传入的参数mobile和password上加上注解判断，如@NotNull，@Length判断是否为空，手机号或密码长度是否符合，另外自定义全局异常处理，在不满足参数格式的情况下直接抛出异常。本项目中自定义了一个注解@IsMobile,利用正则表达式，规则是以1开头+10位整数的手机号格式。不符合格式的直接抛出异常。</br>
2.4 分布式session</br>
    使用redis缓存客户端的信息，生成token,放在cookie中。</br>
    客户端每次访问时携带cookie,向服务端发起请求。服务端就能从cookie中找出用户的信息了。</br>
    做法是：在登录密码比较完了以后，生成一个Token，调用addCookie(),将Token与用户的信息放入Redis缓存中。
    再设置cookie的有效期，生成cookie。将cookie返回给response,返回给客户端。
    每次登陆都会生成一个新的Token。
    但是客户端在每次访问页面的时候，就不再进行密码的比较，而是请求中携带着cookie信息，
    getByToken():取出cookie中的Token,然后去redis中查询用户信息。在这里访问的时候，会进行自动的有效期延长。</br>
三、实现秒杀功能</br>
 主要内容</br>
 3.1 数据库的设计</br>
    商品表goods，秒杀用户表miaosha_user，秒杀商品表miaosha_goods，秒杀商品订单表miaosha_order,订单信息表order_info</br>
 3.2 商品列表页</br>
 ![image](https://user-images.githubusercontent.com/58498940/127273682-ddb57f45-9992-4377-a047-50c17df156bc.png)

 为了展示秒杀商品。点击详情进入商品具体详情页</br>
 3.3 商品详情页</br>
 ![image](https://user-images.githubusercontent.com/58498940/127273933-5a56db76-e415-4ea7-9765-2faa25c66257.png)
 ![image](https://user-images.githubusercontent.com/58498940/127274268-7da7c575-1a86-4efb-bb0c-f0a0af4c2125.png)
 3.4 订单详情页</br>
 ![image](https://user-images.githubusercontent.com/58498940/127274606-cbbf6911-4176-4451-8662-5ce565cf8f58.png)

    此时已经完成了秒杀的所有功能。</br>
四、页面优化技术</br>
内容</br>
页面缓存+URL缓存+对象缓存</br>
页面静态化，前后端分离</br>
静态资源优化</br>
4.1 
    页面缓存的步骤：从redisservice中取缓存，若缓存中没有则手动渲染，利用thymeleaf模板，然后将页面加入缓存，并返回渲染页面。</br>
    URL缓存：与页面缓存的步骤基本一致，但是需要取缓存和加缓存时要加入参数GoodsId</br>
    对象缓存：指的是User对象。对象缓存是长期缓存，第一步是取缓存，若缓存中没有就去数据库中查找，并加入缓存。如数据库中没有就报错更新用户的密码。</br>
4.2 页面静态化，前后端分离</br>
页面静态化也就是使用纯HTML页面+AJAX请求json数据后再填充页面；</br>
若A页面跳转到B页面之前需要条件判断可以让A页面利用ajax请求判断后再跳转。如果不需要条件判断可以直接跳转到B的静态页面，让B自己用ajax请求数据。</br>
防超卖</br>
    一种情况是发生在减库存的时候。解决办法是在update语句中加一个判断。还有一种情况就是一个用户同时发生了两个请求，假如库存充足，且没有订单生成就会减两次库存。最好的办法就是建立用户与商品的唯一索引。</br>
五、接口优化</br>
内容</br>
5.1 Redis预减库存减少数据库的访问</br>
5.2 内存标记减少redis访问</br>
5.3 RabbitMQ队列缓冲，异步下单，增强用户体验</br>
六、安全优化</br>
主要内容</br>
秒杀接口地址隐藏</br>
数学公式验证码</br>
接口限流防刷</br>
6.1 秒杀接口地址隐藏</br>
    虽然前端页面在秒杀前秒杀按钮设置为不可用，但是有可能用户通过前端js代码找到秒杀地址在秒杀未开始之前就直接访问。秒杀接口地址隐藏就是让用户通过js代码获取到的秒杀地址并不能完成其秒杀功能。在秒杀前要先通过Contoroller中的path路径下的类随机生成一个path,然后和用户的ID一起存入Redis,在执行秒杀的时候再从redis中取path进行验证，然后进行秒杀。</br> 
6.2 数学公式验证码</br>
    接口防刷，错开请求</br>
6.3 接口限流防刷</br>
    当一个用户访问接口时，把访问次数写入缓存，并设置有效期。一分钟之内如果用户访问，则缓存中的次数加1，如果次数超出限制，那么进行限流操作。如果一分钟内没有超限，有效期到了，缓存中数据消失。下次再访问时会重新写入缓存。</br>
使用一个通用拦截器，首先写一个注解Accesslimit，后面每个类只需要加注解就可以设置防刷次数。
定义拦截器：继承HandlerInterceptorAdpter类。











   
