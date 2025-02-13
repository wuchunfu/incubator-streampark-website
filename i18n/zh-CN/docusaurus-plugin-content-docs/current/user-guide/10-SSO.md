---
id: 'SSO'
title: 'SSO集成'
sidebar_position: 10
---

## 背景介绍
作为企业实践，在应用程序中应用单点登录 (SSO) 是很常见的，这样可以通过集中且安全的方式管理用户凭证。

基于 Streampark 使用 Apache Shiro 进行身份验证和授权的事实，我们将使用 Pac4j 框架来实现单点登录 (SSO) 支持功能。 Pac4j 是 Shiro 社区推荐的 SSO 集成解决方案，也被其他 Apache 项目应用，如 Knox、Durid、Zeppelin 等。


## SSO 登录工作流
我们提出了三个主要用例，其工作流程如下所示：

a) SSO启用时，新用户登录
<img src="/doc/image/sso/new-user-login-process.png"/><br></br>

b) SSO启用时，现有用户登录
<img src="/doc/image/sso/existing-user-login-process.png"/><br></br>

c) SSO未启用时，用户登录
<img src="/doc/image/sso/user-login-sso-not-enabled.png"/><br></br>

## 如何启用SSO登录
- 从配置文件`application.yml`启用SSO:
```
...
spring:
  profiles:
    active: mysql #[h2,pgsql,mysql]
    include: sso
...
sso:
    # If turn to true, please provide the sso properties the application-sso.yml
    enable: true
```

- 选择第三方方式，例如Github或谷歌登录，填写如下所示的`application-sso.yml`配置:
```
pac4j:
  callbackUrl: http://localhost:10000/callback
  # Put all parameters under `properties`
  # Check supported sso config parameters for different authentication clients from the below link
  # https://github.com/pac4j/pac4j/blob/master/documentation/docs/config-module.md
  properties:
    # principalNameAttribute:
    # Optional, change by authentication client
    # Please replace and fill in your client config below when enabled SSO
    principalNameAttribute: email
    oidc:
      type: google
      id: xxx
      secret: xxx
      useNonce: true
    # github:
      # id: xxx
      # secret: xxx
```

- 重启Streampark，检查是否会重定向至正确的第三方登录页，并成功完成登录过程:

<img src="/doc/image/sso/github-login.png"/><br></br>
<img src="/doc/image/sso/google-login.png"/><br></br>
<img src="/doc/image/sso/login-success-redirect.png"/><br></br>

## 注意事项
目前我们仅支持`OAuth`和`OpenID Connect (OIDC)`作为常规支持的登录方式，如果您需要支持`Saml`或`CAS`，请转到`streampark-console/streampark-console-service/pom.xml`，将它们更为包含在依赖当中:
```
        <!-- Include pac4j-config/core/oauth/oidc-->
        <dependency>
            <groupId>org.pac4j</groupId>
            <artifactId>pac4j-springboot</artifactId>
            <version>${pac4jVersion}</version>
            <exclusions>
                <exclusion>
                    <groupId>commons-collections</groupId>
                    <artifactId>commons-collections</artifactId>
                </exclusion>
                <!-- cas & opensaml is not supported-->
                <exclusion>
                    <groupId>org.pac4j</groupId>
                    <artifactId>pac4j-cas</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.pac4j</groupId>
                    <artifactId>pac4j-saml-opensamlv3</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
```