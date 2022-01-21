# Specification: Ballerina PostgreSQL Library

_Owners_: @daneshk @niveathika  
_Reviewers_: @daneshk  
_Created_: 2022/01/14   
_Updated_: 2022/01/14  
_Issue_: [#2291](https://github.com/ballerina-platform/ballerina-standard-library/issues/2291)

# Introduction

This is the specification for PostgreSQL standard library, which provides the functionality that is required to access and
manipulate data stored in a PostgreSQL database in the [Ballerina programming language](https://ballerina.io/),
which is an open-source programming language for the cloud that makes it easier to use, combine, and create network
services.

# Contents

1. [Overview](#1-overview)
2. [Client](#2-client)  
   2.1. [Connection Pool Handling](#21-connection-pool-handling)  
   2.2. [Closing the Client](#22-closing-the-client)
3. [Queries and Values](#3-queries-and-values)
4. [Database Operations](#4-database-operations)

# 1. Overview

This specification elaborates on usage of PostgreSQL `Client` object to interface with an PostgreSQL database.

`Client` supports five database operations as follows,
1. Executes the query, which may return multiple results.
2. Executes the query, which is expected to return at most one row of the result.
3. Executes the SQL query. Only the metadata of the execution is returned.
4. Executes the SQL query with multiple sets of parameters in a batch. Only the metadata of the execution is returned.
5. Executes a SQL query, which calls a stored procedure. This can either return results or nil.

All the above operations make use of `sql:ParameterizedQuery` object, backtick surrounded string template to pass
SQL statements to the database. `sql:ParameterizedQuery` supports passing of Ballerina basic types or Typed SQL Values
such as `sql:CharValue`, `sql:BigIntValue`, etc. to indicate parameter types in SQL statements.

# 2. Client

Each client represents a pool of connections to the database. The pool of connections is maintained throughout the
lifetime of the client.

**Initialisation of the Client:**
```ballerina
# Initializes the PostgreSQL client.
#
# + host - Hostname of the PostgreSQL server
# + user - If the PostgreSQL server is secured, the username
# + password - The password associated with the provided username
# + database - The name of the database. The default is to connect to a database with the
#              same name as the user name
# + port - Port of the PostgreSQL server
# + options - The database-specific PostgreSQL client properties
# + connectionPool - The `sql:ConnectionPool` object to be used within the database client. If there is no
#                    `connectionPool` provided, the global connection pool will be used
# + return - An `sql:Error` if the client creation fails
public isolated function init(string host = "localhost", string? username = (), string? password = (), string? database = (),
            int port = 5432, Options? options = (), sql:ConnectionPool? connectionPool = ()) returns sql:Error?
```

**Configurations available for initializing the PostgreSQL client:**
* Connection properties:
  ```ballerina
  # Provides a set of configuration related to PostgreSQL database connection.
  #
  # + ssl - SSL Configuration to be used
  # + connectTimeout - Timeout (in seconds) to be used when connecting to the Oracle server
  # + socketTimeout - Socket timeout (in seconds) during the read/write operations with the Oracle server
  #                   (0 means no socket timeout)
  # + loginTimeout - Timeout (in seconds) when connecting to the Oracle server and authentication (0 means no timeout)
  # + rowFetchSize - The number of rows to be fetched by one trip to the database.
  # + cachedMetadataFieldsCount - Specifies the maximum number of fields to be cached per connection.
  #                           A value of 0 disables the cache.
  # + cachedMetadataFieldSize - Specifies the maximum size (in megabytes) of fields to be cached per connection.
  #                            A value of 0 disables the cache.
  # + preparedStatementThreshold - Determine the number of `PreparedStatement` executions required before switching
  #                                over to use server-side prepared statements.
  # + preparedStatementCacheQueries - Determine the number of queries that are cached in each connection.
  # + preparedStatementCacheSize - Determine the maximum size (in mebibytes) of the prepared queries.
  # + cancelSignalTimeout - Time (in seconds) by which, the cancel command is sent out of band over its own connection
  #                         so that the cancel message itself can get stuck. The default value is 10 seconds
  # + keepAliveTcpProbe - Enable or disable the TCP keep-alive probe 
  # + binaryTransfer - Use the binary format for sending and receiving data if possible
  public type Options record {|
      SecureSocket ssl = {};
      decimal connectTimeout = 0;
      decimal socketTimeout = 0;
      decimal loginTimeout = 0;
      int rowFetchSize?;
      int cachedMetadataFieldsCount?;
      int cachedMetadataFieldSize?;
      int preparedStatementThreshold?;
      int preparedStatementCacheQueries?;
      int preparedStatementCacheSize?;
      decimal cancelSignalTimeout = 10;
      boolean keepAliveTcpProbe?;
      boolean binaryTransfer?;
  |};
  ``` 
* SSL Connection:
  ```
  # The SSL configuration to be used when connecting to the PostgreSQL server.
  #
  # + mode - The `SSLMode` to be used during the connection (PREFER/REQUIRE/DISABLE/ALLOW/VERIFY_CA/VERIFY_FULL)
  # + key - Keystore configuration of the client certificates
  # + rootcert - File name of the SSL root certificate. Defaults to the `defaultdir/root.crt`.
  #             in which the `defaultdir` is `${user.home}/.postgresql/` in Unix systems and
  #             `%appdata%/postgresql/` on Windows.
  public type SecureSocket record {|
      SSLMode mode = PREFER;
      string rootcert?;
      crypto:KeyStore|CertKey key?;
  |};

  # Represents the combination of the certificate, private key, and private key password if encrypted
  #
  # + certFile - A file containing the client certificate
  # + keyFile - A file containing the client private key
  # + keyPassword - Password of the private key if it is encrypted
  public type CertKey record {|
      string certFile;
      string keyFile;
      string keyPassword?;
  |};
  ```

## 2.1. Connection Pool Handling

Connection Pool Handling is generic and implemented through `sql` module. For more information, see the
[SQL Specification](https://github.com/ballerina-platform/module-ballerina-sql/blob/master/docs/spec/spec.md#21-connection-pool-handling)

## 2.2. Closing the Client

Once all the database operations are performed, the client can be closed by invoking the `close()`
operation. This will close the corresponding connection pool if it is not shared by any other database clients.

   ```ballerina
    # Closes the PostgreSQL client and shuts down the connection pool.
    #
    # + return - Possible error when closing the client
    public isolated function close() returns Error?;
   ```

# 3. Queries and Values

All the generic `sql` Queries and Values are supported. For more information, see the
[SQL Specification](https://github.com/ballerina-platform/module-ballerina-sql/blob/master/docs/spec/spec.md#3-queries-and-values)

In addition to `sql` Values, `postgresql` package supports the following Typed Values for PostgreSQL Data Types,
1. InetValue
2. CidrValue
3. MacAddrValue
4. MacAddr8Value
5. PointValue
6. LineValue
7. LineSegmentValue
8. BoxValue
9. PathValue
10. PolygonValue
11. CircleValue
12. UuidValue
13. TsVectorValue
14. TsQueryValue
15. JsonValue
16. JsonBinaryValue
17. JsonPathValue
18. IntervalValue
19. IntegerRangeValue
20. LongRangeValue
21. NumericRangeValue
22. TsRangeValue
23. TsTzRangeValue
24. DateRangeValue
25. PglsnValue
26. BitStringValue
27. VarBitStringValue
28. PGBitValue
29. MoneyValue
30. RegClassValue
31. RegConfigValue
32. RegDictionaryValue
33. RegNamespaceValue
34. RegOperValue
35. RegOperatorValue
36. RegProcValue
37. RegProcedureValue
38. RegRoleValue
39. RegTypeValue
40. PGXmlValue
41. CustomTypeValue
42. EnumValue

All the above values are supported as Typed OutParameters for `call()` operation.

# 4. Database Operations

`Client` supports five database operations as follows,
1. Executes the query, which may return multiple results.
2. Executes the query, which is expected to return at most one row of the result.
3. Executes the SQL query. Only the metadata of the execution is returned.
4. Executes the SQL query with multiple sets of parameters in a batch. Only the metadata of the execution is returned.
5. Executes a SQL query, which calls a stored procedure. This can either return results or nil.

For more information on Database Operations see the [SQL Specification](https://github.com/niveathika/module-ballerina-sql/blob/master/docs/spec/spec.md#4-database-operations)