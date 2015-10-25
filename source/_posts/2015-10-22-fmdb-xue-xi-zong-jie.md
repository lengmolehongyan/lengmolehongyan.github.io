---
layout: post
title: "FMDB 使用方法"
date: 2015-10-22 22:41:56 +0800
comments: true
categories: 
keywords: fmdb，数据库，SQLite，使用方法
description: FMDB 使用方法
---

> 优秀的第三方库，README 也是很优秀的，理解了 README，会对使用带来很多便利。

## ARC 和 MRC

项目中使用 ARC 还是 MRC，对使用 FMDB 都没有任何影响，FMDB 会在编译项目时自动匹配。

## 使用

在 FMDB 中有三个重要的类：

1. `FMDatabase`：是一个提供 SQLite 数据库的类，用于执行 SQL 语句。
2. `FMResultSet`：用在 `FMDatabase` 中执行查询的结果的类。
3. `FMDatabaseQueue`：在多线程下查询和更新数据库用到的类。
<p></p>

<!--more-->

## 数据库创建

`FMDatabase` 是通过一个 SQLite 数据库文件路径创建的，此路径可以是以下三者之一：

1. 一个文件的系统路径。磁盘中可以不存在此文件，因为如果不存在会自动为你创建。
2. 一个空的字符串 `@""`。会在临时位置创建一个空的数据库，当 `FMDatabase` 连接关闭时，该数据库会被删除。
3. `NULL`。会在内存中创建一个数据库，当 `FMDatabase` 连接关闭时，该数据库会被销毁。

<p></p>
```objc
// 创建数据库示例
FMDatabase *db = [FMDatabase databaseWithPath:@"/tmp/tmp.db"];
```

## 打开数据库

数据库必须是打开状态，才能与之交互。如果没有足够的资源和权限来打开\创建数据库，数据库会打开失败。

## 数据库更新

SQL 语句中除过 `SELECT` 语句都可以称之为更新操作。包括 `CREATE`，`UPDATE`，`INSERT`，`ALTER`，`COMMIT`，`BEGIN`，`DETACH`，`DROP`，`END`，`EXPLAIN`，`VACUUM`，`REPLACE` 等。一般只要不是以 `SELECT` 开头的 SQL 语句，都是更新语句。

执行更新语句后会返回一个 `BOOL` 值，返回 `YES` 表示执行更新语句成功，返回 `NO` 表示出现错误，可以通过调用 `-lastErrorMessage` 和 `-lastErrorCode` 方法获取更多错误信息。

## 数据库查询

通过调用 `-executeQuery...` 方法之一执行 `SELECT` 语句进行数据库查询操作。

执行查询操作后，如果成功会返回一个 `FMResultSet` 对象，反之会返回 `nil`。通过 `-lastErrorMessage` 和 `-lastErrorCode` 方法可以确定为什么会查询失败。

为了遍历查询结果，需要 `while()` 循环，然后逐条记录查看。在 FMDB 中，可以通过下面的简单方式实现：

```objc
FMResultSet *s = [db executeQuery:@"SELECT * FROM myTable"];
while ([s next]) {
	// 每天记录的检索值
}
```

即使只需要获取一个数据，也还是必须在访问查询结果前调用 `-[FMResultSet next]`。

```objc
// 示例
FMResultSet *s = [db executeQuery:@"SELECT COUNT(*) FROM myTable"];
if ([s next]) {
	int totalCount = [s intForColumnIndex:0];
}
```

`FMResultSet` 提供了很多方便的方法来查询数据：

* `intForColumn:`
* `longForColumn:`
* `longLongIntForColumn:`
* `boolForColumn:`
* `doubleForColumn:`
* `stringForColumn:`
* `dateForColumn:`
* `dataForColumn:`
* `dataNoCopyForColumn:`
* `UTF8StringForColumn:`
* `objectForColumn:`

<p></p>
这些方法都有一个 `{type}ForColumnIndex:` 变体，是基于列的位置来查询数据。

通常情况下，一个 `FMResultSet` 没有必要手动 `-close`，因为结果集合 (result set) 被释放或者源数据库关闭会自动关闭。

## 关闭数据库

当对数据库进行查询和更新操作完成后，需要调用 `-close` 关闭数据库 `FMDatabase` 的连接。

