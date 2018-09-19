# 新建UnitOfWork事务

* 默认情况下，当使用UnitOfWork的方法嵌套调用时，Abp自动维护调用的数据库上下文（DbContext），使所有方法使用同一个数据库上下文。

``` CSharp
public class A{
    public virtual ICollection<B> Bs {get;set;}
}

public class B{
    public virtual int AId {get;set;}
    public virtual A A {get;set;}
}

```
* 不使用级联删
* 对类A进行软删除
* 对类B进行软删除
* 在同一个UnitOfWork中先删除A的所有B（但是B没有被删除，实际上是更新操作），再删除A。在保存（SaveChanges）时，虽然B已经被删除了，但是因为是软删的缘故，B存在于数据库上下文中（会DetectChanges），此时B关联A，所以无法删除A。

``` CSharp
public static void UsingNewScope(this IUnitOfWorkManager unitOfWorkManager, Action action)
{
    using (var uow = unitOfWorkManager.Begin(new UnitOfWorkOptions
    {
        Scope = TransactionScopeOption.RequiresNew
    }))
    {
        action();
        uow.Complete();
    }
}
```

* 当使用了新事务时，因为查询B和查询A不在同一个数据库上下文，因此A上下文中没有对B数据的Detech，所以删除A时，不对B的合法性进行检查。

## 其他方案

重载A的repository的delete方法，在删除A时，先遍历A的Bs，将所有B置为Detached，再删除A。
