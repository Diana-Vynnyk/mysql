# mysql

[Screenshots](https://drive.google.com/drive/folders/1rX1SBjM27tKgDkpuPw5f4WRPelr-FUA3?usp=sharing)

[working_services](https://drive.google.com/file/d/140bu5sMIMJvNQW4qF0aX_eb_LjgEs7Yz/view?usp=sharing)

[replica_status](https://drive.google.com/file/d/1lGHfdJwF_RBVL7SfRydbxf8JPdNMMs-Z/view?usp=sharing)

[db_example](https://drive.google.com/file/d/10dvFjoPSyqaVMnEBe_fsXUWFD3RodOIa/view?usp=sharing)

MySQL is an open-source database management system, commonly installed as part of the popular LAMP (Linux, Apache, MySQL, PHP/Python/Perl) stack. It implements the relational model and uses Structured Query Language (better known as SQL) to manage its data.

When working with databases, it can be useful to have multiple copies of your data. This provides redundancy in case one of the database servers fails and can improve a database’s availability, scalability, and overall performance. The practice of synchronizing data across multiple separate databases is called replication.

MySQL is a relational database management system, and is the most popular open-source relational database in the world today. It comes installed with a number of built-in replication features, allowing you to maintain multiple copies of your data.

# Understanding Replication in MySQL

In MySQL, replication involves the source database writing down every change made to the data held within one or more databases in a special file known as the binary log. Once the replica instance has been initialized, it creates two threaded processes. The first, called the IO thread, connects to the source MySQL instance and reads the binary log events line by line, and then copies them over to a local file on the replica’s server called the relay log. The second thread, called the SQL thread, reads events from the relay log and then applies them to the replica instance as fast as possible.

Recent versions of MySQL support two methods for replicating data. The difference between these replication methods has to do with how replicas track which database events from the source they’ve already processed.

MySQL refers to its traditional replication method as binary log file position-based replication. When you turn a MySQL instance into a replica using this method, you must provide it with a set of binary log coordinates. These consist of the name of the binary log file on the source which the replica must read and a specific position within that file which represents the first database event the replica should copy to its own MySQL instance.

These coordinates are important since replicas receive a copy of their source’s entire binary log and, without the right coordinates, they will begin replicating every database event recorded within it. This can lead to problems if you only want to replicate data after a certain point in time or only want to replicate a subset of the source’s data.

Binary log file position-based replication is viable for many use cases, but this method can become clunky in more complex setups. This led to the development of MySQL’s newer native replication method, which is sometimes referred to as transaction-based replication. This method involves creating a global transaction identifier (GTID) for each transaction — or, an isolated piece of work performed by a database — that the source MySQL instance executes.

The mechanics of transaction-based replication are similar to binary log file-based replication: whenever a database transaction occurs on the source, MySQL assigns and records a GTID for the transaction in the binary log file along with the transaction itself. The GTID and the transaction are then transmitted to the source’s replicas for them to process.

MySQL’s transaction-based replication has a number of benefits over its traditional replication method. For example, because both a source and its replicas preserve GTIDs, if either the source or a replica encounter a transaction with a GTID that they have processed before they will skip that transaction. This helps to ensure consistency between the source and its replicas. Additionally, with transaction-based replication replicas don’t need to know the binary log coordinates of the next database event to process. This means that starting new replicas or changing the order of replicas in a replication chain is far less complicated.

Keep in mind that this is only a general explanation of how MySQL handles replication; MySQL provides many options which you can tweak to optimize your own replication setup. This guide outlines how to set up binary log file position-based replication. If you’re interested in configuring a different type of replication environment, though, we encourage you to check out MySQL’s official documentation .

Replication enables data from one MySQL database server (known as a source) to be copied to one or more MySQL database servers (known as replicas). Replication is asynchronous by default; replicas do not need to be connected permanently to receive updates from a source. Depending on the configuration, you can replicate all databases, selected databases, or even selected tables within a database.

Advantages of replication in MySQL include:

- Scale-out solutions - spreading the load among multiple replicas to improve performance. In this environment, all writes and updates must take place on the source server. Reads, however, may take place on one or more replicas. This model can improve the performance of writes (since the source is dedicated to updates), while dramatically increasing read speed across an increasing number of replicas.

- Data security - because the replica can pause the replication process, it is possible to run backup services on the replica without corrupting the corresponding source data.

- Analytics - live data can be created on the source, while the analysis of the information can take place on the replica without affecting the performance of the source.

- Long-distance data distribution - you can use replication to create a local copy of data for a remote site to use, without permanent access to the source.

MySQL 8.0 supports different methods of replication. The traditional method is based on replicating events from the source's binary log, and requires the log files and positions in them to be synchronized between source and replica. The newer method based on global transaction identifiers (GTIDs) is transactional and therefore does not require working with log files or positions within these files, which greatly simplifies many common replication tasks. Replication using GTIDs guarantees consistency between source and replica as long as all transactions committed on the source have also been applied on the replica. For more information about GTIDs and GTID-based replication in MySQL, see Section 17.1.3, “Replication with Global Transaction Identifiers”. For information on using binary log file position based replication, see Section 17.1, “Configuring Replication”.

Replication in MySQL supports different types of synchronization. The original type of synchronization is one-way, asynchronous replication, in which one server acts as the source, while one or more other servers act as replicas. This is in contrast to the synchronous replication which is a characteristic of NDB Cluster (see Chapter 23, MySQL NDB Cluster 8.0). In MySQL 8.0, semisynchronous replication is supported in addition to the built-in asynchronous replication. With semisynchronous replication, a commit performed on the source blocks before returning to the session that performed the transaction until at least one replica acknowledges that it has received and logged the events for the transaction; see Section 17.4.10, “Semisynchronous Replication”. MySQL 8.0 also supports delayed replication such that a replica deliberately lags behind the source by at least a specified amount of time; see Section 17.4.11, “Delayed Replication”. For scenarios where synchronous replication is required, use NDB Cluster (see Chapter 23, MySQL NDB Cluster 8.0).

There are a number of solutions available for setting up replication between servers, and the best method to use depends on the presence of data and the engine types you are using. For more information on the available options, see Section 17.1.2, “Setting Up Binary Log File Position Based Replication”.

There are two core types of replication format, Statement Based Replication (SBR), which replicates entire SQL statements, and Row Based Replication (RBR), which replicates only the changed rows. You can also use a third variety, Mixed Based Replication (MBR). For more information on the different replication formats, see Section 17.2.1, “Replication Formats”.

Replication is controlled through a number of different options and variables. For more information, see Section 17.1.6, “Replication and Binary Logging Options and Variables”. Additional security measures can be applied to a replication topology, as described in Section 17.3, “Replication Security”.

You can use replication to solve a number of different problems, including performance, supporting the backup of different databases, and as part of a larger solution to alleviate system failures. For information on how to address these issues, see Section 17.4, “Replication Solutions”.

# Switching Sources During Failover

You can tell a replica to change to a new source using the CHANGE REPLICATION SOURCE TO statement (prior to MySQL 8.0.23: CHANGE MASTER TO. The replica does not check whether the databases on the source are compatible with those on the replica; it simply begins reading and executing events from the specified coordinates in the new source's binary log. In a failover situation, all the servers in the group are typically executing the same events from the same binary log file, so changing the source of the events should not affect the structure or integrity of the database, provided that you exercise care in making the change.

Replicas should be run with binary logging enabled (the --log-bin option), which is the default. If you are not using GTIDs for replication, then the replicas should also be run with --log-slave-updates=OFF (logging replica updates is the default). In this way, the replica is ready to become a source without restarting the replica mysqld. Assume that you have the structure shown in Figure 3.4, “Redundancy Using Replication, Initial Structure”.

![Redundancy Using Replication, Initial Structure](https://dev.mysql.com/doc/mysql-replication-excerpt/8.0/en/images/redundancy-before.png)

In this diagram, the Source holds the source database, the Replica* hosts are replicas, and the Web Client machines are issuing database reads and writes. Web clients that issue only reads (and would normally be connected to the replicas) are not shown, as they do not need to switch to a new server in the event of failure. For a more detailed example of a read/write scale-out replication structure, see Section 3.5, “Using Replication for Scale-Out”.

Each MySQL replica (Replica 1, Replica 2, and Replica 3) is a replica running with binary logging enabled, and with --log-slave-updates=OFF. Because updates received by a replica from the source are not written to the binary log when --log-slave-updates=OFF is specified, the binary log on each replica is initially empty. If for some reason Source becomes unavailable, you can pick one of the replicas to become the new source. For example, if you pick Replica 1, all Web Clients should be redirected to Replica 1, which writes the updates to its binary log. Replica 2 and Replica 3 should then replicate from Replica 1.

The reason for running the replica with --log-slave-updates=OFF is to prevent replicas from receiving updates twice in case you cause one of the replicas to become the new source. If Replica 1 has --log-slave-updates enabled, which is the default, it writes any updates that it receives from Source in its own binary log. This means that, when Replica 2 changes from Source to Replica 1 as its source, it may receive updates from Replica 1 that it has already received from Source.

Make sure that all replicas have processed any statements in their relay log. On each replica, issue STOP REPLICA IO_THREAD, then check the output of SHOW PROCESSLIST until you see Has read all relay log. When this is true for all replicas, they can be reconfigured to the new setup. On the replica Replica 1 being promoted to become the source, issue STOP REPLICA and RESET MASTER.

On the other replicas Replica 2 and Replica 3, use STOP REPLICA and CHANGE REPLICATION SOURCE TO SOURCE_HOST='Replica1' or CHANGE MASTER TO MASTER_HOST='Replica1' (where 'Replica1' represents the real host name of Replica 1). To use CHANGE REPLICATION SOURCE TO, add all information about how to connect to Replica 1 from Replica 2 or Replica 3 (user, password, port). When issuing the statement in this scenario, there is no need to specify the name of the Replica 1 binary log file or log position to read from, since the first binary log file and position 4 are the defaults. Finally, execute START REPLICA on Replica 2 and Replica 3.

Once the new replication setup is in place, you need to tell each Web Client to direct its statements to Replica 1. From that point on, all updates sent by Web Client to Replica 1 are written to the binary log of Replica 1, which then contains every update sent to Replica 1 since Source became unavailable.

The resulting server structure is shown in Figure 3.5, “Redundancy Using Replication, After Source Failure”.

![Figure 3.5, “Redundancy Using Replication, After Source Failure”.](https://dev.mysql.com/doc/mysql-replication-excerpt/8.0/en/images/redundancy-after.png)

When Source becomes available again, you should make it a replica of Replica 1. To do this, issue on Source the same CHANGE REPLICATION SOURCE TO (or CHANGE MASTER TO) statement as that issued on Replica 2 and Replica 3 previously. Source then becomes a replica of Replica 1 and picks up the Web Client writes that it missed while it was offline.

To make Source a source again, use the preceding procedure as if Replica 1 were unavailable and Source were to be the new source. During this procedure, do not forget to run RESET MASTER on Source before making Replica 1, Replica 2, and Replica 3 replicas of Source. If you fail to do this, the replicas may pick up stale writes from the Web Client applications dating from before the point at which Source became unavailable.

You should be aware that there is no synchronization between replicas, even when they share the same source, and thus some replicas might be considerably ahead of others. This means that in some cases the procedure outlined in the previous example might not work as expected. In practice, however, relay logs on all replicas should be relatively close together.

One way to keep applications informed about the location of the source is to have a dynamic DNS entry for the source host. With BIND, you can use nsupdate to update the DNS dynamically.


# Sources

[How To Install MySQL on Ubuntu 22.04](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-22-04)

[How To Set Up Replication in MySQL](https://www.digitalocean.com/community/tutorials/how-to-set-up-replication-in-mysql)

[Replication](https://dev.mysql.com/doc/refman/8.0/en/replication.html)

[Switching Sources During Failover](https://dev.mysql.com/doc/mysql-replication-excerpt/8.0/en/replication-solutions-switch.html)
