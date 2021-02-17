---
title: Spring Security --- Authentication
date: 2021-02-08 15:36:54
description: "Spring Security的认证设计"
categories: 
- Spring Security
tags:
---

## Authentication设计

### 认证处理流程

![认证处理流程](abstractauthenticationprocessingfilter.png)

- SecurityContextHolder, 认证成功后, 可以在后续的处理中通过它来获取认证后的用户信息

![](SecurityContext.png)

### 认证的设计

```plantuml
interface Authentication {
    Collection<? extends GrantedAuthority> getAuthorities(); // 权限信息
    Object getCredentials(); // 用户校验信息
    Object getDetails();
    Object getPrincipal(); // 用户标识信息
}
abstract class AbstractAuthenticationProcessingFilter

AbstractAuthenticationProcessingFilter --> Authentication: get input from request

interface AuthenticationManager
class ProviderManager implements AuthenticationManager
interface AuthenticationProvider {
    Authentication authenticate(Authentication authentication) throws AuthenticationException;
    boolean supports(Class<?> authentication);
}
ProviderManager "1" *--> "*" AuthenticationProvider
class DaoAuthenticationProvider implements AuthenticationProvider
class JwtAuthenticationProvider implements AuthenticationProvider

Authentication --> AuthenticationManager: input for authentication
AuthenticationManager --> Authentication: get success response

interface SecurityContext
class SecurityContextHolder
SecurityContextHolder "1" *--> "1" SecurityContext
SecurityContext *--> Authentication: after autenticate sucess
```

![](providermanagers-parent.png)

## 常见的认证方式

```plantuml
title
|= **常见的认证方式** |= **Spring Security实现** |
| Basic/FormLogin | DaoAuthenticationProvider |
| OAuth2 | 待续 |
| LDAP | 待续 |
| 通过第三方认证 | Pre-Authentication Framework |
end title
```

### DaoAuthenticationProvider对Basic/FormLogin的支持

用于对username/password这种形式的认证信息进行校验,
认证信息通过**UsernamePasswordAuthenticationToken**进行包装.
其中password可以从本地服务中获取.

#### FormLogin流程

![](loginurlauthenticationentrypoint.png)

![](usernamepasswordauthenticationfilter.png)

- LoginUrlAuthenticationEntryPoint, 在认证失败时, 重导向到登录页面
- UsernamePasswordAuthenticationFilter处理登录页面的认证请求

#### Basic流程

![](basicauthenticationentrypoint.png)

![](basicauthenticationfilter.png)

- BasicAuthenticationEntryPoint, 在认证失败时返回WWW-Authenticate头信息
- BasicAuthenticationFilter, 处理Basic认证请求

#### DaoAuthenticationProvider的设计

![](daoauthenticationprovider.png)

---

```plantuml
interface Authentication
class UsernamePasswordAuthenticationToken implements Authentication

interface UserDetails {
    Collection<? extends GrantedAuthority> getAuthorities();
    String getPassword();
    String getUsername();
    boolean isAccountNonExpired();
    boolean isAccountNonLocked();
    boolean isCredentialsNonExpired();
    boolean isEnabled();
}
interface UserDetailsService {
    UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
UserDetailsService --> UserDetails: load by username
class InMemoryUserDetailsManager implements UserDetailsService
class JdbcUserDetailsManager implements UserDetailsService

interface PasswordEncoder

abstract class AbstractUserDetailsAuthenticationProvider implements AuthenticationProvider
AbstractUserDetailsAuthenticationProvider --> UsernamePasswordAuthenticationToken: supports

class DaoAuthenticationProvider extends AbstractUserDetailsAuthenticationProvider
DaoAuthenticationProvider "1" o--> "1" UserDetailsService: used to get UserDetails
DaoAuthenticationProvider "1" o--> "1" PasswordEncoder: encode input password

DaoAuthenticationProvider --> Authentication: create with UserDetails after success

```

通过UserDetailsService来查询username来获取对应的UserDetails用户信息, 包括用于校验的密码信息.

由于密码往往通过某种加密方式加密后保存, 
所以需要通过PasswordEncoder对输入的password进行加密后, 再同UserDetails中的密码信息进行比较.

### LDAP

待续

### OAuth2

待续

### Pre-Authentication Framework对第三方认证的集成

```plantuml
interface Authentication
class PreAuthenticatedAuthenticationToken implements Authentication

abstract class AbstractPreAuthenticatedProcessingFilter {
    Object getPreAuthenticatedPrincipal(HttpServletRequest request);
    Object getPreAuthenticatedCredentials(HttpServletRequest request);
}

AbstractPreAuthenticatedProcessingFilter --> PreAuthenticatedAuthenticationToken: create with principal and credentials

class PreAuthenticatedAuthenticationProvider implements AuthenticationProvider
PreAuthenticatedAuthenticationProvider --> PreAuthenticatedAuthenticationToken: supports

interface AuthenticationUserDetailsService<T extends Authentication> {
    UserDetails loadUserDetails(T token) throws UsernameNotFoundException;
}
class PreAuthenticatedGrantedAuthoritiesUserDetailsService<PreAuthenticatedAuthenticationToken> implements AuthenticationUserDetailsService
PreAuthenticatedAuthenticationProvider o--> AuthenticationUserDetailsService: get user
```

- AbstractPreAuthenticatedProcessingFilter, Pre-Authentication Framework的入口
- PreAuthenticatedAuthenticationToken, 包含第三方的用户校验信息或者认证成功后的用户信息
- PreAuthenticatedAuthenticationProvider, Pre-Authentication Framework默认的校验器
- AuthenticationUserDetailsService, 同第三方进行校验, 在校验成功后获取系统中的用户信息

### Remember Me

待续

## Session管理

Http Session相关的功能都在以下组件中进行处理:
- SessionManagementFilter
- SessionAuthenticationStrategy, 对认证成功的用户进行session校验
- SessionRegistry, 存储session和认证后的用户信息的键值对

主要处理以下问题:
- 防御会话固定攻击(session-fixation protection attack)
  - SessionFixationProtectionStrategy
- 检查Session是否超时
  - InvalidSessionStrategy, 对无效session进行处理
- 限制用户的同时登录数
  - ConcurrentSessionFilter
  - ConcurrentSessionControlAuthenticationStrategy

## 登出处理

登出时会对用户相关的登录信息进行清理:
- session无效化
- 清理SecurityContextHolder
- 重导向到登录页面
- 清理Cookies

通过LogoutHandler进行清理, 不可以抛出异常.

LogoutSuccessHandler在logout成功后, 
进行重定向(默认为/login?logout)或者返回指定的http status code(默认为200).

## Authentication Events

登录成功或者失败都会触发AuthenticationException事件, 根据事件做额外的处理.
通过AuthenticationEventPublisher来监听这些事件.

## Reference

- [Servlet Authentication](https://docs.spring.io/spring-security/site/docs/5.4.2/reference/html5/#servlet-authentication)
