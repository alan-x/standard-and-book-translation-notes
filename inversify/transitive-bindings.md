### 传递绑定

一个传递类型绑定允许声明一个类型声明被前面类型绑定声明解析。

一个传递类型绑定可以使用`toService`方法声明：

```ts
@injectable()
class MySqlDatabaseTransactionLog {
    public time: number;
    public name: string;
    public constructor() {
        this.time = new Date().getTime();
        this.name = "MySqlDatabaseTransactionLog";
    }
}

@injectable()
class DatabaseTransactionLog {
    public time: number;
    public name: string;
}

@injectable()
class TransactionLog {
    public time: number;
    public name: string;
}

const container = new Container();
container.bind(MySqlDatabaseTransactionLog).toSelf().inSingletonScope();
container.bind(DatabaseTransactionLog).toService(MySqlDatabaseTransactionLog);
container.bind(TransactionLog).toService(DatabaseTransactionLog);

const mySqlDatabaseTransactionLog = container.get(MySqlDatabaseTransactionLog);
const databaseTransactionLog = container.get(DatabaseTransactionLog);
const transactionLog = container.get(TransactionLog);

expect(mySqlDatabaseTransactionLog.name).to.eq("MySqlDatabaseTransactionLog");
expect(databaseTransactionLog.name).to.eq("MySqlDatabaseTransactionLog");
expect(transactionLog.name).to.eq("MySqlDatabaseTransactionLog");
expect(mySqlDatabaseTransactionLog.time).to.eq(databaseTransactionLog.time);
expect(databaseTransactionLog.time).to.eq(transactionLog.time);
```

当然一个`multiBindToService`工具函数允许我们去声明多个传递绑定到一个中。

例子，与其下面这么写：
```ts
container.bind(DatabaseTransactionLog).toService(MySqlDatabaseTransactionLog);
container.bind(TransactionLog).toService(DatabaseTransactionLog);
```

我们可以使用`multiBindToService`如下编写：
```ts
multiBindToService(container)(MySqlDatabaseTransactionLog)
    (DatabaseTransactionLog, TransactionLog);

```