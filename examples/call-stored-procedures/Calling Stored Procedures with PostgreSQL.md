# Overview

The `call-stored-operation` project demonstrates how to use the PostgreSQL client to execute a stored procedure.

# Prerequisite

* Install the PostgreSQL server and create a database 

* Add required configurations in the `Config.toml` file 

# Run the example
 
To run the example, move into the `call-stored-operation` project and execute the below command.
 
```shell
bal run
```
It will build the `call-stored-operation` Ballerina project and then run it.

# Output of the example

This gives the following output when running this project.

```shell
Call stored procedure `InsertStudent`.
Inserted data: {"id":1,"age":24,"name":"George"}
Call stored procedure `GetCount`.
Age of the student with id '1' : 24
Total student count: 1
```
