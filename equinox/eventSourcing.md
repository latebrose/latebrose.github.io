
1. CustomerAppService使用IMediatorHandler发送UpdateCustomerCommand
``` CSharp
public void Update(CustomerViewModel customerViewModel)
{
    var updateCommand = _mapper.Map<UpdateCustomerCommand>(customerViewModel);
    Bus.SendCommand(updateCommand);
}
```
2. IMediatorHandler由InMemoryBus实现,使用IMediator发送Command
``` CSharp
public Task SendCommand<T>(T command) where T : Command
{
    return _mediator.Send(command);
}
```

3. IMediator在Startup中注册
``` CSharp
// Adding MediatR for Domain Events and Notifications
services.AddMediatR(typeof(Startup));
```

4. 所以CustomerCommandHandler实现了IRequestHandler接口，所以监听到（消息由MediatR发出，由MediatR负责调用实现了IRequestHandler的类处理UpdateCustomerCommand，并进行处理

``` CSharp
public class CustomerCommandHandler : CommandHandler,
    IRequestHandler<RegisterNewCustomerCommand>,
    IRequestHandler<UpdateCustomerCommand>,
    IRequestHandler<RemoveCustomerCommand>
```

5. UpdateCustomerCommand处理成功后，又触发了CustomerUpdatedEvent
``` CSharp
if (Commit())
{
    Bus.RaiseEvent(new CustomerUpdatedEvent(customer.Id, customer.Name, customer.Email, customer.BirthDate));
}
```

6. 在InMemoryBus,保存CustomerUpdatedEvent
``` CSharp
public Task RaiseEvent<T>(T @event) where T : Event
{
    if (!@event.MessageType.Equals("DomainNotification"))
        _eventStore?.Save(@event);

    return _mediator.Publish(@event);
}
```