Обычно стандартный вопрос: **какие есть виды регистрации зависимостей**?

- _Transient_ – Новый экземпляр будет создан при каждом обращении к контейнеру.
- _Scoped_ – Новый экземпляр будет создан для каждого запроса к серверу. При этом в рамках самого запроса всегда будет передаваться один и тоже экземпляр.
- _Singleton_ – Всегда будет использоваться один и тоже экземпляр данного типа.

Второй стандартный вопрос: **как вызвать scoped зависимость внутри singleton**? Например вызвать базу в фоновой джобе. Если мы попытаемся внедрить ее напрямую, будет выкинуто исключение.

Решение - открыть новый scope, и внутри него вызывать зависимости. При закрытии скоупа, зависимости уничтожатся.

```C#
static void ScopeExample(IServiceProvider serviceProvider)
{
    // время жизни скоупа - до конца using
    using IServiceScope serviceScope = serviceProvider.CreateScope();
    IServiceProvider provider = serviceScope.ServiceProvider;

    var scopedService = provider.GetRequiredService<IMyScopedService>();
    var transientService = provider.GetRequiredService<IMyTransientService>();
}
```

Третий нестандартный вопрос: **диспозит ли di сервисы самостоятельно?**

Ответ: да, если он сам их создает. Если мы пишем `AddScoped(new ExampleService())`, то не диспозит.

Transient Disposable сервисы не диспозятся до окончания работы приложения или до окончания скоупа, что может привести к утечке.

Отдельная тема Razor/Blazor - там нужно скорее всего надо следить за временем жизни сервисов самостоятельно: [https://github.com/dotnet/aspnetcore/issues/5496](https://github.com/dotnet/aspnetcore/issues/5496)

На вопрос про другие контейнеры: есть autofac, он немного гибче настраивается, в целом то же самое.

Читать: [https://andrey.moveax.ru/post/asp-net-core-dependency-injection](https://andrey.moveax.ru/post/asp-net-core-dependency-injection)