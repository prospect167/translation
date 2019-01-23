# Chapter 6 Connector/J Reference

# MySQL-JDBC驱动-连接参数说明

## 6.1 Driver/Datasource Class Name

## 6.1 Driver/Datasource 对应的类名

The name of the class that implements `java.sql.Driver` in MySQL Connector/J is `com.mysql.cj.jdbc.Driver`.

在MySQL的JDBC驱动,`Connector/J`中, 5.1版本对应的`java.sql.Driver`实现类是 `com.mysql.jdbc.Driver`, 当然, 到8.0版本底层实现改为 `com.mysql.cj.jdbc.Driver`, 通过 extends 实现兼容.

> `com.mysql.jdbc.Driver extends com.mysql.cj.jdbc.Driver`


## 6.2 Connection URL Syntax

## 6.2 连接地址的URL格式

This section explains the syntax of the URLs for connecting to MySQL.

本节主要介绍JDBC连接MySQL数据库的URL语法格式.

This is the generic format of the connection URL:

通用的connection URL格式如下:

```
protocol//[hosts][/database][?properties]
```

The URL consists of the following parts:

包含以下部分:

> #### Important
> #### 提示
>
> Any reserved characters for URLs (for example, /, :, @, (, ), [, ], &, #, =, ?, and space) that appear in any part of the connection URL must be percent encoded.
>
> 在连接URL的各部分中, 所有保留字符都必须进行百分号转码(percent encoded), 例如, `/`, `:`, `@`, `(`, `)`, `[`, `]`, `&`, `#`, `=`, `?`, 以及英文空格).

### `protocol`

There are four possible protocols for a connection:

MySQL-JDBC支持4种协议:

- `jdbc:mysql:` is for ordinary and basic failover connections.
- `jdbc:mysql:loadbalance:` is for configuring load balancing.
- `jdbc:mysql:replication:` is for configuring a replication setup.
- `mysqlx:` is for connections using the X Protocol.

- `jdbc:mysql:` 是最基本连接方式
- `jdbc:mysql:loadbalance:` 用于配置负载均衡.
- `jdbc:mysql:replication:` 用于设置主从(replication).
- `mysqlx:` 对应使用扩展协议(X Protocol)的连接.

### `hosts`

Depending on the situation, the `hosts` part may consist simply of a host name, or it can be a complex structure consisting of various elements like multiple host names, port numbers, host-specific properties, and user credentials.

根据具体情况确定, `hosts`部分可以是简单的主机名称; 也可能是包含多个元素的组合, 比如多个主机名端口号,特定属性的字符串,以及用户认证信息等。

#### Single host:

#### 单个服务器(host)的情况:

- Single-host connections without adding host-specific properties:
- 单服务器(Single-host), 且不指定特定属性(host-specific properties)的情况:

  - The `hosts` part is written in the format of `host:port`. This is an example of a simple single-host connection URL:
  - `hosts`部分的格式为 `host:port`. 例如:

    ```
    jdbc:mysql://host1:33060/sakila
    ```

  - `host` can be an IPv4 or an IPv6 host name string, and in the latter case it must be put inside square brackets, for example “[1000:2000::abcd].” When `host` is not specified, the default value of `localhost` is used.
  - `host` 可以是域名, IPv4地址; 或者IPv6地址（以方括号括起来,如 `[1000:2000::abcd]`之类.） 如果不指定 `host`, 则使用默认值 `localhost`.

  - `port` is a standard port number, i.e., an integer between `1` and `65535`. The default port number for an ordinary MySQL connection is `3306`, and it is `33060` for a connection using the X Protocol. If `port` is not specified, the corresponding default is used.
  - `port` 部分就是一个TCP端口号, 取值范围从`1` 到 `65535`. 普通MySQL连接的端口号默认值为 `3306`, 如果使用X Protocol, 则默认值为`33060`. 如果不指定端口号, 则使用对应的默认值.
​

