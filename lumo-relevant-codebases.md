<!-- SPDX-License-Identifier: CC-BY-SA-4.0 -->
<!-- SPDX-FileCopyrightText: 2020 The LumoSQL Authors -->
<!-- SPDX-ArtifactOfProjectName: LumoSQL -->
<!-- SPDX-FileType: Documentation -->
<!-- SPDX-FileComment: Original by Dan Shearer, 2019 -->


Table of Contents
=================

   * [Codebases Relevant to LumoSQL](#codebases-relevant-to-lumosql)
   * [What is a Relevant Codebase?](#what-is-a-relevant-codebase)
   * [Useful Dead SQLite Forks](#useful-dead-sqlite-forks)
   * [Oracle BDB and Oracle BDB-SQL Codebases](#oracle-bdb-and-oracle-bdb-sql-codebases)
   * [Distributed or Clustered Codebases](#distributed-or-clustered-codebases)
   * [Code to Potentially Include in LumoSQL](#code-to-potentially-include-in-lumosql)
   * [List of On-disk File Format-related Knowledge](#list-of-on-disk-file-format-related-knowledge)
   * [List of Relevant Benchmarking and Test Knowledge](#list-of-relevant-benchmarking-and-test-knowledge)
   * [List of Just a Few SQLite Encryption Projects](#list-of-just-a-few-sqlite-encryption-projects)
   * [List of from-scratch MySQL SQL and MySQL Server implementations](#list-of-from-scratch-mysql-sql-and-mysql-server-implementations)

Codebases Relevant to LumoSQL
=============================

![Ingredients](./images/lumo-relevant-codebases-intro.jpg "FuQin Market in Chengdu, CC license, https://chopsticksinmysuitcase.typepad.com/.a/6a0133f251ecbe970b0133f60d5807970b-500wi")


What is a Relevant Codebase?
============================

There are three dimensions to codebases relevant to LumoSQL:

1. Code that is a derivative of SQLite code adding a feature or improvement, and
2. Code that has nothing to do with SQLite but implements an interesting database feature we want to use in LumoSQL
3. Code that supports the development of LumoSQL such as testing, benchmarking or analysing relevant codebases

This is a discussion document, describing our findings as we look at hundreds
of thousands of lines of other people's code. In addition there is the [Full Knowledgebase Relevant to LumoSQL](./lumo-relevant-knowledgebase.md) .
There is a lot more in the Knowledgebase than there is in this more detailed document.

As to what we are implementing in LumoSQL, that is in [LumoSQL Implementation](./lumo-implementation.md).

# Useful Dead SQLite Forks

Code becomes unmaintained for many reasons, and some interesting code is definitely dead. We have considered the following codebases.

| Project | Last modified | Description   |
| ------------- | ------------- | --------|
| [sqlightning](https://github.com/LMDB/sqlightning)  | 2013 | SQLight ported to the LMDB key-value store |
| [SQLHeavy](https://github.com/btrask/sqlheavy)  | 2016 | SQLite ported to LevelDB, LMDB, RocksDB and more, with a key-value store library abstraction |
| [libkvstore](https://github.com/btrask/libkvstore) | 2016 | The backend library used by SQLHeavy |
| [SQLite 4](https://sqlite.org/src4/tree?ci=trunk) | 2014 | Abandoned new version of SQLite with improved backend support and other features |

The library ```libkvstore``` is after the style of what we are implementing at the API interception point in ```backend.c```, and the author remains active in this general area.

SQLHeavy never got beyond prototype stage but it did support multiple K-V stores. Despite looking at a great deal of code we never noticed SQLHeavy until we had already done our own work to revive sqlightning. This is a reminder that we may still be missing important existing contributions.

The defunct version 4 of SQLite intended to implement [pluggable storage engines](https://sqlite.org/src4/doc/trunk/www/storage.wiki), to similar effect
to LumoSQL. The intention was that "Transaction commit and rollback is handled
by the storage engine", meaning that SQLite4 was to be quite conservative technically, not using 
MVCC-compliant K-V stores, and still using Write-Ahead Logs.

# Oracle BDB and Oracle BDB-SQL Codebases

As of June 2020, Oracle announced that it has dropped support for the BDB port
to SQLite. DB 18.1.32 is the last version to carry this, which is based on
SQLite from 2017. This is the reference and basis for the BDB backend in
LumoSQL.

| Project | Last modified | Description   |
| ------------- | ------------- | --------|
| [Sleepycat/Oracle BDB](https://fossies.org/linux/misc/db-18.1.32.tar.gz) | current | The original ubiquitous Unix K-V store, disused in open source since Oracle's 2013 license change; the API template for most of the k-v btree stores around. Now includes many additional features including full MVCC transactions, networking and replication. This link is a mirror of code from download.oracle.com, which requires a login | 
| [Sleepycat/Oracle BDB-SQL](https://fossies.org/linux/misc/db-18.1.32.tar.gz) | current | Port of SQLite to the Sleepycat/Oracle transactional bdb K-V store. As of 5th March 2020 this mirror is identical to Oracle's login-protected tarball for db 18.1.32 | 

# Distributed or Clustered Codebases

The following four projects are widely-varying examples of how SQLite data can
be distributed, whether across just a few local nodes or across a much higher
number of internet-connected nodes.

| Project | Last modified | Description   |
| ------------- | ------------- | --------|
| [rqlite](https://github.com/rqlite/rqlite) | current | Distributed database with networking and Raft consensus on top of SQLite nodes |
| [Bedrock](https://github.com/Expensify/Bedrock) | current | WAN-replicated blockchain multimaster database built on SQLite. Has MySQL emulation |
| [ActorDB](https://github.com/biokoda/actordb) | current | SQLite with a data sharding/distribution system across clustered nodes. Each node stores data in LMDB, which is connected to SQLite at the SQLite WAL layer |
| [WAL-G](https://github.com/wal-g/wal-g) | current | Backup/replication tool that intercepts the WAL journal log for each of Postgres, Mysql, MonogoDB and Redis |

Unlike all other solutions we have found, rqlite builds its replication on top of SQLite nodes rather than underneath the SQLite storage API layer.

ActorDB uses LMDB but still has a WAL.

WAL-G illustrates a useful side-effect of having a WAL, in that it can be used as a list of transactions for archival as well as replay reasons. A non-WAL storage databases such as LMDB can also generate transaction logs for these sorts of purpose, but they aren't for replay reasons. 

Oracle BDB-SQL discussed in the previous section also has replication. 

| [sql.js](https://github.com/kripken/sql.js/) | current | SQLite compiled to JavaScript WebAssembly through Emscripten |

# Code to Potentially Include in LumoSQL

| Project | Last modified | Description   |
| ------------- | ------------- | --------|
| [sqlite3odbc](https://github.com/gdev2018/sqlite3odbc) | current | ODBC driver for SQLite by [Christian Werner](http://www.ch-werner.de/sqliteodbc/) as used by many projects including LibreOffice |

SQLite3ODBC is a wrapper around the whole of the SQLite library. None of the
LumoSQL API interception points can be used for this, nevertheless, ODBC is an
important cross-platform standard heavily used on Windows and IBM's operating
systems.

| [Spatialite](https://www.gaia-gis.it/fossil/libspatialite/index)| current | Geospatial GIS extension to SQLite, similar to PostGIS |

GIS features are a vertical use case, but one that is very popular and
widely-used. PostGIS would very likely have been merged into PostgreSQL long
ago, except for a difference in licensing. Spatialite is offered under the MPL
license (among others) and so does not pose any such problem being included
with LumoSQL. LumoSQL needs to consider packaging Spatialite in the early
stages of LumoSQL. 

| [Gigimushroom's Database Backend Engine](https://github.com/gigimushroom/DatabaseBackendEngine)|2019| A good example of an alternative BTree storage engine implemented using SQLite's Virtual Table Interface. This approach is not what LumoSQL has chosen for many reasons, but this code demonstrates virtual tables can work, and also that storage engines implemented at virtual tables can be ported to be LumoSQL backends.|

# On-disk File Format-related Knowledge

The on-disk file format is important to many SQLite use cases, and introspection tools are both important and rare. Other K-V stores also have third-party on-disk introspection tools. There are advantages to having investigative tools that do not use the original/canonical source code to read and write these databases. The SQLite file format is promoted as being a stable, backwards-compatible transport (recommend by the Library of Congress as an archive format) but it also has significant drawbacks as discussed elsewhere in the LumoSQL documentation.

| Project | Last modified | Description |
| ------- | ------------- | ----------- |
| [A standardized corpus for SQLite database forensics](https://www.sciencedirect.com/science/article/pii/S1742287618300471) | current | Sample SQLite databases and evaluations of 5 tools that do extraction and recovery from SQLite, including Undark and SQLite Deleted Records Parser |
| [FastoNoSQL](https://github.com/fastogt/fastonosql) | current | GUI inspector and management tool for on-disk databases including LMDB and LevelDB |
| [Undark](https://github.com/inflex/undark) | 2016 | SQLite deleted and corrupted data recovery tool |
| [SQLite Deleted Records Parser](https://github.com/mdegrazia/SQLite-Deleted-Records-Parser) | 2015 | Script to recover deleted entries in an SQLite database |
| [lua-mdb](https://github.com/catwell/cw-lua/tree/master/lua-mdb) | 2016 | Parse and investigate LMDB file format |

# List of Relevant Benchmarking and Test Knowledge

Benchmarking is a big part of LumoSQL, to determine if changes are an improvement. The trouble is that SQLite and other top databases are not really benchmarked in realistic and consistent way, despite SQL server benchmarking using tools like TPC being an obsessive industry in itself, and there being myriad of testing tools released with SQLite, Postgresql, MariaDB etc. But in practical terms there is no way of comparing the most-used databases with each other, or even of being sure that the tests that do exist are in any way realistic, or even of simply reproducing results that other people have found. LumoSQL covers so many codebases and use cases that better SQL benchmarking is a project requirement. Benchmarking and testing overlap, which is addressed in the code and docs.

The well-described [testing of SQLite](https://sqlite.org/testing.html) involves some open code, some closed code, and many ad hoc processes. Clearly the SQLite team have an internal culture of testing that has benefitted the world. However that is very different to reproducible testing, which is in turn very different to reproducible benchmarking, and that is even without considering whether the benchmarking is a reasonable approximation of actual use cases.

To highlight how poorly SQL benchmarking is done: there are virtually no test harnesses that cover encrypted databases and/or encrypted database connections, despite encryption being frequently required, and despite crypto implementation decisions making a very big difference in performance.

| Project | Last modified | Description | 
| ------- | ------------- | ----------- |
| [Dangers and complexity of sqlite3 benchmarking](https://www.cs.utexas.edu/~vijay/papers/apsys17-sqlite.pdf)| n/a | Helpful 2017 paper: "...changing just one parameter in SQLite can change the performance by 11.8X... up to 28X difference in performance" |
| [sqllogictest](https://www.sqlite.org/sqllogictest/doc/trunk/about.wiki)|2017 | [sqlite.org code](https://www.sqlite.org/sqllogictest/artifact/2c354f3d44da6356) to [compare the results](https://gerardnico.com/data/type/relation/sql/test) of many SQL statements between multiple SQL servers, either SQLite or an ODBC-supporting server |
| [TCL SQLite tests](https://github.com/sqlite/sqlite/tree/master/test)|current| These are a mixture of code covereage tests, unit tests and test coverage. Actively maintained. |
| [Yahoo Cloud Serving Benchmark](https://github.com/brianfrankcooper/YCSB/)| current | Benchmarking tool for K-V stores and cloud-accessible databases |
| [Example Android Storage Benchmark](https://github.com/greenrobot/android-database-performance) | 2018 | This code is an example of the very many Android benchmarking/testing tools. This needs further investigation |
| [Sysbench](https://github.com/akopytov/sysbench) | current | A multithreaded generic benchmarking tool, with one well-supported use case being networked SQL servers, and [MySQL in particular](https://www.percona.com/blog/2019/04/25/creating-custom-sysbench-scripts/) |


# List of Just a Few SQLite Encryption Projects

Encryption is a major problem for SQLite users looking for open code. There are no official implementations in open source, although the APIs are documented (seemingly by an SCM mistake years ago (?), see sqlite3-dbx below) and most solutions use the SQLite extension interaface. This means that there are many mutually-incompatible implementations, several of them seeming to be very popular. None appear to have received encryption certification (?) and none seem to publish test results to reassure users about compatibility with SQLite upstream or with the file format. Besides the closed source solution from sqlite.org, there are also at least three other closed source options not listed here. This choice between either closed source or fragmented solutions is a poor security approach from the point of view of maintainance as well as peer-reviewed security. This means that SQLite in 2020 does not have a good approach to privacy.

| Project | Last modified | Description | 
| ------- | ------------- | ----------- |
| [SQLite Encryption Extension](https://www.sqlite.org/see/doc/release/www/readme.wiki)| current | Info about the (closed source) official SQLite crypto solution, illustrating that there is little to be compatible with in the wider SQLite landscape |
| [SQLCipher](https://github.com/sqlcipher/sqlcipher) | current | Adds at-rest encryption to SQLite [at the pager level](https://www.zetetic.net/sqlcipher/design/), using OpenSSL (the default) or optionally other providers |
| [sqleet](https://github.com/resilar/sqleet) | current | Implements SHA256 encryption, also at the pager level |
| [sqlite3-dbx](https://github.com/newsoft/sqlite3-dbx) | kinda-current | Interesting documentation that perhaps sqlite.org never meant to publish their crypto APIs? |
| [SQLite3-Encryption](https://github.com/darkman66/SQLite3-Encryption) | current | No crypto libraries (DIY crypto!) and based on the similar-sounding SQLite3-with-Encryption project | 

... there are many more crypto projects for SQLite. 

# List of from-scratch MySQL SQL and MySQL Server implementations

If we want to make SQLite able to process MySQL queries there is a lot of existing code in this area to consider. There are at least 80 projects on github which implement some or all of the MySQL network-parse-optimise-execute SQL pathway, a few of them implement all of it. None so far reviewed used MySQL or MariaDB code to do so. Perhaps that is because the SQL processing code alone in these databases is many times bigger than the whole of SQLite, and it isn't even clear how to add them to this table if we wanted to. Only a few of these projects put a MySQL frontend on SQLite, but two well-maintained projects do, showing us two ways of implementing this.

| Project | Last modified | Description |
| ------- | ------------- | ----------- |
| [Bedrock](https://github.com/Expensify/Bedrock) | current | The MySQL compatibility seems to be popular and is actively supported but it is also small. It speaks the MySQL/MariaDB protocol accurately but doesn't seem to try very hard to match MySQL SQL language semantics and extensions, rather relying on the fact that SQLite substantially overlaps with MySQL. |
| [TiDB](https://github.com/pingcap/tidb/) | current | Distributed database with MySQL emulation as the primary dialect and referred to throughout the code, with frequent detailed bugfixes on deviations from MySQL SQL language behaviour. |
| [phpMyAdmin parser](https://github.com/phpmyadmin/sql-parser) | current | A very complete parser for MySQL code, demonstrating that completeness is not the unrealistic goal some claim it to be |
| [Go MySQL Server](https://github.com/src-d/go-mysql-server) | current | A MySQL server written in Go that executes queries but mostly leaves the backend for the user to implement. Intended to put a compliant MySQL server on top of arbitary backend sources. |
| [ClickHouse MySQL Frontend](https://github.com/ClickHouse/ClickHouse/tree/146109fe27074229a38cd704d60f23ec7bd2ed67/base/mysqlxx) | current | Yandex' [Clickhouse](https://clickhouse.tech/) has a MySQL frontend.|
