# RunCached

Execute commands while caching (memoizing) their output (stdout, stderr, exit code) on subsequent calls 
for a configurable duration.

Command output will be cached for <cacheperiod> seconds and "replayed" for 
any subsequent calls. Original exit status will also be emulated.

## Details
If command is run after cacheperiod has expired, the actual command will be re-executed and a new result 
will be cached. 

Cache data is tied to the **command** and **arguments** executed and the 
**path** of the executable. Cache results are stored in /tmp

You can use runcached to run resource-expensive commands multiple times, 
parsing different parts of their output each time. Those commands will be
run only once for each cacheperiod. 

Implementation is not fancy, just works. It is provided in 3 languages, python, C, BASH to suit different environments. The BASH version is not really suggested but it works. The python is probably what you want.


## Usage

### Python
runcached.py [-c cacheperiod] <command to execute with args>

### C
runcached [-c cacheperiod] <command to execute with args>

### Bash
runcached.sh  <command to execute with args>



## Examples


### Example 1:  Run the date command. Each time it executes, it displays the same date for 5 seconds at a time.
runcached.py -c 5 date

### Example 2: Zabbix userparameter which can be called multiple times, but in reality executes only once every 20 seconds. 
Query multiple parameters of mysql at the same time, without re-running the query.


```
#!

UserParameter=mysql.globalstatus[*],/usr/local/bin/runcached.py -c 20 /usr/bin/mysql -ANe \"show global status\"|egrep '$1\b'|awk '{print $ 2}'
```


And then define some items like so:

```
#!nolang
Item Name                      Item Key
--------------                  --------------
MySQL DELETES	 	mysql.globalstatus[Com_delete]
MySQL INSERTS	 	mysql.globalstatus[Com_insert]
MySQL UPDATES	 	mysql.globalstatus[Com_update]
MySQL CREATE TABLE	mysql.globalstatus[Com_create_table]
MySQL SELECTS	 	mysql.globalstatus[Com_select]
MySQL Uptime	 	mysql.globalstatus[Uptime]
MySQL ALTER TABLE	mysql.globalstatus[Com_alter_table]

E.g. for DELETE: 
Type: Numeric, 
Data Type: Decimal. 
Units: QPS
Store Value: Delta (Speed per second)
Show Value: As Is
```
