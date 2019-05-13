---
title: 使用 Ruby 连接到 Azure Database for PostgreSQL - 单一服务器
description: 本快速入门提供了可用于从 Azure Database for PostgreSQL - 单一服务器连接和查询数据的 Ruby 代码示例。
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.custom: mvc
ms.devlang: ruby
ms.topic: quickstart
ms.date: 5/6/2019
ms.openlocfilehash: 242f5724e81f9b8ac6177e5c9d3844b3fab9a7f2
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65067283"
---
# <a name="azure-database-for-postgresql---single-server-use-ruby-to-connect-and-query-data"></a>Azure Database for PostgreSQL - 单一服务器：使用 Ruby 连接和查询数据
本快速入门演示了如何使用 [Ruby](https://www.ruby-lang.org) 应用程序连接到 Azure Database for PostgreSQL。 同时还介绍了如何使用 SQL 语句在数据库中查询、插入、更新和删除数据。 本文中的步骤假定你熟悉如何使用 Ruby 进行开发，但不熟悉如何使用 Azure Database for PostgreSQL。

## <a name="prerequisites"></a>先决条件
此快速入门使用以下任意指南中创建的资源作为起点：
- [创建 DB - 门户](quickstart-create-server-database-portal.md)
- [创建 DB - Azure CLI](quickstart-create-server-database-azure-cli.md)

还需要安装：
- [Ruby](https://www.ruby-lang.org/en/downloads/)
- Ruby pg（适用于 Ruby 的 PostgreSQL 模块）

## <a name="get-connection-information"></a>获取连接信息
获取连接到 Azure Database for PostgreSQL 所需的连接信息。 需要完全限定的服务器名称和登录凭据。

1. 登录到 [Azure 门户](https://portal.azure.com/)。
2. 在 Azure 门户的左侧菜单中，单击“所有资源”，然后搜索已创建的服务器（例如 mydemoserver）。
3. 单击服务器名称。
4. 从服务器的“概览”面板中记下“服务器名称”和“服务器管理员登录名”。 如果忘记了密码，也可通过此面板来重置密码。
 ![Azure Database for PostgreSQL 服务器名称](./media/connect-ruby/1-connection-string.png)

> [!NOTE]
> Azure Postgres 用户名中的 `@` 符号已在所有连接字符串中以 url 编码为 `%40`。 

## <a name="connect-and-create-a-table"></a>进行连接并创建表
使用以下代码进行连接，使用 **CREATE TABLE** SQL 语句创建表，然后使用 **INSERT INTO** SQL 语句将行添加到表中。

该代码使用 [PG::Connection](https://www.rubydoc.info/gems/pg/PG/Connection) 对象和构造函数 [new()](https://www.rubydoc.info/gems/pg/PG%2FConnection:initialize) 来连接到 Azure Database for PostgreSQL。 然后调用 [exec()](https://www.rubydoc.info/gems/pg/PG/Connection#exec-instance_method) 方法，以便运行 DROP、CREATE TABLE 和 INSERT INTO 命令。 代码使用 [PG::Error](https://www.rubydoc.info/gems/pg/PG/Error) 类来检查是否存在错误。 然后，它会调用方法 [close()](https://www.rubydoc.info/gems/pg/PG/Connection#lo_close-instance_method)，在终止之前关闭连接。

将 `host`、`database`、`user` 和 `password` 字符串替换为你自己的值。 


```ruby
require 'pg'

begin
    # Initialize connection variables.
    host = String('mydemoserver.postgres.database.azure.com')
    database = String('postgres')
    user = String('mylogin%40mydemoserver')
    password = String('<server_admin_password>')

    # Initialize connection object.
    connection = PG::Connection.new(:host => host, :user => user, :dbname => database, :port => '5432', :password => password)
    puts 'Successfully created connection to database'

    # Drop previous table of same name if one exists
    connection.exec('DROP TABLE IF EXISTS inventory;')
    puts 'Finished dropping table (if existed).'

    # Drop previous table of same name if one exists.
    connection.exec('CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);')
    puts 'Finished creating table.'

    # Insert some data into table.
    connection.exec("INSERT INTO inventory VALUES(1, 'banana', 150)")
    connection.exec("INSERT INTO inventory VALUES(2, 'orange', 154)")
    connection.exec("INSERT INTO inventory VALUES(3, 'apple', 100)")
    puts 'Inserted 3 rows of data.'

rescue PG::Error => e
    puts e.message 
    
ensure
    connection.close if connection
end
```

## <a name="read-data"></a>读取数据
使用以下代码进行连接，并使用 **SELECT** SQL 语句来读取数据。 

该代码使用 [PG::Connection](https://www.rubydoc.info/gems/pg/PG/Connection) 对象和构造函数 [new()](https://www.rubydoc.info/gems/pg/PG%2FConnection:initialize) 来连接到 Azure Database for PostgreSQL。 然后，它会调用方法 [exec()](https://www.rubydoc.info/gems/pg/PG/Connection#exec-instance_method) 来运行 SELECT 命令，将结果保存在结果集中。 结果集集合使用 `resultSet.each do` 循环进行循环访问，将最新的行值保存在 `row` 变量中。 代码使用 [PG::Error](https://www.rubydoc.info/gems/pg/PG/Error) 类来检查是否存在错误。 然后，它会调用方法 [close()](https://www.rubydoc.info/gems/pg/PG/Connection#lo_close-instance_method)，在终止之前关闭连接。

将 `host`、`database`、`user` 和 `password` 字符串替换为你自己的值。 

```ruby
require 'pg'

begin
    # Initialize connection variables.
    host = String('mydemoserver.postgres.database.azure.com')
    database = String('postgres')
    user = String('mylogin%40mydemoserver')
    password = String('<server_admin_password>')

    # Initialize connection object.
    connection = PG::Connection.new(:host => host, :user => user, :database => dbname, :port => '5432', :password => password)
    puts 'Successfully created connection to database.'

    resultSet = connection.exec('SELECT * from inventory;')
    resultSet.each do |row|
        puts 'Data row = (%s, %s, %s)' % [row['id'], row['name'], row['quantity']]
    end

rescue PG::Error => e
    puts e.message 
    
ensure
    connection.close if connection
end
```

## <a name="update-data"></a>更新数据
使用以下代码进行连接，并使用 **UPDATE** SQL 语句更新数据。

该代码使用 [PG::Connection](https://www.rubydoc.info/gems/pg/PG/Connection) 对象和构造函数 [new()](https://www.rubydoc.info/gems/pg/PG%2FConnection:initialize) 来连接到 Azure Database for PostgreSQL。 然后，它会调用方法 [exec()](https://www.rubydoc.info/gems/pg/PG/Connection#exec-instance_method) 来运行 UPDATE 命令。 代码使用 [PG::Error](https://www.rubydoc.info/gems/pg/PG/Error) 类来检查是否存在错误。 然后，它会调用方法 [close()](https://www.rubydoc.info/gems/pg/PG/Connection#lo_close-instance_method)，在终止之前关闭连接。

将 `host`、`database`、`user` 和 `password` 字符串替换为你自己的值。 

```ruby
require 'pg'

begin
    # Initialize connection variables.
    host = String('mydemoserver.postgres.database.azure.com')
    database = String('postgres')
    user = String('mylogin%40mydemoserver')
    password = String('<server_admin_password>')

    # Initialize connection object.
    connection = PG::Connection.new(:host => host, :user => user, :dbname => database, :port => '5432', :password => password)
    puts 'Successfully created connection to database.'

    # Modify some data in table.
    connection.exec('UPDATE inventory SET quantity = %d WHERE name = %s;' % [200, '\'banana\''])
    puts 'Updated 1 row of data.'

rescue PG::Error => e
    puts e.message 
    
ensure
    connection.close if connection
end
```


## <a name="delete-data"></a>删除数据
使用以下代码进行连接，并使用 **DELETE** SQL 语句读取数据。 

该代码使用 [PG::Connection](https://www.rubydoc.info/gems/pg/PG/Connection) 对象和构造函数 [new()](https://www.rubydoc.info/gems/pg/PG%2FConnection:initialize) 来连接到 Azure Database for PostgreSQL。 然后，它会调用方法 [exec()](https://www.rubydoc.info/gems/pg/PG/Connection#exec-instance_method) 来运行 UPDATE 命令。 代码使用 [PG::Error](https://www.rubydoc.info/gems/pg/PG/Error) 类来检查是否存在错误。 然后，它会调用方法 [close()](https://www.rubydoc.info/gems/pg/PG/Connection#lo_close-instance_method)，在终止之前关闭连接。

将 `host`、`database`、`user` 和 `password` 字符串替换为你自己的值。 

```ruby
require 'pg'

begin
    # Initialize connection variables.
    host = String('mydemoserver.postgres.database.azure.com')
    database = String('postgres')
    user = String('mylogin%40mydemoserver')
    password = String('<server_admin_password>')

    # Initialize connection object.
    connection = PG::Connection.new(:host => host, :user => user, :dbname => database, :port => '5432', :password => password)
    puts 'Successfully created connection to database.'

    # Modify some data in table.
    connection.exec('DELETE FROM inventory WHERE name = %s;' % ['\'orange\''])
    puts 'Deleted 1 row of data.'

rescue PG::Error => e
    puts e.message 
    
ensure
    connection.close if connection
end
```

## <a name="next-steps"></a>后续步骤
> [!div class="nextstepaction"]
> [使用导出和导入功能迁移数据库](./howto-migrate-using-export-and-import.md)
