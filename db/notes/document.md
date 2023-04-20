<p align="center">
  <b>Introduce to Document-oriented Database: MongoBD.</b>
</p>

# Conception

- Document -> Collection -> Database
- Example: MongoDB (using in Web App) 
  - using **B-Tree** as storage structure
- **Object Oriented Programming**
  - Unifying programming model and data model
  - Everything is treated as object

## Document

- like `json`, the `key-value` pair
- ![kv](/images/kv.png)
- Document can be seen as `object`

## Collection

- a class of document
- can be seen as `class`

## Database

- some documents constitute a database
- usually for one application

# Interface

## Insert

```sql
> db.foo.insert({"bar":"bsd})
> db.foo.batchInsert([{"_id":0},{"_id":1},{"_id":2}])
```

## Find

![find](/images/find.png)

## Delete

```sql
> db.foo.remove()
> db.mailing.list.remove({"opt-out":true})
```

## Update

![update](/images/update.png)



# Useful link

- [MongoDB CRUD Operations](https://docs.mongodb.com/manual/crud/)
- [json](https://www.w3schools.cn/js/js_json_intro.asp)
- [A Technical Introduction to WiredTiger](https://www.mongodb.com/presentations/a-technical-introduction-to-wiredtiger)
- [NoSQL Databases Explained](https://www.mongodb.com/nosql-explained)
- [Creating Great Programmers with a Software Design Studio - John Ousterhout (Stanford)](https://www.youtube.com/watch?v=ajFq31OV9Bk&t=180s)


