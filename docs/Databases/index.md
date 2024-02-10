---
title: Ссылки по базам данных. Материалы для изучения.
share: "true"
---
Инструменты для разработчиков:
- [ReplyByte](https://github.com/Qovery/replibyte) is a blazingly fast tool to seed your databases with your production data while keeping sensitive data safe.
- [Altas](https://atlasgo.io/) — open-source schema migration tool.  `curl -sSf https://atlasgo.sh | sh`
- [Prisma](https://www.prisma.io) — Next-generation ORM for Node.js and TypeScript. Prisma unlocks a new level of **developer experience** when working with databases thanks to its intuitive data model, automated migrations, type-safety & auto-completion.
- [Barman](http://www.pgbarman.org/) - Backup and Recovery Manager for PostgreSQL. Allows your company to implement disaster recovery solutions for PostgreSQL databases with high requirements of business continuity. Taking an online hot backup of PostgreSQL is now as easy as ordering a good espresso coffee.
- [Orchestrator](https://github.com/github/orchestrator) — a MySQL high availability and replication management tool, runs as a service and provides command line access, HTTP API and Web interface.
- [mycli](https://www.mycli.net/) - MyCLI is a command line interface for MySQL, MariaDB, and Percona with auto-completion and syntax highlighting.
- [Percona XtraBackup](https://github.com/percona/percona-xtrabackup) — open-source hot backup utility for MySQL - based servers that doesn’t lock your database during the backup. Percona XtraBackup 8.0 can back up data from InnoDB, XtraDB, MyISAM, and MyRocks tables on MySQL 8.0 servers as well as Percona Server for MySQL with XtraDB, Percona Server for MySQL 8.0, and Percona XtraDB Cluster 8.0. [Документация](https://docs.percona.com/percona-xtrabackup/8.0/index.html). Другой софт для бд от [Percona](https://www.percona.com/mysql/software).
- [Percona Monitoring and Management](https://www.percona.com/software/database-tools/percona-monitoring-and-management) (PMM) is an open source database observability, monitoring, and management tool for MySQL, PostgreSQL, and MongoDB. With PMM, you can spot critical performance issues faster, understand the root cause of incidents better, and troubleshoot them more efficiently.

Программы и веб-интерфейсы для администрирования БД:
- [MySQL Workbench](https://www.mysql.com/products/workbench/) is a unified visual tool for database architects, developers, and DBAs. MySQL Workbench provides data modeling, SQL development, and comprehensive administration tools for server configuration, user administration, backup, and much more. MySQL Workbench is available on Windows, Linux and Mac OS X.
- [Dbeaver Community](https://dbeaver.io/) is a free cross-platform database tool for developers, database administrators, analysts, and everyone working with data. It supports all popular SQL databases like MySQL, MariaDB, PostgreSQL, SQLite, Apache Family, and more.
- [HeidiSQL](https://www.heidisql.com/) — free software, and has the aim to be easy to learn. "Heidi" lets you see and edit data and structures from computers running one of the database systems MariaDB, MySQL, Microsoft SQL, PostgreSQL and SQLite. Invented in 2002 by Ansgar, HeidiSQL belongs to the most popular tools for MariaDB and MySQL worldwide.
- [phpMyAdmin](https://www.phpmyadmin.net/) is a free software tool written in [PHP](https://php.net/), intended to handle the administration of [MySQL](https://www.mysql.com/) over the Web. phpMyAdmin supports a wide range of operations on MySQL and MariaDB. Frequently used operations (managing databases, tables, columns, relations, indexes, users, permissions, etc) can be performed via the user interface, while you still have the ability to directly execute any SQL statement.
- [Adminer](https://www.adminer.org/) - Database management in a single PHP file. Replace **phpMyAdmin** with **Adminer** and you will get a tidier user interface, better support for MySQL features, higher performance and more security.
- [dbKoda](https://www.dbkoda.com/) A modern open source database development and admin tool, now available for MongoDB. **Latest Release:** v1.1.0. It has features to support development, administration and performance tuning on MongoDB databases.
- [SQLPad](https://getsqlpad.com/en/introduction/) is a web app for writing and running SQL queries and visualizing the results. Supports Postgres, MySQL, SQL Server, ClickHouse, Crate, Vertica, Trino, Presto, Pinot, Drill, SAP HANA, BigQuery, SQLite, TiDB and many others via ODBC.

Установка phpMyAdmin и Adminer с плагинами будет рассмотрена в отдельной статье.

**Материалы для изучения:**
- [NOSQL Databases](http://nosql-database.org/) — списки NoSQL баз и материалы по ним
- [Planet MySQL](https://planet.mysql.com/) - статьи по MySQL
- [Planet PostgreSQL](https://planet.postgresql.org/) - статьи по PostgreSQL
- [Database Performance Blog](https://www.percona.com/blog/) — блог по бд компании Percona
- [Awesome Postgres](https://github.com/dhamaniasad/awesome-postgres) — a curated list of awesome [PostgreSQL](https://www.postgresql.org/) software, libraries, tools and resources, inspired by [awesome-mysql](http://shlomi-noach.github.io/awesome-mysql/)
- [Awesome Mysql](https://github.com/shlomi-noach/awesome-mysql) - A curated list of awesome MySQL software, libraries, tools and resources
- [MySQL Tutorial](http://www.mysqltutorial.org/) - Learn MySQL Fast, Easy and Fun.
- [Free MongoDB Course with Python](http://freemongodbcourse.com/) — курс

**YouTube каналы**
- [Теория БД](https://www.youtube.com/live/zWtJoWGHsiI?si=P47gGj_WaIu9hCx6)от [Дмитрия Елисеева](https://elisdn.ru/). Немного о теории реляционных баз данных. На форуме и в личных сообщениях часто спрашивают о проектировании базы данных и о работе со связями в ActiveRecord во фреймворке. Про сохранение связанных моделей, про их вывод, про сортировку и поиск. Но и при этом многие не знают, откуда эти связи берутся, как организовываются и как за ними нужно следить. Поэтому отдельно поговорили о теории баз данных с практическим уклоном на нормализацию и внешние ключи.
- [Percona Database Performance](https://www.youtube.com/user/PerconaMySQL) - contains technical videos, webinars and screencasts about everything related to MySQL and MongoDB
- [Postgres Open](https://www.youtube.com/channel/UCCDA5Yte0itW_Bf6UHpbHug) — is a non-profit, community-run conference series in the United States focused on business users, database professionals and developers of PostgreSQL, the open source database.
- [PGCon](https://www.youtube.com/channel/UCer4R0y7DrLsOXo-bI71O6A) — PostgreSQL Conference for Users and Developers
- [MongoDB](https://www.youtube.com/user/MongoDB/videos)-


**RDBMS (Relational DBMS, Реляционные базы данных)**:
- [Firebird](http://www.firebirdsql.org/) - True universal database.
- [Galera](http://galeracluster.com/) - Galera Cluster for MySQL is an easy-to-use high-availability solution with high system up-time, no data loss, and scalability for future growth.
- [MariaDB](https://mariadb.org/) - Community-developed fork of the MySQL.
- [Percona Server](https://www.percona.com/software) - Enhanced, drop-in MySQL replacement.
- [PostgreSQL](http://www.postgresql.org/) - Object-relational database management system (ORDBMS).
- [PostgreSQL-XL](http://www.postgres-xl.org/) - Scalable PostgreSQL-based database cluster.
- [SQLite](http://sqlite.org/) - Library that implements a self-contained, serverless, zero-configuration, transactional SQL DBS.

Курсы:
- [MySQL 8 for Administrators](https://www.packtpub.com/product/mysql-8-for-administrators-video/9781788398329)
- [CBT Nuggets - Database Fundamentals](https://www.cbtnuggets.com/it-training/skills/database-fundamentals)
- [Introduction to MongoDB](https://www.coursera.org/learn/introduction-to-mongodb#modules)
- O'Reilly - [Berglund and McCullough on Mastering Cassandra for Architects](https://www.careervira.com/course/berglund-and-mccullough-on-mastering-cassandra-for-architects)
- [The Ultimate MySQL Bootcamp: Go from SQL Beginner to Expert](https://www.udemy.com/course/the-ultimate-mysql-bootcamp-go-from-sql-beginner-to-expert/)
- Udemy - [The Complete SQL Bootcamp](https://www.udemy.com/course/the-complete-sql-bootcamp/)