```objc
// 示例
[db close];
```

## 事务

`FMDatabase` 可以通过调用方法来开始和提交事务，也可以通过执行开始\结束事务  (begin\end transaction) 语句。

## 多语句和批处理

`FMDatabase` 可以通过 `-executeStatements:withResultBlock:` 方法在一个字符串中执行多语句。

```objc
// 示例
NSString *sql = @"create table bulktest1 (id integer primary key autoincrement, x text);"
                 "create table bulktest2 (id integer primary key autoincrement, y text);"
                 "create table bulktest3 (id integer primary key autoincrement, z text);"
                 "insert into bulktest1 (x) values ('XXX');"
                 "insert into bulktest2 (y) values ('YYY');"
                 "insert into bulktest3 (z) values ('ZZZ');";

success = [db executeStatements:sql];

sql = @"select count(*) as count from bulktest1;"
       "select count(*) as count from bulktest2;"
       "select count(*) as count from bulktest3;";

success = [self.db executeStatements:sql withResultBlock:^int(NSDictionary *dictionary) {
    NSInteger count = [dictionary[@"count"] integerValue];
    XCTAssertEqual(count, 1, @"expected one record for dictionary %@", dictionary);
    return 0;
}];
```

## 数据处理

当给 FMDB 提供 SQL 语句时，在插入前不应该处理任何数据，而应该使用标准的 SQLite 的绑定语法。

```sql
// 示例
INSERT INTO myTable VALUES (?, ?, ?)
```

`?` 问号在 SQLite 中意为即将插入的值的占位符，FMDB 执行语句的方法都接受多个参数 (或者参数集合，比如 `NSArray`，`NSDictionary`，`va_list`)，它们都会正确转义。

也可以使用命名参数语法：

```sql
// 示例
INSERT INTO myTable VALUES (:id, :name, :value)
```

这些参数必须以冒号开头，SQLite 自身支持其他字符，但是命名时字典的键内部以冒号开头，就不能在你的字典的键中包含冒号。

```objc
// 示例
NSDictionary *argsDict = [NSDictionary dictionaryWithObjectsAndKeys:@"My Name", @"name", nil];
[db executeUpdate:@"INSERT INTO myTable (name) VALUES (:name)" withParameterDictionary:argsDict];
```

因此，不应该写类似下面这行一样的错误代码：

```objc
// 错误示例
[db executeUpdate:[NSString stringWithFormat:@"INSERT INTO myTable VALUES (%@)", @"this has \" lots of ' bizarre \" quotes '"]];
```

而应该这样写：

```objc
// 正确示例
[db executeUpdate:@"INSERT INTO myTable VALUES (?)", @"this has \" lots of ' bizarre \" quotes '"];
```

所有传递给 `-executeUpdate:` 方法的参数都必须是对象。下面写法执行不会起作用而且会引发崩溃：

```objc
// 错误示例
[db executeUpdate:@"INSERT INTO myTable VALUES (?)", 42];
```

插入一个数的正确方法是把这个数字包装成 `NSNumber` 对象：

```objc
// 正确示例
[db executeUpdate:@"INSERT INTO myTable VALUES (?)", [NSNumber numberWithInt:42]];
```

也可以使用 `-execute*WithFormat:` 这个方法将数字转换成字符串：

```objc
// 转换成字符串示例
[db executeUpdateWithFormat:@"INSERT INTO myTable VALUES (@d)", 42];
```

`-execute*WithFormat:` 这些方法后面都可以接格式字符串参数，以下 % 百分号格式符都是可以识别的：`%@`, `%c`, `%s`, `%d`, `%D`, `%i`, `%u`, `%U`, `%hi`, `%hu`, `%qi`, `%qu`, `%f`, `%g`, `%ld`, `%lu`, `%lld`, `%llu`。使用其他格式符可能会出现不可预知的问题。出于某种原因，可能需要在你的 SQL 语句中使用 `%` 字符，应该使用百分号转义一下 `%%`。

## FMDatabaseQueue 队列和线程安全

在多线程中同时使用 FMDatabase 单例是极其错误的想法，会导致每个线程创建一个 FMDatabase 对象。不要跨线程使用单例，也不要同时跨多线程，不然会奔溃或者异常。

*因此不要实例化一个 FMDatabase 单例来跨线程使用。*

