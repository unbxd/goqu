# Updating

* [Create a UpdateDataset](#create)
* Examples
  * [Set with `goqu.Record`](#set-record)
  * [Set with struct](#set-struct)
  * [Set with map](#set-map)
  * [Where](#where)
  * [Order](#order)
  * [Limit](#limit)
  * [Returning](#returning)
  * [Executing](#executing)
  
<a name="create"></a>  
To create a [`UpdateDataset`](https://godoc.org/github.com/doug-martin/goqu/#UpdateDataset)  you can use

**[`goqu.Update`](https://godoc.org/github.com/doug-martin/goqu/#Update)**

When you just want to create some quick SQL, this mostly follows the `Postgres` with the exception of placeholders for prepared statements.

```go
ds := goqu.Update("user").Set(
    goqu.Record{"first_name": "Greg", "last_name": "Farley"},
)
updateSQL, _, _ := ds.ToSQL()
fmt.Println(insertSQL, args)
```
Output:
```
UPDATE "user" SET "first_name"='Greg', "last_name"='Farley'
```

**[`SelectDataset.Update`](https://godoc.org/github.com/doug-martin/goqu/#SelectDataset.Update)**

If you already have a `SelectDataset` you can invoke `Update()` to get a `UpdateDataset`

**NOTE** This method will also copy over the `WITH`, `WHERE`, `ORDER`, and `LIMIT` clauses from the update

```go
ds := goqu.From("user")

updateSQL, _, _ := ds.Update().Set(
    goqu.Record{"first_name": "Greg", "last_name": "Farley"},
).ToSQL()
fmt.Println(insertSQL, args)

updateSQL, _, _ = ds.Where(goqu.C("first_name").Eq("Gregory")).Update().Set(
    goqu.Record{"first_name": "Greg", "last_name": "Farley"},
).ToSQL()
fmt.Println(insertSQL, args)
```
Output:
```
UPDATE "user" SET "first_name"='Greg', "last_name"='Farley'
UPDATE "user" SET "first_name"='Greg', "last_name"='Farley' WHERE "first_name"='Gregory'
```

**[`DialectWrapper.Update`](https://godoc.org/github.com/doug-martin/goqu/#DialectWrapper.Update)**

Use this when you want to create SQL for a specific `dialect`

```go
// import _ "github.com/doug-martin/goqu/v8/dialect/mysql"

dialect := goqu.Dialect("mysql")

ds := dialect.Update("user").Set(
    goqu.Record{"first_name": "Greg", "last_name": "Farley"},
)
updateSQL, _, _ := ds.ToSQL()
fmt.Println(insertSQL, args)
```
Output:
```
UPDATE `user` SET `first_name`='Greg', `last_name`='Farley'
```

**[`Database.Update`](https://godoc.org/github.com/doug-martin/goqu/#DialectWrapper.Update)**

Use this when you want to execute the SQL or create SQL for the drivers dialect.

```go
// import _ "github.com/doug-martin/goqu/v8/dialect/mysql"

mysqlDB := //initialize your db
db := goqu.New("mysql", mysqlDB)

ds := db.Update("user").Set(
    goqu.Record{"first_name": "Greg", "last_name": "Farley"},
)
updateSQL, _, _ := ds.ToSQL()
fmt.Println(insertSQL, args)
```
Output:
```
UPDATE `user` SET `first_name`='Greg', `last_name`='Farley'
```

### Examples

For more examples visit the **[Docs](https://godoc.org/github.com/doug-martin/goqu/#UpdateDataset)**

<a name="set-record"></a>
**[Set with `goqu.Record`](https://godoc.org/github.com/doug-martin/goqu/#UpdateDataset.Set)**

```go
sql, args, _ := goqu.Update("items").Set(
	goqu.Record{"name": "Test", "address": "111 Test Addr"},
).ToSQL()
fmt.Println(sql, args)
```

Output:
```
UPDATE "items" SET "address"='111 Test Addr',"name"='Test' []
```

<a name="set-struct"></a>
**[Set with Struct](https://godoc.org/github.com/doug-martin/goqu/#UpdateDataset.Set)**

```go
type item struct {
	Address string `db:"address"`
	Name    string `db:"name"`
}
sql, args, _ := goqu.Update("items").Set(
	item{Name: "Test", Address: "111 Test Addr"},
).ToSQL()
fmt.Println(sql, args)
```

Output:
```
UPDATE "items" SET "address"='111 Test Addr',"name"='Test' []
```

With structs you can also skip fields by using the `skipupdate` tag

```go
type item struct {
	Address string `db:"address"`
	Name    string `db:"name" goqu:"skipupdate"`
}
sql, args, _ := goqu.Update("items").Set(
	item{Name: "Test", Address: "111 Test Addr"},
).ToSQL()
fmt.Println(sql, args)
```

Output:
```
UPDATE "items" SET "address"='111 Test Addr' []
```

<a name="set-map"></a>
**[Set with Map](https://godoc.org/github.com/doug-martin/goqu/#UpdateDataset.Set)**

```go
sql, args, _ := goqu.Update("items").Set(
	map[string]interface{}{"name": "Test", "address": "111 Test Addr"},
).ToSQL()
fmt.Println(sql, args)
```

Output:
```
UPDATE "items" SET "address"='111 Test Addr',"name"='Test' []
```

<a name="where"></a>
**[Where](https://godoc.org/github.com/doug-martin/goqu/#UpdateDataset.Where)**

```go
sql, _, _ := goqu.Update("test").
	Set(goqu.Record{"foo": "bar"}).
	Where(goqu.Ex{
		"a": goqu.Op{"gt": 10},
		"b": goqu.Op{"lt": 10},
		"c": nil,
		"d": []string{"a", "b", "c"},
	}).ToSQL()
fmt.Println(sql)
```

Output:
```
UPDATE "test" SET "foo"='bar' WHERE (("a" > 10) AND ("b" < 10) AND ("c" IS NULL) AND ("d" IN ('a', 'b', 'c')))
```

<a name="order"></a>
**[Order](https://godoc.org/github.com/doug-martin/goqu/#UpdateDataset.Order)**

**NOTE** This will only work if your dialect supports it

```go
// import _ "github.com/doug-martin/goqu/v8/dialect/mysql"

ds := goqu.Dialect("mysql").
	Update("test").
	Set(goqu.Record{"foo": "bar"}).
	Order(goqu.C("a").Asc())
sql, _, _ := ds.ToSQL()
fmt.Println(sql)
```

Output:
```
UPDATE `test` SET `foo`='bar' ORDER BY `a` ASC
```

<a name="limit"></a>
**[Order](https://godoc.org/github.com/doug-martin/goqu/#UpdateDataset.Limit)**

**NOTE** This will only work if your dialect supports it

```go
// import _ "github.com/doug-martin/goqu/v8/dialect/mysql"

ds := goqu.Dialect("mysql").
	Update("test").
	Set(goqu.Record{"foo": "bar"}).
	Limit(10)
sql, _, _ := ds.ToSQL()
fmt.Println(sql)
```

Output:
```
UPDATE `test` SET `foo`='bar' LIMIT 10
```

<a name="returning"></a>
**[Returning](https://godoc.org/github.com/doug-martin/goqu/#UpdateDataset.Returning)**

Returning a single column example.

```go
sql, _, _ := goqu.Update("test").
	Set(goqu.Record{"foo": "bar"}).
	Returning("id").
	ToSQL()
fmt.Println(sql)
```

Output:
```
UPDATE "test" SET "foo"='bar' RETURNING "id"
```

Returning multiple columns

```go
sql, _, _ := goqu.Update("test").
	Set(goqu.Record{"foo": "bar"}).
	Returning("a", "b").
	ToSQL()
fmt.Println(sql)
```

Output:
```
UPDATE "test" SET "foo"='bar' RETURNING "a", "b"
```

Returning all columns

```go
sql, _, _ := goqu.Update("test").
	Set(goqu.Record{"foo": "bar"}).
	Returning(goqu.T("test").All()).
	ToSQL()
fmt.Println(sql)
```

Output:
```
UPDATE "test" SET "foo"='bar' RETURNING "test".*
```

<a name="executing"></a>
## Executing Updates

To execute Updates use [`goqu.Database#Update`](https://godoc.org/github.com/doug-martin/goqu/#Database.Update) to create your dataset

### Examples

**Executing an update**
```go
db := getDb()

update := db.Update("goqu_user").
	Where(goqu.C("first_name").Eq("Bob")).
	Set(goqu.Record{"first_name": "Bobby"}).
	Executor()

if r, err := update.Exec(); err != nil {
	fmt.Println(err.Error())
} else {
	c, _ := r.RowsAffected()
	fmt.Printf("Updated %d users", c)
}
```

Output:

```
Updated 1 users
```

**Executing with Returning**

```go
db := getDb()

update := db.Update("goqu_user").
	Set(goqu.Record{"last_name": "ucon"}).
	Where(goqu.Ex{"last_name": "Yukon"}).
	Returning("id").
	Executor()

var ids []int64
if err := update.ScanVals(&ids); err != nil {
	fmt.Println(err.Error())
} else {
	fmt.Printf("Updated users with ids %+v", ids)
}

```

Output:
```
Updated users with ids [1 2 3]
```