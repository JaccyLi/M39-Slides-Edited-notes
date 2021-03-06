# 一.Tomcat 安全配置

1. 删除 webapps 目录下的所有文件,禁用 tomcat 管理界面;
2. 注释或删除 tomcat-users.xml 文件内的所有用户权限;
3. 更改关闭 tomcat 指令或禁用;
   tomcat 的 `server.xml` 中定义了可以直接关闭 Tomcat 实例的管理端口
   (默认 8005)。可以通过 telnet 连接上该端口之后,输入 `SHUTDONN` (此
   为默认关闭指令)即可关闭 Tomcat 实例(注意,此时虽然实例关闭了,但是进程
   还是存在的)。由于默认关闭 romcat 的端口和指令都很简单。默认端口为
   8005 ,指令为 SHUTDOWN。

   安全方案一:

   ```xml
   <!-- 更改端口号和指令 : -->
   <server port="8456" shutdown="suosuoli_shutdown">
   ```

   安全方案二:

   ```xml
   <!--禁用 8005 端口 : -->
   <server port="-1" ahutdown="SHUTDOWN">
   ```

4. 定义错误页面
   在 `webapps/ROOT` 目录下定义错误页面 404.html , 500.html ; 然后在
   `tomcat/conf/web.xml` 中进行配置错误页面:
   ```xml
   <error-page>
       <error-code>404</error-code>
       <locat1on>/404.htm1</location>
   </error-page>
   <error-page>
        <error-code>500<Jerror-code>
        <location>/500.html</1cation>
   </error-page>
   ```
   这样配置之后,用户在访问资源时出现 404, 500 这样的异常,会看到自定义的
   错误页面,而不会看到异常的堆栈信息，提升了用户体验，也避免暴露技术栈，
   保障了服务的安全性。

# 二.Tomcat 应用安全

在大部分的 web 应用中,特别是一些后台应用系统 ,都会实现自己的安全管理模块
(权限模块)，用于控制应用系统的安全访问,基本包含两个部分:**认证**(登录/单
点登录)和**授权**(功能权限、数据权限)两个部分。对于当前的业务系统,可以自
己做一套适用于自己业务系统的权限模块，也有很多的应用系统直接使用一些功能
完善的安全框架,将其集成到我们的 web 应用中,如: springsecurity、
Apache shiro 等。

# 三.Tomcat 传输安全

## 3.1 HTTPS 介绍

HTTPS 的全称是`超文本传输安全协议`(Hypertext Transfer protocol secure) ,
是一种网络安全传输协议。在 HTT 的基础上加入 SSL/TLS 来进行数据加密,保护交换
数据不被泄露、窃取。

SSL 和 TLS 是用于网络通信安全的加密协议,它允许客户端和服务器之间通过安全
链接通信。SSL 协议的 3 个特性:

1. 保密:通过 SSL 链接传输的数据时加密的。
2. 鉴别:通信双方的身份鉴别,通常是可选的,单至少有一 方需要验证。
3. 完整性 :传输数据的完整性检查。

从性能角度考虑,加解密是一项计算昂贵的处理,尽量不要将整个 web 应用采用
SSL 链接，实际部署过程中 ，选择有必要进行安全加密的页面(存在敏感信息传输的
页面)采用 SSL 通信。对于 Tomcat 服务器来说，其计算资源非常珍贵，可以将加密
通讯部署到代理服务器，内部的 Tomcat 服务器尽量少用或者不同 SSL 加密连接。

**HTTPS 和 HTTP 的区别主要为以对四点**:

1. HTTPS 协议需要到证书颁发机构 cA 申请 ssL 证书， 然后与域名进行绑定, HTTP
   不用申请证书;
2. HTTP 是超文本传输协议,属于应用层信息传输, HTTPS 则是具有 SSL 加密传输的
   安全性传输协议,对数据的传输进行加密,相当于 HTTP 的升级版;
3. HTTP 和 HTTPS 使用的是完全不同的连接方式 ,用的端口也不一样,前者是 80 ,
   后者是 443.
4. HTTP 的连接很简单 ,是无状态的; HITPS 协议是由 SSL+HTTP 协议构建的可进行
   加密传输、身份认证的网络协议, 比 HTTP 协议安全。

**HTTPS 协议优势**:

1. 提高网站排名 ,有利于 SEO。谷歌已经公开声明两个网站在搜索结果方面相同,如果
   一个网站启用了 SSL ，它可能会获得略高于没有 SSL 网站的等级,而且百度也表明
   对安装了 SSL 的网站表示友好。因此,网站上的内容中启用 SSL 都有明显的 SEO 优势。
2. 隐私信息加密,防止流量劫持。特别是涉及到隐私信息的网站,互联网大型的数据泄露
   的事件频发发生,网站进行信息加密势在必行。
3. 浏览器受信任。 自从各大主流浏览器大力支持 HTTPS 协议之后 ,访问 HTTP 的网站
   都会提示"不安全"的警告信息。

## 3.2 Tomcat 配置 HTTPS

**Tomcat 不建议使用 HTTPS**
