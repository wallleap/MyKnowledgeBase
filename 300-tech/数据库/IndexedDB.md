---
title: IndexedDB
date: 2024-03-25T09:31:00+08:00
updated: 2024-08-21T10:32:34+08:00
dg-publish: false
---

## IndexedDB

Cookie 的大小通常为 4KB；LocalStorage 的大小通常在 2.5MB 到 10MB 之间，且它不提供搜索功能，也不能建立自定义的索引。因此，它们都不适合存储大量的数据。为了填补浏览器在这方面的不足，IndexedDB 应运而生。

IndexedDB 是浏览器提供的本地数据库，它是一个事务型数据库系统，可以通过网页脚本创建和操作。相较于 LocalStorage，IndexedDB 更适合存储大量的数据。它具备查找接口和建立索引的功能，而这些是 LocalStorage 所不具备的。与传统的关系型数据库不同，IndexedDB 不支持 SQL 查询语句，更类似于 NoSQL 数据库。

## [](https://www.mengke.me/blog/202308/IndexedDB_Quick_Start#%E6%89%93%E5%BC%80%E6%95%B0%E6%8D%AE%E5%BA%93) 打开数据库

IndexedDB 可以直接打开，如果指定的数据库不存在时会自动新建数据库。

`open` 方法可以传入两个参数：第一个参数是数据库名称，第二个参数是数据库版本。数据库的版本不传时默认时当前版本，如果数据库不存在自动新建数据库时版本默认为 `1`。

```js
const request = indexedDB.open(databaseName, version)
```

`open` 方法返回的是一个 `IDBRequest` 实例。

如果 `request` 对象成功执行了，结果可以通过 `result` 属性访问到，并且该 `request` 对象上会触发 `success` 事件。

如果操作中有错误发生，一个 `error` 事件会触发，并且会通过 `result` 属性抛出一个异常。

如果指定的版本大于当前版本，则会出发 `upgradeneeded` 事件，当数据库不存在自动创建数据库时也会触发这个事件。

```js
const request = window.indexedDB.open('testDB')
request.onsuccess = (event) => {
  const db = event.target.result
  console.log('Database opened successfully!')
}
request.onerror = (event) => {
  console.log('Database opened failed!')
}
request.onupgradeneeded = (event) => {
  console.log('Database upgrade needed!')
}
```

## [](https://www.mengke.me/blog/202308/IndexedDB_Quick_Start#%E6%96%B0%E5%BB%BA%E8%A1%A8) 新建“表”

打开或新建数据库后，通过 `result` 拿到数据库对象，后续的操作我们都在此对象上进行。

通过 `createObjectStore` 方法可以创建一个 store，也就相当于是表。

```js
// version 默认为 1
const request = indexedDB.open('testDB')

request.onupgradeneeded = (event) => {
  const db = event.target.result
  let objectStore
  if (!db.objectStoreNames.contains('user')) {
    objectStore = db.createObjectStore('user', { keyPath: 'id' })
  }
}
```

通过以上代码，打开数据库，判断 `user` 数据库是否存在，如果不存在泽新建一个名为 `user` 的表，并指定主键为 `id`。

主键（key）是默认建立索引的属性。比如，数据记录是 `{ id: 1, name: '张三' }`，那么 `id` 属性可以作为主键。主键也可以指定为下一层对象的属性，比如 `{ foo: { bar: 'baz' } }` 的 `foo.bar` 也可以指定为主键。

如果数据记录里面没有合适作为主键的属性，那么可以让 IndexedDB 自动生成主键。下面代码中，指定主键为一个递增的整数。

```js
objectStore = db.createObjectStore('user', { autoIncrement: true })
```

## [](https://www.mengke.me/blog/202308/IndexedDB_Quick_Start#%E5%88%9B%E5%BB%BA%E7%B4%A2%E5%BC%95) 创建索引

可以通过 `createIndex` 方法创建表索引。

