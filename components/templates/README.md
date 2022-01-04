# Templates

Templates are special tools that are ready to use on the system in a predefined way and contain items, triggers and many more components. It is possible to find pre-defined templates suitable for almost every need.

## Items

The starting point of the monitoring process is to gather information from the target to be tracked. For us, "items" does the job. Items are objects that run the assigned job and store the returned result.

For example, the key below is executed by the zabbix agent and returns the hostname of the target server. You can find more information about the items [here](https://www.zabbix.com/documentation/2.2/en/manual/config/items/itemtypes).

```bash
agent.hostname
```

There are some important points to consider when defining an item. You can find them below.

* While defining the item, the "history" parameter should be given as short as possible. Thus, the database load is alleviated. It should be as small as possible but not 0. Because Zabbix server uses historical data when calculating trigger.
* It is necessary to choose the correct return value type for the item we are running. For example, if the item you defined is "cat /etc/redhat-release", its return type will be "text". Otherwise, the item will get an error and will not collect data.
* "Update Interval" is another important parameter. This value should not be greater or less than necessary. Very low intervals can cause data inconsistency and load on the server.