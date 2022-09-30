<!-- .slide: data-background="#003d73" -->
## Mongo transaction and query

![AU Logo](./../img/aulogo_uk_var2_white.png "AU Logo") <!-- .element style="width: 200px; position: fixed; bottom: 50px; left: 50px" -->

----

## Agenda

* Transactions
    * Two phase-commit
    * Multiple documents
* Querying

---

### Why Transactions

* Deletion in 1-N relationship
* Adjusting quantity of inline object
* Keeping consistency

----

### Two-phace commit

In a distributed system (multiple DB Servers)

1. Each server prepares to execute a transaction (locking documents or collections)
1. All servers apply the changes
1. All servers acknowlegde the changes
1. Lock is removed

This is potential a **really slow** operation. So we need something different

----

### 1-N relationship

3 solutions:

1. `order` then `order_items`
1. `order_items` then `order`
1. if `order_items` are embedded - just delete `order`

----

### Problems with Embedding

* Changing `qty` in memeory - race conditions
```javascript
{ _id: '11223',
 total: 500.94, ...
 items: [
 { sku: '123', price: 55.11, qty: 2 },
 { sku: '...', ... },...
 ] }
```
* Using db.collection.update - here we still need to check if document has changed
```javascript
db.orders.update(
 { '_id': order_id, 'items.sku': sku },
 { '$inc': {
 'total': total_update,
 'items.$.qty': qty }
 }
)
```

----

### Consistency between documents

1. Update each document with `db.collection.update()`
    * What if something trows and exception?
1. Emulate transactions in data model
    * Create a transaction collection
        * Document can be in *new*, *committed* or *rollback* state.
        * Clean up accordingly

----

### Modeling transaction

```javascript
// Transaction document
{ _id: ObjectId(...),
 state: 'new',
 ts: ISODateTime(...),
 amt: 55.22,
 src: 1,
 dst: 2
}
```

* Algorithm
    1. Insert transaction
    1. Withdraw money
    1. Insert money
    1. Update transaction
* Cleanup (delete and rollback periodically)

---

## Eventually consistency

* Avoid using transactions altogther
* Accept that all data isn't always up to date
    * As long as the will be at some point


----

### CQRS as an example

![CQRS](./img/sync_write_read.jpeg " source: https://awesomeopensource.com/project/fals/cqrs-clean-eventual-consistency?categoryPage=2")


---

## Querying

![Querying](./img/querying.png "")

----

### Projection

```csharp
public IEnumerable<string> GetFormats(string id)
{
    var projection = Builders<Book>.Projection
            .Include(b => b.Formats);
    var bson = _books.Find<Book>(book => book.Id == id)
        .Project(projection).FirstOrDefault();
    var array = bson.GetElement("Formats").Value.AsBsonArray;
    return array.Select(str => str.AsString);
}
```

----

### Aggregation

```csharp
var results = db.GetCollection<ZipEntry>.Aggregate()
    .Group(x => x.State,
           g => new { State = g.Key,
                      TotalPopulation =
                            g.Sum(x => x.Population)
                    }
           )
    .Match(x => x.TotalPopulation > 20000)
    .ToList();
```

----

### Update schema


```csharp
[BsonIgnoreExtraElements]
class Model : ISupportInitialize {
    ...
    [BsonExtraElements]
    public IDictionary<string, object> ExtraElements { get; set; }
    public void BeginInit() { }
    
    public void EndInit()
    {
      object oldValue;
      if (!ExtraElements.TryGetValue("OldField", out oldValue))
      {
        return;
      }
    
      var value = (string) olderValue;
      ExtraElements.Remove("OldField");
      // Set new values
    }
}

