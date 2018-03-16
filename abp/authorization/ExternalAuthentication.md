# ExternalAuthentication实现

## 登录

Abp.Zero/Authorization/AbpLoginManager.cs中有两种类型的登录

1.  
``` CSharp
public virtual async Task<AbpLoginResult<TTenant, TUser>> LoginAsync(UserLoginInfo login, string tenancyName = null)
```

2. 
``` CSharp
public virtual async Task<AbpLoginResult<TTenant, TUser>> LoginAsync(string userNameOrEmailAddress, string plainPassword, string tenancyName = null, bool shouldLockout = true)
```

从UI进行登录使用的是第二种。

LoginAsync会调用LoginAsyncInternal， LoginAsyncInternal调用TryLoginFromExternalAuthenticationSources，在方法中遍历所有已经注册的ExternalAuthenticationSource，并调用ExternalAuthenticationSource的TryAuthenticateAsync方法进行登录。

## 创建/更新本地副本

登录成功会保存本地副本

![图片](https://raw.githubusercontent.com/latebrose/images/master/webside/abp/authorization/Snipaste_2018-03-16_12-28-39.jpg)

## 注册

https://github.com/aspnetboilerplate/aspnetboilerplate/blob/dev/doc/WebSite/Zero/User-Management.md#external-authentication