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

``` CSharp
using (var source = IocResolver.ResolveAsDisposable<IExternalAuthenticationSource<TTenant, TUser>>(sourceType))
{
    if (await source.Object.TryAuthenticateAsync(userNameOrEmailAddress, plainPassword, tenant))
    {
        var tenantId = tenant == null ? (int?)null : tenant.Id;
        using (UnitOfWorkManager.Current.SetTenantId(tenantId))
        {
            var user = await UserManager.AbpStore.FindByNameOrEmailAsync(tenantId, userNameOrEmailAddress);
            if (user == null)
            {
                user = await source.Object.CreateUserAsync(userNameOrEmailAddress, tenant);

                user.TenantId = tenantId;
                user.AuthenticationSource = source.Object.Name;
                user.Password = UserManager.PasswordHasher.HashPassword(Guid.NewGuid().ToString("N").Left(16)); //Setting a random password since it will not be used

                if (user.Roles == null)
                {
                    user.Roles = new List<UserRole>();
                    foreach (var defaultRole in RoleManager.Roles.Where(r => r.TenantId == tenantId && r.IsDefault).ToList())
                    {
                        user.Roles.Add(new UserRole(tenantId, user.Id, defaultRole.Id));
                    }
                }

                await UserManager.AbpStore.CreateAsync(user);
            }
            else
            {
                await source.Object.UpdateUserAsync(user, tenant);

                user.AuthenticationSource = source.Object.Name;

                await UserManager.AbpStore.UpdateAsync(user);
            }

            await UnitOfWorkManager.Current.SaveChangesAsync();

            return true;
        }
    }
}
```

## 注册

https://github.com/aspnetboilerplate/aspnetboilerplate/blob/dev/doc/WebSite/Zero/User-Management.md#external-authentication