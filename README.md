# Strongly Typed Prefixed Ids
Sample implementation of strongly typed, prefixed id's that work with FastEndpoints, Swagger and Ef Core.

## Why?
I was getting tired of GUIDs. They're too long, too ugly for users in the url and impossible to remember for developers while debugging. \
I like the way Stripe does it. A small prefix in an id like "cc" makes it hard to plug the wrong id into something. \
"cc-apgd823-fjx9f23" is much better than just "apgd823-fjx9f23".

## Basic Usage
The code allows you to do this to define an entity for ef core. \
This automatically generates a id with 6 random characters like **"cc-osp4ny"**.
```cs
public record CreditCardId : TypedId
{
    protected override string prefix => "cc";
}

public class CreditCard : IHasTypedId<CreditCardId>
{
    public CreditCardId Id { get; set; } = new();
    public string CVC { get; set; } = "";
}
```

## Info
- Id's will be stored as strings in the database.
- In requests and responses id's just show up as strings and get (de)serialized into the correct type.
```cs
{
  "creditCard": {
    "id": "cc-osp4ny",
    "cvc": "123"
  }
}
```
- 4 Lines of configuration
- EF Core Update => AddOrUpdate behaviour stays consistent. \
 (Ef cores Update method is actually AddOrUpdate behind the scenes. If you call Update(newEntity) and save, ef core checks whether the primary key is `default` and fires an `INSERT` instead.)

### Duplicate Ids
This code does not handle duplicate id's. There are no retries, no checking before inserting.

I left this piece of code in so you can experiemnt and get a feel for the likelyhood of duplicates
```cs
for (int i = 0; i < 10; i++)
{
    var dbRowCount = 100000;
    var idLength = 6;
    var ids = Enumerable.Range(0, dbRowCount).Select(x => TypedId.GenerateRandomString(idLength)).ToList();
    Console.WriteLine($"ids: {ids.Count}, duplicates: {ids.Count - ids.ToHashSet().Count}");
}
```
This seems to average out at about **3 duplicates for 100.000 rows**. That's fine with me for the types of apps I'm working on.

When you try to insert a duplicate id postgres will throw an exception and any app getting to 100.000 rows, will already be handling exceptions. Then the user can simply try again.

If 100.000 was too low a guess, you can easily adjust the number of random characters after the fact too.
  