相反，使用 FMDatabaseQueue，下面就是它的使用方法：

第一，创建队列。

```objc
// 创建 FMdatabaseQueue 示例
FMDatabaseQueue *queue = [FMDatabaseQueue databaseQueueWithPath:aPath];
```

然后这样使用：

```objc
// 示例
[queue inDatabase:^(FMDatabase *db) {
	[db executeUpdate:@"INSERT INTO myTable VALUES (?)", [NSNumber numberWithInt:1]];
	[db executeUpdate:@"INSERT INTO myTable VALUES (?)", [NSNumber numberWithInt:2]];
	[db executeUpdate:@"INSERT INTO myTable VALUES (?)", [NSNumber numberWithInt:3]];
	
	FMResultSet *rs = [db executeQuery:@"select * from foo"];
	while ([rs next]) {
		...
	}
}];
```

把操作放在事务中也很简单，比如：

```objc
// 示例
[queue inTransaction:^(FMDatabase *db, BOOL *rollback) {
	[db executeUpdate:@"INSERT INTO myTable VALUES (?)", [NSNumber numberWithInt:1]];
	[db executeUpdate:@"INSERT INTO myTable VALUES (?)", [NSNumber numberWithInt:2]];
	[db executeUpdate:@"INSERT INTO myTable VALUES (?)", [NSNumber numberWithInt:3]];
	
	if (whoopsSomethingWrongHappened) {
		*rollback = YES;
		return;
	}
	// ...
	[db executeUpdate:@"INSERT INTO myTable VALUES (?)", [NSNumber numberWithInt:4]];
}];
```

FMDatabase 将块代码 block 运行在一个串行队列上，即使在多线程同时调用 FMDatabaseQueue 的方法，它们仍然还是顺序执行。这种查询和更新方式不会影响其它，是线程安全的。

## 基于 block 自定义 SQLite 函数

这是可以的，例子可以在 main.m 中的 `makeFunctionNamed:` 方法查看。

## Swift

在 Swift 项目中也可以使用 FMDB，需要做以下步骤：

1. 将 FMDB 的 `.m` 和 `.h` 全部文件拖进你的项目。
2. 如果 Xcode 提示创建桥接文件，需要点击创建。如果没有提示，且项目中也没有桥接文件，需要手动添加。[点此查看桥接文件更多信息](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/BuildingCocoaApps/MixandMatch.html#//apple_ref/doc/uid/TP40014216-CH10-XID_76)
3. 在桥接文件中，添加这行代码：`#import "FMDB.h"`
4. 可以从 "src/extra/Swift Extension" 文件夹中拷贝 `FMDatabaseVariadic.swift` 文件到项目中，就可以使用 `executeUpdate` 和 `executeQuery` 多参数了。

<p></p>
做完上述几步，就可以使用 FMDatabase 写 Swift 代码了。

```swift
// 示例
let documentsFolder = NSSearchPathForDirectoriesInDomains(.DocumentDirectory, .UserDomainMask, true)[0] as String
let path = documentsFolder.stringByAppendingPathComponent("test.sqlite")

let database = FMDatabase(path: path)

if !database.open() {
    println("Unable to open database")
    return
}

if !database.executeUpdate("create table test(x text, y text, z text)", withArgumentsInArray: nil) {
    println("create table failed: \(database.lastErrorMessage())")
}

if !database.executeUpdate("insert into test (x, y, z) values (?, ?, ?)", withArgumentsInArray: ["a", "b", "c"]) {
    println("insert 1 table failed: \(database.lastErrorMessage())")
}

if !database.executeUpdate("insert into test (x, y, z) values (?, ?, ?)", withArgumentsInArray: ["e", "f", "g"]) {
    println("insert 2 table failed: \(database.lastErrorMessage())")
}

if let rs = database.executeQuery("select x, y, z from test", withArgumentsInArray: nil) {
    while rs.next() {
        let x = rs.stringForColumn("x")
        let y = rs.stringForColumn("y")
        let z = rs.stringForColumn("z")
        println("x = \(x); y = \(y); z = \(z)")
    }
} else {
    println("select failed: \(database.lastErrorMessage())")
}

database.close()
```

本文实际是对 [FMDB](https://github.com/ccgus/fmdb) 的 README 简单翻译，以方便使用 FMDB。




