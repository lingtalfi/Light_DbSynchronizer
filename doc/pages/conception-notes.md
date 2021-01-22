Light_DbSynchronizer
===========
2020-06-09 -> 2021-01-22




The goal of this tool is to synchronize parts of an existing database from a **create file**, which is a sql file which contains **create table** statements.


It's a semi-automated tool, meaning it assists you in doing the synchronization.



So the idea is that this tool reads the [create file](https://github.com/lingtalfi/TheBar/blob/master/discussions/create-file.md),
and basically syncs it with the current database.

So, it might either add new tables to the existing database, or update the table structure, or even remove tables.

For the removing of tables, for the user/developer's own safety, our tool will only remove a table if it's in a list provided by the user (to avoid accidental removal of important tables).

Note: the **create file** might contains other statements, such as drop statements, but our tool doesn't care about them, it just
parses the **create table** statements. So basically you can use files generated with tools such as MysqlWorkBench to generate your **create files**.


This tool does generally a good job, however, there are some things you should be aware of.

In particular, you need to be careful when renaming tables and columns.


The thing is that our tool can't guess if you want to rename a table (or a column), or if you want to delete it and recreate it.

Note: humans can't guess it either if you think about it.

So, by default, it will delete it and recreate it, which means if your thing (table or column) contained data, it won't anymore.


There is a workaround, explained in the [renaming items](#renaming-items) section, which allows us to tell the synchronizer tool to really rename items, 
instead of dumbly just delete/recreate them.






Renaming items
------------
2020-06-18 -> 2021-01-22


In the **create file**, we can rename columns, using a "special statement" like this one:

```sql
-- @rename lud_user:name->maurice
```

This will prepare the synchronizer to rename the "name" column from the lud_user table to "maurice".
Note: we still need to update the **CREATE TABLE** statement accordingly (i.e. change the name column to maurice) in order for the renaming to be effective.


A special statement must fit on one line.

Note that it starts with the double dash, so that it's interpreted as a comment in sql language.



If you rename multiple columns, just use multiple "special statements", one per line:


```sql
-- @rename lud_user:name->label
-- @rename lud_user:description->biography
```


We can also indicate the renamed tables, by adding the table prefix, like this:

```sql
-- @rename table lud_user->lud_user2
```

Optionally, we can add the column prefix to indicate column renaming, in order to make the syntax more consistent:


```sql
-- @rename column lud_user:name->label
-- @rename column lud_user:description->biography
-- @rename table lud_user->lud_user2
```




So, as we can we use the sql comment (double dash followed by space at the beginning of the line), so that we can safely add those
special statements to the **create file**.


Don't forget to remove those special statements after usage, just for the sake of having a clean file.








  





Logging
=========
2020-06-19

This tool uses the [Light_Logger](https://github.com/lingtalfi/Light_Logger) plugin to log two types of messages:


- debug messages
- error messages


Debug messages include error messages.


By default, debug messages are sent on the **db_synchronizer.debug** channel, in a file: **${app_dir}/log/db_synchronizer_debug.txt**,
and error messages are sent on the **db_synchronizer.error** channel, in a file: **${app_dir}/log/db_synchronizer_error.txt**.

The debug messages file is reset on every call to the tool's **synchronize** method (I found this is practical to debug to have a clean debug page every time). 

You can change any of this from the service configuration.


The logged errors are all failed statements, such as:

- SQLSTATE[HY000]: General error: 1553 Cannot drop index 'PRIMARY': needed in a foreign key constraint
- ...


In other words, whenever a "synchronization statement" fails, it will be reported in the logs.


The debug log will trace all the steps taken by the tool.
 
This includes all the executed statements.

As the name suggests, it's useful for debugging.