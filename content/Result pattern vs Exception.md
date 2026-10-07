# Result pattern vs Exception

**Проблема.** Частая ошибка — управлять потоком выполнения через исключения:

```csharp
// Плохо: исключение как способ вернуть бизнес-ошибку
public Account GetAccount(int userId)
{
    var account = _db.Accounts.Find(userId);
    if (account == null)
        throw new Exception("Account not found"); // это не исключительная ситуация
    return account;
}
```

Исключения дорогие: при выбросе среда выполнения собирает трассу стека и раскручивает стек до ближайшего `catch`. Они нужны для неожиданных сбоев, а не для ожидаемых бизнес-ошибок. К тому же по сигнатуре метода не видно, какие ошибки он может вернуть.

**Result pattern.** Метод всегда возвращает объект-результат, а не «взрывается». Вызывающий код сам решает, что делать с ошибкой.

Минимальная реализация:

```csharp
public class Result<T>
{
    public bool IsSuccess { get; }
    public T? Value { get; }
    public string? Error { get; }

    private Result(bool isSuccess, T? value, string? error)
    {
        IsSuccess = isSuccess;
        Value = value;
        Error = error;
    }

    public static Result<T> Success(T value) => new(true, value, null);
    public static Result<T> Failure(string error) => new(false, default, error);
}
```

Тот же метод с Result:

```csharp
public Result<Account> GetAccount(int userId)
{
    var account = _db.Accounts.Find(userId);
    return account is null
        ? Result<Account>.Failure("Account not found")
        : Result<Account>.Success(account);
}
```

**Как это выглядит в проекте.** Цепочка в учебном проекте: `TransferCommandHandler` → `Result<T>` → `FinanceController`.

- Хендлер возвращает `Result` вместо `throw`.
- Контроллер смотрит на `IsSuccess` и решает, какой HTTP-статус отдать.
- Ошибка явно проходит через слои, а не «всплывает» сквозь стек.

Исключения при этом никуда не деваются: недоступная БД, сетевая ошибка, `NullReferenceException` — это неожиданные сбои, их ловит глобальный обработчик ошибок.

**Когда что использовать**

| Ситуация | Инструмент |
|---|---|
| Пользователь не найден | Result |
| Недостаточно средств | Result |
| БД недоступна | Exception |
| `NullReferenceException`, выход за границы массива | Exception |
| Валидация входных данных | FluentValidation |

Правило: если ситуация — часть бизнес-логики и её можно ожидать, возвращай Result. Если что-то сломалось — бросай исключение.

**Что дальше.** Ошибки API стоит отдавать клиенту в стандартном формате Problem Details (RFC 9457, раньше RFC 7807).

**Задание.** Добавить в проект ещё один тип ошибки (например, `ErrorCode` enum вместо строки) и доработать маппинг ошибок в HTTP-статусы в контроллере.

Читать:

- [Рекомендации по исключениям](https://learn.microsoft.com/ru-ru/dotnet/standard/exceptions/best-practices-for-exceptions) — Microsoft: когда бросать исключения и как их обрабатывать.
- [Обработка ошибок в API ASP.NET Core](https://learn.microsoft.com/ru-ru/aspnet/core/web-api/handle-errors) — Microsoft: глобальная обработка исключений и Problem Details.
- [RFC 9457: Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457) — стандарт формата ошибок (англ.).
