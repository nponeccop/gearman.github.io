This file is maintained in the Launchpad tree, so any modifications should be made there.
<html><pre>
The Gearman MySQL user defined functions (UDFs) allow queries in MySQL
to act as a Gearman client. This gives you the ability to run functions
in you query not just on the local machine, but to any machine running
a Gearman worker. This can be useful for a number of reasons, including
processor or memory intensive functions that you wish to run on other
machines, functions that can run in the background, functions that
can run in parallel for aggregate operations, and also to trigger
jobs or other actions from a MySQL query. See the Gearman website
for information at:

http://www.gearman.org/

The MySQL UDFs require the Gearman C server and library package to be
installed. You can find more information about how to do this through
the website above. Once this is installed, you can then compile the
Gearman MySQL UDF package. This can usually be done with the normal:

To build:

./configure --with-mysql=[path to mysql_config] --libdir=[path to mysql plugin]

For example:

./configure --with-mysql=/usr/local/mysql/bin/mysql_config --libdir=/usr/local/mysql/lib/plugin/

make
make install

Please keep in mind that for your UDF to be loaded, it must be in
the library path for your server. On most UNIX-like systems you can
set this by exporting the correct path in LD_LIBRARY_PATH for your
mysql server.

Once the UDFs have been compiled and installed, you load them into
MySQL using the following queries:

CREATE FUNCTION gman_do RETURNS STRING
       SONAME "libgearman_mysql_udf.so";
CREATE FUNCTION gman_do_high RETURNS STRING
       SONAME "libgearman_mysql_udf.so";
CREATE FUNCTION gman_do_low RETURNS STRING
       SONAME "libgearman_mysql_udf.so";
CREATE FUNCTION gman_do_background RETURNS STRING
       SONAME "libgearman_mysql_udf.so";
CREATE FUNCTION gman_do_high_background RETURNS STRING
       SONAME "libgearman_mysql_udf.so";
CREATE FUNCTION gman_do_low_background RETURNS STRING
       SONAME "libgearman_mysql_udf.so";
CREATE AGGREGATE FUNCTION gman_sum RETURNS INTEGER
       SONAME "libgearman_mysql_udf.so";
CREATE FUNCTION gman_servers_set RETURNS STRING
       SONAME "libgearman_mysql_udf.so";

Once loaded, you'll need to add servers for the clients to query
first. This can be done with:

SELECT gman_servers_set("127.0.0.1");

You can also add a different set of servers for each function
type. For example:

SELECT gman_servers_set("192.168.1.1", "resize");

This would direct future queries to use 192.168.1.1 for all Gearman
functions calls to "resize". You can also specify multiple job servers
and port numbers in a single query using the following syntax:

SELECT gman_servers_set("192.168.1.3:7004,192.168.1.4:7004", "index");

Once your servers are setup, you can then run jobs from your queries:

SELECT gman_do("reverse", Host) AS test FROM mysql.user;

SELECT gman_do_high("reverse", Host) AS test FROM mysql.user;

SELECT gman_do_background("reverse", Host) AS test FROM mysql.user;

SELECT gman_sum("wc", Host) AS test FROM mysql.user;

These examples run a normal job, a high-priority job, or a background
job, respectively. The last job does not return a result, but instead
a job handle that you can later use to query for status. In order to
get the result from a background job, the worker would need to store
the result someplace a client can find it again later, such as back
into a MySQL table, into memcached, or even a file somewhere both
the client and worker can access.

The 'gman_sum' function is provided as an aggregate example, and may be
useful for certain cases. You may wish to write your own specialized
aggregate operator that does something else with the results besides
sum the values. The main difference between 'sum(gman_do(...))' and
'gman_sum(...)' is that 'gman_sum' run all jobs concurrently, where
'gman_do' must run jobs serially. This means if you have multiple
workers setup, a 'gman_sum' can be much faster than 'gman_do'.

If you would like to get the above examples working, you need to run
'gearmand' to start the job server, and then run 'reverse_worker'
or 'wc_worker' (depending on the function above) from the examples
directory in the C server and library package.

The gman_do* functions take an optional third parameter, which is
the unique job ID. This allows you to submit multiple jobs under
the same unique ID, and they will be "merged" in the queue to be
run only once. Note that gearmand will only merge jobs that are in
queue or running, it doesn't keep track of unique IDs after a job
finishes. For example, the following makes sure you only run the job
once for each host:

SELECT gman_do_background("reverse", Host, Host) AS test FROM mysql.user;

If you ever need to remove the functions from MySQL, you can run:

DROP FUNCTION gman_do;
DROP FUNCTION gman_do_high;
DROP FUNCTION gman_do_low;
DROP FUNCTION gman_do_background;
DROP FUNCTION gman_do_high_background;
DROP FUNCTION gman_do_low_background;
DROP FUNCTION gman_sum;
DROP FUNCTION gman_servers_set;

Enjoy!
-Eric
</pre></html>