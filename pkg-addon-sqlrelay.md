# sqlrelay

### Description:
영구적인 데이터베이스 연결 시스템

SQL Relay is a persistent database connection pooling, proxying, throttling,
load balancing and query routing/filtering system for Unix and Linux
supporting ODBC, Oracle, MySQL, PostgreSQL, SAP/Sybase, MS SQL Server, IBM
DB2, Informix, Firebird, SQLite and MS Access (minimally) with APIs for C,
C++, .NET, Perl, Perl-DBI, Python, Python-DB, PHP, PHP PDO, Ruby, Java, TCL,
Erlang, and node.js, ODBC and ADO.NET drivers, drop-in replacement libraries
for MySQL and PostgreSQL, command line clients and extensive documentation.
The APIs support advanced database operations such as bind variables, multi-row
fetches, client-side result set caching and suspended transactions.  It is
ideal for speeding up database-driven web-based applications, accessing
databases from unsupported platforms, migrating between databases, distributing
access to replicated or clustered databases and throttling database access.

### Features:


### Reference:
* http://sqlrelay.sourceforge.net/

### Dependencies:
* [rudiments](pkg-addon-rudiments.md)

### Sub Packages:
* **sqlrelay-client-devel-c** - Development files for developing programs C that use SQL Relay
* **sqlrelay-client-devel-c++** - Development files for developing programs in C++ that use SQL Relay
* **sqlrelay-client-mysql** - Drop in replacement library allowing MySQL clients to use SQL Relay instead
* **sqlrelay-client-odbc** - ODBC driver
* **sqlrelay-client-postgresql** - Drop in replacement library allowing PostgreSQL clients to use SQL Relay instead
* **sqlrelay-client-runtime-c** - Runtime libraries for SQL Relay clients written in C
* **sqlrelay-client-runtime-c++** - Runtime libraries for SQL Relay clients written in C++
* **sqlrelay-clients** - Command line applications for accessing databases through SQL Relay
* **sqlrelay-doc** - Documentation for SQL Relay
* **sqlrelay-freetds** - SQL Relay connection plugin for FreeTDS (SAP/Sybase and MS SQL Server)
* **sqlrelay-man** - Man pages for SQL Relay
* **sqlrelay-mysql** - SQL Relay connection plugin for MySQL
* **sqlrelay-odbc** - SQL Relay connection plugin for ODBC
* **sqlrelay-oracle** - SQL Relay connection plugin for Oracle
* **sqlrelay-postgresql** - SQL Relay connection plugin for PostgreSQL
* **sqlrelay-router** - SQL Relay query routing daemon
* **sqlrelay-server-devel** - Development files for developing modules for the SQL Relay server
* **sqlrelay-sqlite** - SQL Relay connection plugin for SQLite
* **java-sqlrelay** - SQL Relay module for Java
* **php-sqlreiay** - SQL Relay module for PHP
* **python-sqlrelay** - SQL Relay module for Python
* **ruby-sqlrelay** - SQL Relay module for Ruby

### Related Packages:
* None