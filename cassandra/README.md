# Cassandra Overview

Apache Cassandra is an open-source NoSQL database by Apache foundation.

## Contents

TODO


## SQL DB vs Apache Cassandra

1. Cassandra follows its own modeling methodology where it puts the needs of the application before that of the data. In SQL dbs, we first think of the data, create a data model based on it and then think of the application. In Cassandra, application and its needs are thought about first. Once the queries are in place, the data is shaped around them and then the model is created and stored in the DB.
2. Cassandra is optimized to give high performance to queries while SQL dbs give more preference to entities (data) than the query performance.
3. Primary keys are used for uniqueness in SQL dbs where as primary keys are also used in optimzing performance on a large scale.
4. Cassandra by default uses distributed architecture while relational DBs don't (by default).
5. Cassandra doesn't support ACID (Atomicity, Consistency, Isolation and Durability) properties and has data denormalized inorder to have less consistency, in favor of higher availability ([CAP theorem](https://github.com/kalyan-ch/interview-prep-system-design/blob/main/system-design-concepts.md#cap-theorem)). However, inserts, updates and deletes are atomic, isolated and durable.
6. There are no joins in Cassandra as they would have negative impact on latency. Denormalization is used instead.

## Terminology

These are the terms used in Cassandra with specific connotations:

1. Data Model - abstract model for organizing elements of the data.
2. Keyspace - outermost logical container of the table. It stores tables and replica data. Similar to DB schema, keyspace defines replication settings.
3. Table - collection of rows and columns.
4. Partitions - are a collection of rows of a table stored on nodes of the DB based on partitioning strategy.
5. Key-Value pairs - each row stores data in key value pairs
6. Primary Key - a key that determines the uniqueness of data and also determines the location of data in the cluster. Every cassandra table has a primary key.
7. Partition key - first part of the primary key that identifies which partition the data belongs to.
8. Column - as also known as cells
9. Clustering column - determines the order in which rows are arranged inside a partition.

## Cassandra Query Language (CQL)

### Create keyspace

```
create KEYSPACE killrvideo WITHREPLICATION={'class':'SimpleStrategy', 'replication_factor':1};

use killrvideo;
```

### Create table

```
create TABLE users (
    user_id UUID,
    first_name TEXT,
    last_name TEXT,
    PRIMARY_KEY(user_id)
);
```

### Misc statements

Select - This is used to read data from a table - very similar to SQL.

```
select * from table limit 20;
```

Truncate - removes data from a table. data once removed can be restored from backup.

Alter - used to change datatype, add, drop or rename columns, change other table properties. Cannot be used to change primary key columns.

```
ALTER TABLE table_name ADD column_name2 INT;
```

Source - allows to execute CQL commands from a file. `source './script.cql'`


## Partitions

1. Partitions are collections of rows of a table that have been divided according to some strategy.
2. Partition keys allow us to locate data effectively.
3. Data in cassandra is organized and modeled according to queries that the application might use. So partition keys are determined by the specific use case that the app needs.
4. Instead of creating 3 different tables like in SQL for movie_titles, comments and users, Cassandra creates tables for use case. For example, comments_for_movie or comments_by_user.
5. This allows us to quickly retrieve the data that is needed instead of spending time on joining the data at the DB level - since Cassandra doesn't support joins. Based on the example above, partition key can be either the move in the first table or the user_id in the second table.


## Datatypes in Cassandra

Collections, counters and User Defined Types are the broad categories of datatypes in Cassandra.

### Collections

1. Collection types are used to group and store data together in a single column.
2. These columns are multi-valued columns and are designed to store a small amount of data.
3. They cannot be a part of primary key, partition key or clustering column.
4. Three types of Collections - set(stores unique values in no particular order, returns sorted order of values), list(stores values in a single cell in a particular order and returns according to an index) and map(allows to store related values in key-value pair). Examples below
4. Frozen type of collection allows for serialization of multiple components into a single value. Cassandra treats this as a blob, so entire value must be rewritten for updates.

```
CREATE TABLE users (
    user_id text,
    fname text,
    lname text,
    emails set<text>,
    destinations list<text>,
    itinerary map<timestamp, text>,
    PRIMARY_KEY(user_id)
);

INSERT INTO users(user_id, fname, lname, emails) values( 'wolfie', 'kalyan', 'c', {'kal123@gmail.com', 'kal250@gmail.com'} );

UPDATE users SET destinations = {'San Francisco', 'Denver', 'San Francisco'} where user_id='wolfie';

UDPATE users SET itinerary = {'2024-2-26':'San Francisco', '2024-2-27':'San Jose', '2024-2-28':'Los Angeles'} where user_id='wolfie';
```

### Counter Type

1. It is a singed 64 bit integer that supports two operations: increment and decrement (using update statement).
2. Counter is used to increase or decrease the value of a column.
3. This column cannot be a primary key.

```
    CREATE TABLE cow_count (
        ranch_name text,
        ranch_id int PRIMARY KEY,
        number_of_cows counter
    );

    UPDATE cow_count SET number_of_cows = number_of_cows + 10 where ranch_id='5';
```

### User Defined Types (UDTs)

1. UDTs can group together related info and can include any supported datatypes.
2. They allow to embed more complex data within a single column.

```
CREATE TYPE address (
    street text,
    apt text,
    city text,
    zip int,
    phone set<int>
);

CREATE TYPE fullname (
    first_name text,
    middle_name text,
    last_name text
);

CREATE TABLE users (
    user_id text PRIMARY KEY,
    full_name <fullname>,
    addresses map<text, <address>>
);
```

## Data Modeling

TODO