- Single-host connections adding host-specific properties:
- 单服务器(Single-host), 并指定特定属性(host-specific properties)的情况:

  - In this case, the host is defined as a succession of `key=value` pairs. Keys are used to identify the host, the port, as well as any host-specific properties. There are two alternate formats for specifying keys:
  - 这种情况下, host部分由多个`key=value`对组成. Key部分指定host, port,以及特定属性. 支持两种格式:

    - The “address-equals” form:
    - “address-equals”方式:

      ```
      address=(host=host_or_ip)(port=port)(key1=value1)(key2=value2)...(keyN=valueN)
      ```

      Here is a sample URL using the“address-equals” form :
      “address-equals”示例:

      ```
      jdbc:mysql://address=(host=myhost)(port=1111)(key1=value1)/db
      ```

    - The “key-value” form:
    - “key-value” 方式:

      ```
      (host=host,port=port,key1=value1,key2=value2,...,keyN=valueN)
      ```

      Here is a sample URL using the “key-value” form :
      “key-value” 示例:

      ```
      jdbc:mysql://(host=myhost,port=1111,key1=value1)/db
      ```

      ​

  - The host and the port are identified by the keys `host` and `port`. The descriptions of the format and default values of`host` and `port` in [Single host without host-specific properties](https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-reference-jdbc-url-format.html#connector-j-url-single-host-without-props) above also apply here.

  - Other keys that can be added include `user`, `password`, `protocol`, and so on. They override the global values set in the[*properties* part of the URL](https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-reference-jdbc-url-format.html#connector-j-url-properties). Limit the overrides to user, password, network timeouts, and statement and metadata cache sizes; the effects of other per-host overrides are not defined.

  - Different protocols may require different keys. For example, the `mysqlx:` scheme uses two special keys, *address* and *priority*. *address* is a `host`:`port` pair and *priority* an integer. For example:

      ```
      mysqlx://(address=host:1111,priority=1,key1=value1)/db
      ```

  - `key` is case-sensitive. Two keys differing in case only are considered conflicting, and there are no guarantees on which one will be used.

#### Multiple hosts

  There are two formats for specifying multiple hosts:

- List hosts in a comma-separated list:

    ```
    host1,host2,...,hostN
    ```

    Each host can be specified in any of the three ways described in [Single host](https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-reference-jdbc-url-format.html#connector-j-url-single-host) above. Here are some examples:

    ```
    jdbc:mysql://myhost1:1111,myhost2:2222/db
    jdbc:mysql://address=(host=myhost1)(port=1111)(key1=value1),address=(host=myhost2)(port=2222)(key2=value2)/db
    jdbc:mysql://(host=myhost1,port=1111,key1=value1),(host=myhost2,port=2222,key2=value2)/db
    jdbc:mysql://myhost1:1111,(host=myhost2,port=2222,key2=value2)/db
    mysqlx://(address=host1:1111,priority=1,key1=value1),(address=host2:2222,priority=2,key2=value2)/db
    ```

- List hosts in a comma-separated list, and then encloses the list by square brackets:

    ```
    [host1,host2,...,hostN]
    ```

    This is called the host sublist form, which allows sharing of the [user credentials](https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-reference-jdbc-url-format.html#connector-j-url-user-credentials) by all hosts in the list as if they are a single host. Each host in the list can be specified in any of the three ways described in [Single host](https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-reference-jdbc-url-format.html#connector-j-url-single-host) above. Here are some examples:

    ```
    jdbc:mysql://sandy:secret@[myhost1:1111,myhost2:2222]/db
    jdbc:mysql://sandy:secret@[address=(host=myhost1)(port=1111)(key1=value1),address=(host=myhost2)(port=2222)(key2=value2)]/db
    jdbc:mysql://sandy:secret@[myhost1:1111,address=(host=myhost2)(port=2222)(key2=value2)]/db
    ```

    While it is not possible to write host sublists recursively, a host list may contain host sublists as its member hosts.

#### User credentials

  User credentials can be set outside of the connection URL—for example, as arguments when getting a connection from the`java.sql.DriverManager` (see [Section 6.3, “Configuration Properties”](https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-reference-configuration-properties.html) for details). When set with the connection URL, there are several ways to specify them:

- Prefix the a single host, a host sublist (see [Multiple hosts](https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-reference-jdbc-url-format.html#connector-j-url-multiple-hosts)), or any host in a list of hosts with the user credentials with an `@`:

    ```
     user:password@host_or_host_sublist
    ```

    For example:

    ```
    mysqlx://sandy:secret@[(address=host1:1111,priority=1,key1=value1),(address=host2:2222,priority=2,key2=value2))]/db
    ```

- Use the keys `user` and `password` to specify credentials for each host:

    ```
    (user=sandy)(password=mypass)
    ```

    For example:

    ```
    jdbc:mysql://[(host=myhost1,port=1111,user=sandy,password=secret),(host=myhost2,port=2222,user=finn,password=secret)]/db
    jdbc:mysql://address=(host=myhost1)(port=1111)(user=sandy)(password=secret),address=(host=myhost2)(port=2222)(user=finn)(password=secret)/db
    ```

  In both forms, when multiple user credentials are specified, the one to the left takes precedence—that is, going from left to right in the connection string, the first one found that is applicable to a host is the one that is used.

  *Inside* a host sublist, no host can have user credentials in the @ format, but individual host can have user credentials specified in the key format.

### `database`

The default database or catalog to open. If the database is not specified, the connection is made with no default database. In this case, either call the `setCatalog()` method on the `Connection` instance, or specify table names using the database name (that is, `SELECT *dbname*.*tablename*.*colname* FROM dbname.tablename...`) in your SQL statements. Opening a connection without specifying the database to use is, in general, only useful when building tools that work with multiple databases, such as GUI database managers.

Note

Always use the `Connection.setCatalog()` method to specify the desired database in JDBC applications, rather than the `USE *database*` statement.

### `properties`

A succession of global properties applying to all hosts, preceded by `?` and written as `*key*=*value*` pairs separated by the symbol “`&.`”Here are some examples:

```
jdbc:mysql://(host=myhost1,port=1111),(host=myhost2,port=2222)/db?key1=value1&key2=value2&key3=value3
```

The following are true for the key-value pairs:

- *key* and *value* are just strings. Proper type conversion and validation are performed internally in Connector/J.
- *key* is case-sensitive. Two keys differing in case only are considered conflicting, and it is uncertain which one will be used.
- Any host-specific values specified with key-value pairs as explained in [Single host with host-specific properties](https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-reference-jdbc-url-format.html#connector-j-url-single-host-with-props) and [Multiple hosts](https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-reference-jdbc-url-format.html#connector-j-url-multiple-hosts) above override the global values set here.

See [Section 6.3, “Configuration Properties”]() for details about the configuration properties.



```
-- 6.1 Driver/Datasource Class Name
-- 6.2 Connection URL Syntax
6.3 Configuration Properties
6.4 JDBC API Implementation Notes
6.5 Java, JDBC, and MySQL Types
6.6 Using Character Sets and Unicode
6.7 Connecting Securely Using SSL
6.8 Connecting Using Unix Domain Sockets
6.9 Connecting Using Named Pipes
6.10 Connecting Using PAM Authentication
6.11 Using Master/Slave Replication with ReplicationConnection
6.12 Mapping MySQL Error Numbers to JDBC SQLState Codes
```

<https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-reference.html>