```js
request.onupgradeneeded = function (event) {
  db = event.target.result
  var objectStore = db.createObjectStore('user', { keyPath: 'id' })
  objectStore.createIndex('name', 'name', { unique: false })
  objectStore.createIndex('age', 'age', { unique: true })
}
```

## [](https://www.mengke.me/blog/202308/IndexedDB_Quick_Start#%E6%93%8D%E4%BD%9C%E6%95%B0%E6%8D%AE) 操作数据

### [](https://www.mengke.me/blog/202308/IndexedDB_Quick_Start#%E5%A2%9E) 增

使用 IndexedDB，可以轻松地向数据库中添加新的数据。可以通过创建对象存储空间（Object Store）来存储数据，并使用事务来确保数据的完整性和一致性。

```js
const transaction = db.transaction('user', 'readwrite')
const objectStore = transaction.objectStore('user')

const data = { id: 1, name: 'Mengke', age: 26 }
const request = objectStore.add(data)

request.onsuccess = () => {
  console.log('数据添加成功')
}
```

上面代码中，写入数据需要新建一个事务。新建时必须指定表格名称和操作模式（" 只读 " 或 " 读写 "）。新建事务以后，通过 `IDBTransaction.objectStore(name)` 方法，拿到 IDBObjectStore 对象，再通过表格对象的 `add()` 方法，向表格写入一条记录。

写入操作是一个异步操作，通过监听连接对象的 `success` 事件和 `error` 事件，了解是否写入成功。

### [](https://www.mengke.me/blog/202308/IndexedDB_Quick_Start#%E5%88%A0) 删

如果需要从数据库中删除某些数据，可以使用事务来删除特定的对象或整个对象存储空间。

```js
const transaction = db.transaction('user', 'readwrite')
const objectStore = transaction.objectStore('user')

const request = objectStore.delete(1)

request.onsuccess = () => {
  console.log('数据删除成功')
}
```

### [](https://www.mengke.me/blog/202308/IndexedDB_Quick_Start#%E6%94%B9) 改

当需要修改已有的数据时，可以使用事务来更新数据。通过指定要更新的对象存储空间和键值，可以进行数据的修改。

```js
const transaction = db.transaction('user', 'readwrite')
const objectStore = transaction.objectStore('user')

const request = objectStore.get(1)

request.onsuccess = (event) => {
  const data = event.target.result
  data.age = 27

  const updateRequest = objectStore.put(data)
  updateRequest.onsuccess = () => {
    console.log('数据更新成功')
  }
}
```

### [](https://www.mengke.me/blog/202308/IndexedDB_Quick_Start#%E6%9F%A5) 查

一旦数据存储在数据库中，可以使用查询操作来检索所需的数据。可以根据特定的关键字或范围进行查询，并获取相应的数据结果。

```js
const transaction = db.transaction('user', 'readonly')
const objectStore = transaction.objectStore('user')

const request = objectStore.get(1)

request.onsuccess = (event) => {
  const data = event.target.result
  console.log(data)
}
```

### [](https://www.mengke.me/blog/202308/IndexedDB_Quick_Start#%E9%81%8D%E5%8E%86%E6%95%B0%E6%8D%AE) 遍历数据

IndexedDB 还提供了遍历数据的功能。可以使用游标（Cursor）来逐个迭代数据库中的数据，并进行相应的操作。

```js
const transaction = db.transaction('user', 'readonly')
const objectStore = transaction.objectStore('user')

const request = objectStore.openCursor()

request.onsuccess = (event) => {
  const cursor = event.target.result
  if (cursor) {
    console.log(cursor.value)
    cursor.continue()
  }
}
```

## [](https://www.mengke.me/blog/202308/IndexedDB_Quick_Start#%E7%B4%A2%E5%BC%95) 索引

在 IndexedDB 中，可以创建索引来提升数据的查询性能。索引可以根据特定的字段对数据进行排序和过滤，加快数据检索的速度。

```js
const objectStore = db.createObjectStore('user', { keyPath: 'id' })
objectStore.createIndex('nameIndex', 'name', { unique: false })
```
