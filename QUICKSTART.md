---
title: TiDB Quick Start Guide
category: quick start
---

# TiDB Quick Start Guide

## About TiDB

TiDB (The pronunciation is: /'taɪdiːbi:/ tai-D-B, etymology: titanium) is an open source distributed scalable Hybrid Transactional and Analytical Processing (HTAP) database built by PingCAP. Inspired by the design of Google F1 and Google Spanner, TiDB features infinite horizontal scalability, strong consistency, and high availability. The goal of TiDB is to serve as a one-stop solution for both OLTP (Online Transactional Processing) and OLAP (Online Analytical Processing).

## About this guide

This guide outlines how to perform a quick deployment of a TiDB cluster using TiDB-Ansible and walks you through the basic TiDB operations and administrations.

## Deploy the TiDB cluster

This section describes how to deploy a TiDB cluster. A TiDB cluster consists of different components: TiDB servers, TiKV servers, and Placement Driver (PD) servers.

The architecture is as follows:

![TiDB Architecture](media/tidb-architecture.png)

For details of deploying a TiDB cluster, see [Ansible Deployment](op-guide/ansible-deployment.md).

## Try TiDB

This section describes some basic CRUD operations in TiDB.

### Create, show, and drop a database

- To create a database, use the `CREATE DATABASE` statement. The Syntax is as follows:

    ```sql
    CREATE DATABASE db_name [options];
    ```

    For example, the following statement creates a database with the name `samp_db`:

    ```sql
    CREATE DATABASE IF NOT EXISTS samp_db;
    ```

- To show the databases, use the `SHOW DATABASES` statement:

    ```sql
    SHOW DATABASES;
    ```

- To delete a database, use the `DROP DATABASE` statement. For example:

    ```sql
    DROP DATABASE samp_db;
    ```

### Create, show, and drop a table

- To create a table, use the `CREATE TABLE` statement. The Syntax is as follows:

    ```sql
    CREATE TABLE table_name column_name data_type constraint;
    ```

    For example:

    ```sql
    CREATE TABLE person (
     number INT(11),
     name VARCHAR(255),
     birthday DATE
    );
    ```

    Add `IF NOT EXISTS` to prevent an error if the table exists:

    ```sql
    CREATE TABLE IF NOT EXISTS person (
     number INT(11),
     name VARCHAR(255),
     birthday DATE
    );
    ```

- To view the statement that creates the table, use the `SHOW CREATE` statement. For example:

    ```sql
    SHOW CREATE table person;
    ```

- To show all the tables in a database, use the `SHOW TABLES` statement. For example:

    ```sql
    SHOW TABLES FROM samp_db;
    ```

- To show the information about all the columns in a table, use the `SHOW FULL COLUMNS` statement. For example:

    ```sql
    SHOW FULL COLUMNS FROM person;
    ```

- To delete a table, use the `DROP TABLE` statement. For example:

    ```sql
    DROP TABLE person;
    ```

    or

    ```sql
    DROP TABLE IF EXISTS person;
    ```

### Create, show, and drop an index

- To create an index for the column whose value is not unique, use the `CREATE INDEX` or `ALTER TABLE` statement. For example:

    ```sql
    CREATE INDEX person_num ON person (number);
    ```

    or

    ```sql
    ALTER TABLE person ADD INDEX person_num (number);
    ```

- To create a unique index for the column whose value is unique, use the `CREATE UNIQUE INDEX` or `ALTER TABLE` statement. For example:

    ```sql
    CREATE UNIQUE INDEX person_num ON person (number);
    ```

    or

    ```sql
    ALTER TABLE person ADD UNIQUE person_num on (number);
    ```

- To show all the indexes in a table, use the `SHOW INDEX` statement:

    ```sql
    SHOW INDEX from person;
    ```

- To delete an index, use the `DROP INDEX` or `ALTER TABLE` statement. For example:

    ```sql
    DROP INDEX person_num ON person;
    ALTER TABLE person DROP INDEX person_num;
    ```

### Insert, select, update, and delete data

- To insert data into a table, use the `INSERT` statement. For example:

    ```sql
    INSERT INTO person VALUES("1","tom","20170912");
    ```

- To view the data in a table, use the `SELECT` statement. For example:

    ```sql
    SELECT * FROM person;
    +--------+------+------------+
    | number | name | birthday   |
    +--------+------+------------+
    |      1 | tom  | 2017-09-12 |
    +--------+------+------------+
    ```

- To update the data in a table, use the `UPDATE` statement. For example:

    ```sql
    UPDATE person SET birthday='20171010' WHERE name='tom';

    SELECT * FROM person;
    +--------+------+------------+
    | number | name | birthday   |
    +--------+------+------------+
    |      1 | tom  | 2017-10-10 |
    +--------+------+------------+
    ```

- To delete the data in a table, use the `DELETE` statement. For example:

    ```sql
    DELETE FROM person WHERE number=1;
    SELECT * FROM person;
    Empty set (0.00 sec)
    ```

### Create, authorize, and delete a user

- To create a user, use the `CREATE USER` statement. The following example creates a user named `tiuser` with the password `123456`:

    ```sql
    CREATE USER 'tiuser'@'localhost' IDENTIFIED BY '123456';
    ```

- To grant `tiuser` the privilege to retrieve the tables in the `samp_db` database:

    ```sql
    GRANT SELECT ON samp_db.* TO 'tiuser'@'localhost';
    ```

- To check the privileges of `tiuser`:

    ```sql
    SHOW GRANTS for tiuser@localhost;
    ```

- To delete `tiuser`:

    ```sql
    DROP USER 'tiuser'@'localhost';
    ```

## Monitor the TiDB cluster

Open a browser to access the monitoring platform: `http://172.16.10.3:3000`.

The default account and password are: `admin`/`admin`.

### About the key metrics

Service | Panel Name | Description | Normal Range
---- | ---------------- | ---------------------------------- | --------------
PD | Storage Capacity | the total storage capacity of the TiDB cluster |
PD | Current Storage Size | the occupied storage capacity of the TiDB cluster |
PD | Store Status  -- up store | the number of TiKV nodes that are up |
PD | Store Status  -- down store | the number of TiKV nodes that are down | `0`. If the number is bigger than `0`, it means some node(s) are not down.
PD | Store Status  -- offline store | the number of TiKV nodes that are manually offline|
PD | Store Status  -- Tombstone store | the number of TiKV nodes that are Tombstone|
PD | Current storage usage | the storage occupancy rate of the TiKV cluster | If it exceeds 80%, you need to consider adding more TiKV nodes.
PD | 99% completed cmds duration seconds | the 99th percentile duration to complete a pd-server request| less than 5ms
PD | average completed cmds duration seconds | the average duration to complete a pd-server request | less than 50ms
PD | leader balance ratio | the leader ratio difference of the nodes with the biggest leader ratio and the smallest leader ratio | It is less than 5% for a balanced situation. It becomes bigger when a node is restarting.
PD | region balance ratio | the region ratio difference of the nodes with the biggest region ratio and the smallest region ratio | It is less than 5% for a balanced situation. It becomes bigger when adding or removing a node.
TiDB | handle requests duration seconds | the response time to get TSO from PD| less than 100ms
TiDB | tidb server QPS | the QPS of the cluster | application specific
TiDB | connection count | the number of connections from application servers to the database | Application specific. If the number of connections hops, you need to find out the reasons. If it drops to 0, you can check if the network is broken; if it surges, you need to check the application.
TiDB | statement count | the number of different types of statement within a given time | application specific
TiDB | Query Duration 99th percentile | the 99th percentile query time |
TiKV | 99%  & 99.99% scheduler command duration | the 99th percentile and 99.99th percentile scheduler command duration| For 99%, it is less than 50ms; for 99.99%, it is less than 100ms.
TiKV | 95%  & 99.99% storage async_request duration | the 95th percentile and 99.99th percentile Raft command duration | For 95%, it is less than 50ms; for 99.99%, it is less than 100ms.
TiKV | server report failure message | There might be an issue with the network or the message might not come from this cluster. | If there are large amount of messages which contains `unreachable`, there might be an issue with the network. If the message contains `store not match`, the message does not come from this cluster.
TiKV  | Vote |the frequency of the Raft vote | Usually, the value only changes when there is a split. If the value of Vote remains high for a long time, the system might have a severe issue and some nodes are not working.
TiKV | 95% and 99% coprocessor request duration | the 95th percentile and the 99th percentile coprocessor request duration | Application specific. Usually, the value does not remain high.
TiKV | Pending task | the number of pending tasks | Except for PD worker, it is not normal if the value is too high.
TiKV | stall | RocksDB stall time | If the value is bigger than 0, it means that RocksDB is too busy, and you need to pay attention to IO and CPU usage.
TiKV | channel full | The channel is full and the threads are too busy. | If the value is bigger than 0, the threads are too busy.
TiKV |  95% send message duration seconds | the 95th percentile message sending time | less than 50ms
TiKV | leader/region | the number of leader/region per TiKV server| application specific

## Scale the TiDB cluster

The capacity of a TiDB cluster can be increased or decreased without affecting the online services.

> **Warning:** In decreasing the capacity, if your cluster has a mixed deployment of other services, do not perform the following procedures. The following examples assume that the removed nodes have no mixed deployment of other services.

Assume that the topology is as follows:

| Name | Host IP | Services |
| ---- | ------- | -------- |
| node1 | 172.16.10.1 | PD1 |
| node2 | 172.16.10.2 | PD2 |
| node3 | 172.16.10.3 | PD3, Monitor |
| node4 | 172.16.10.4 | TiDB1 |
| node5 | 172.16.10.5 | TiDB2 |
| node6 | 172.16.10.6 | TiKV1 |
| node7 | 172.16.10.7 | TiKV2 |
| node8 | 172.16.10.8 | TiKV3 |
| node9 | 172.16.10.9 | TiKV4 |

### Increase the capacity of a TiDB/TiKV node

For example, if you want to add two TiDB nodes (node101, node102) with the IP address `172.16.10.101` and `172.16.10.102`, you can use the following procedures:

1. Edit the `inventory.ini` file and append the node information:

    ```ini
    [tidb_servers]
    172.16.10.4
    172.16.10.5
    172.16.10.101
    172.16.10.102

    [pd_servers]
    172.16.10.1
    172.16.10.2
    172.16.10.3

    [tikv_servers]
    172.16.10.6
    172.16.10.7
    172.16.10.8
    172.16.10.9

    [monitored_servers]
    172.16.10.1
    172.16.10.2
    172.16.10.3
    172.16.10.4
    172.16.10.5
    172.16.10.6
    172.16.10.7
    172.16.10.8
    172.16.10.9
    172.16.10.101
    172.16.10.102

    [monitoring_servers]
    172.16.10.3

    [grafana_servers]
    172.16.10.3
    ```

    Now the topology is as follows:

    | Name | Host IP | Services |
    | ---- | ------- | -------- |
    | node1 | 172.16.10.1 | PD1 |
    | node2 | 172.16.10.2 | PD2 |
    | node3 | 172.16.10.3 | PD3, Monitor |
    | node4 | 172.16.10.4 | TiDB1 |
    | node5 | 172.16.10.5 | TiDB2 |
    | **node101** | **172.16.10.101**|**TiDB3** |
    | **node102** | **172.16.10.102**|**TiDB4** |
    | node6 | 172.16.10.6 | TiKV1 |
    | node7 | 172.16.10.7 | TiKV2 |
    | node8 | 172.16.10.8 | TiKV3 |
    | node9 | 172.16.10.9 | TiKV4 |
        
2. Initialize the newly added node:

    ```
    ansible-playbook bootstrap.yml -l 172.16.10.101,172.16.10.102
    ```

3. Deploy the newly added node:

    ```
    ansible-playbook deploy.yml -l 172.16.10.101,172.16.10.102
    ```

4. Start the newly added node:

    ```
    ansible-playbook start.yml -l 172.16.10.101,172.16.10.102
    ```

5. Update the Prometheus configuration and restart the cluster:

    ```
    ansible-playbook rolling_update_monitor.yml --tags=prometheus
    ```

6. Monitor the status of the entire cluster and the newly added node by opening a browser to access the monitoring platform: `http://172.16.10.3:3000`.

You can use the same procedure to add a TiKV node. But to add a PD node, some configuration files need to be manually updated.

### Increase the capacity of a PD node

For example, if you want to add a PD node (node103) with the IP address `172.16.10.103`, you can use the following procedures:

1. Edit the `inventory.ini` file and append the node information:

    ```ini
    [tidb_servers]
    172.16.10.4
    172.16.10.5

    [pd_servers]
    172.16.10.1
    172.16.10.2
    172.16.10.3
    172.16.10.103

    [tikv_servers]
    172.16.10.6
    172.16.10.7
    172.16.10.8
    172.16.10.9

    [monitored_servers]
    172.16.10.4
    172.16.10.5
    172.16.10.1
    172.16.10.2
    172.16.10.3
    172.16.10.103
    172.16.10.6
    172.16.10.7
    172.16.10.8
    172.16.10.9

    [monitoring_servers]
    172.16.10.3

    [grafana_servers]
    172.16.10.3
    ```
  
    Now the topology is as follows:
  
    | Name | Host IP | Services |
    | ---- | ------- | -------- |
    | node1 | 172.16.10.1 | PD1 |
    | node2 | 172.16.10.2 | PD2 |
    | node3 | 172.16.10.3 | PD3, Monitor |
    | **node103** | **172.16.10.103** | **PD4** |
    | node4 | 172.16.10.4 | TiDB1 |
    | node5 | 172.16.10.5 | TiDB2 |
    | node6 | 172.16.10.6 | TiKV1 |
    | node7 | 172.16.10.7 | TiKV2 |
    | node8 | 172.16.10.8 | TiKV3 |
    | node9 | 172.16.10.9 | TiKV4 |

2. Initialize the newly added node:

    ```
    ansible-playbook bootstrap.yml -l 172.16.10.103
    ```

3. Deploy the newly added node:

    ```
    ansible-playbook deploy.yml -l 172.16.10.103
    ```

4. Login the newly added PD node and edit the starting script: 
  
    ```
    {deploy_dir}/scripts/run_pd.sh
    ```

    1. Remove the `--initial-cluster="xxxx" \` configuration.
    2. Add `--join="http://172.16.10.1:2379" \`. The IP address (`172.16.10.1`) can be any of the existing PD IP address in the cluster.
    3. Manually start the PD service in the newly added PD node:
      
        ```
        {deploy_dir}/scripts/start_pd.sh
        ```
      
    4. Use `pd-ctl` to check whether the new node is added successfully:
    
        ```
        ./pd-ctl -u "http://172.16.10.1:2379"
        ```
    
        > **Note:** `pd-ctl` is a command used to check the number of PD nodes.

5. Roll upgrade the entire cluster:
    
    ```
    ansible-playbook rolling_update.yml
    ```
   
6. Update the Prometheus configuration and restart the cluster:

    ```
    ansible-playbook rolling_update_monitor.yml --tags=prometheus
    ```

7. Monitor the status of the entire cluster and the newly added node by opening a browser to access the monitoring platform: `http://172.16.10.3:3000`.

### Decrease the capacity of a TiDB node

For example, if you want to remove a TiDB node (node5) with the IP address `172.16.10.5`, you can use the following procedures:

1. Stop all services on node5:

    ```
    ansible-playbook stop.yml -l 172.16.10.5
    ```

2. Edit the `inventory.ini` file and remove the node information:

    ```ini
    [tidb_servers]
    172.16.10.4
    #172.16.10.5  # the removed node

    [pd_servers]
    172.16.10.1
    172.16.10.2
    172.16.10.3

    [tikv_servers]
    172.16.10.6
    172.16.10.7
    172.16.10.8
    172.16.10.9

    [monitored_servers]
    172.16.10.4
    #172.16.10.5  # the removed node
    172.16.10.1
    172.16.10.2
    172.16.10.3
    172.16.10.6
    172.16.10.7
    172.16.10.8
    172.16.10.9

    [monitoring_servers]
    172.16.10.3

    [grafana_servers]
    172.16.10.3
    ```

    Now the topology is as follows:

    | Name | Host IP | Services |
    | ---- | ------- | -------- |
    | node1 | 172.16.10.1 | PD1 |
    | node2 | 172.16.10.2 | PD2 |
    | node3 | 172.16.10.3 | PD3, Monitor |
    | node4 | 172.16.10.4 | TiDB1 |
    | **node5** | **172.16.10.5** | **TiDB2 removed** |
    | node6 | 172.16.10.6 | TiKV1 |
    | node7 | 172.16.10.7 | TiKV2 |
    | node8 | 172.16.10.8 | TiKV3 |
    | node9 | 172.16.10.9 | TiKV4 |

3. Update the Prometheus configuration and restart the cluster:

    ```
    ansible-playbook rolling_update_monitor.yml --tags=prometheus
    ```

4. Monitor the status of the entire cluster by opening a browser to access the monitoring platform: `http://172.16.10.3:3000`.

### Decrease the capacity of a TiKV node

For example, if you want to remove a TiKV node (node9) with the IP address `172.16.10.9`, you can use the following procedures:

1. Remove the node from the cluster using `pd-ctl`:

    1. View the store id of node9:
        
        ```
        ./pd-ctl -u "http://172.16.10.1:2379" -d store
        ```

    2. Remove node9 from the cluster, assuming that the store id is 10:
        
        ```
        ./pd-ctl -u "http://172.16.10.1:2379" -d store delete 10
        ```
        
2. Use Grafana or `pd-ctl` to check whether the node is successfully removed:

    ```
    ./pd-ctl -u "http://172.16.10.1:2379" -d store 10
    ```

    > **Note:** It takes some time to remove the node. If node9 does not show in the result, the node is successfully removed.

3. After the node is successfully removed, stop the services on node9:

    ```
    ansible-playbook stop.yml -l 172.16.10.9
    ```

4. Edit the `inventory.ini` file and remove the node information:

    ```ini
    [tidb_servers]
    172.16.10.4
    172.16.10.5

    [pd_servers]
    172.16.10.1
    172.16.10.2
    172.16.10.3

    [tikv_servers]
    172.16.10.6
    172.16.10.7
    172.16.10.8
    #172.16.10.9  # the removed node

    [monitored_servers]
    172.16.10.4
    172.16.10.5
    172.16.10.1
    172.16.10.2
    172.16.10.3
    172.16.10.6
    172.16.10.7
    172.16.10.8
    #172.16.10.9  # the removed node

    [monitoring_servers]
    172.16.10.3

    [grafana_servers]
    172.16.10.3
    ```

    Now the topology is as follows:

    | Name | Host IP | Services |
    | ---- | ------- | -------- |
    | node1 | 172.16.10.1 | PD1 |
    | node2 | 172.16.10.2 | PD2 |
    | node3 | 172.16.10.3 | PD3, Monitor |
    | node4 | 172.16.10.4 | TiDB1 |
    | node5 | 172.16.10.5 | TiDB2 |
    | node6 | 172.16.10.6 | TiKV1 |
    | node7 | 172.16.10.7 | TiKV2 |
    | node8 | 172.16.10.8 | TiKV3 |
    | **node9** | **172.16.10.9** | **TiKV4 removed** |

5. Update the Prometheus configuration and restart the cluster:

    ```
    ansible-playbook rolling_update_monitor.yml --tags=prometheus
    ```

6. Monitor the status of the entire cluster by opening a browser to access the monitoring platform: `http://172.16.10.3:3000`.

### Decrease the capacity of a PD node

For example, if you want to remove a PD node (node2) with the IP address `172.16.10.2`, you can use the following procedures:

1. Remove the node from the cluster using `pd-ctl`:

    1. View the name of node2:

        ```
        ./pd-ctl -u "http://172.16.10.1:2379" -d member
        ```

    2. Remove node2 from the cluster, assuming that the name is pd2:
        
        ```
        ./pd-ctl -u "http://172.16.10.1:2379" -d member delete name pd2
        ```

2. Use Grafana or `pd-ctl` to check whether the node is successfully removed:

    ```
    ./pd-ctl -u "http://172.16.10.1:2379" -d member
    ```

3. After the node is successfully removed, stop the services on node2:

    ```
    ansible-playbook stop.yml -l 172.16.10.2
    ```

4. Edit the `inventory.ini` file and remove the node information:

    ```ini
    [tidb_servers]
    172.16.10.4
    172.16.10.5

    [pd_servers]
    172.16.10.1
    #172.16.10.2  # the removed node
    172.16.10.3

    [tikv_servers]
    172.16.10.6
    172.16.10.7
    172.16.10.8
    172.16.10.9

    [monitored_servers]
    172.16.10.4
    172.16.10.5
    172.16.10.1
    #172.16.10.2  # the removed node
    172.16.10.3
    172.16.10.6
    172.16.10.7
    172.16.10.8
    172.16.10.9

    [monitoring_servers]
    172.16.10.3

    [grafana_servers]
    172.16.10.3
    ```

    Now the topology is as follows:

    | Name | Host IP | Services |
    | ---- | ------- | -------- |
    | node1 | 172.16.10.1 | PD1 |
    | **node2** | **172.16.10.2** | **PD2 removed** |
    | node3 | 172.16.10.3 | PD3, Monitor |
    | node4 | 172.16.10.4 | TiDB1 |
    | node5 | 172.16.10.5 | TiDB2 |
    | node6 | 172.16.10.6 | TiKV1 |
    | node7 | 172.16.10.7 | TiKV2 |
    | node8 | 172.16.10.8 | TiKV3 |
    | node9 | 172.16.10.9 | TiKV4 |

5. Update the Prometheus configuration and restart the cluster:

    ```
    ansible-playbook rolling_update_monitor.yml --tags=prometheus
    ```

6. Monitor the status of the entire cluster by opening a browser to access the monitoring platform: `http://172.16.10.3:3000`.

## Destroy the TiDB cluster

Stop the cluster:

```
ansible-playbook stop.yml
```

Destroy the cluster:

```
ansible-playbook unsafe_cleanup.yml
```
