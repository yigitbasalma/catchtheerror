# Templates

Templates are special tools that are ready to use on the system in a predefined way and contain items, triggers and many more components. It is possible to find pre-defined templates suitable for almost every need.

You can access the templates on the system under "General Settings > Templates", click on the three dots in the upper right corner of the template cards and examine the details by clicking the "Details" button from the drop-down list.

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

## Triggers

We talked about how to get data from the systems we monitor. Now it's time to turn them into alarms. The name of the concept that came into our lives here is "Triggers". By evaluating the items we collect, we can turn them into meaningful triggers. This often allows us to take action before the event occurs.

For example, the trigger definition below will start to warn if the CPU usage of the system stays above the threshold you set for 5 minutes. The parameter that plays a critical role here is the threshold value you will set. If this value is defined correctly, the risk of system interruption is minimized.

__Important Note:__ [Makro](../components/macros) is used for threshold value definition.

```bash
min(/Linux CPU by Zabbix agent/system.cpu.util,5m)>{$CPU.UTIL.CRIT}
```

## Trigger With Dependency

Triggers can be used alone, or it is possible to define the necessity of triggering another trigger before a trigger is triggered. The aim here is to quickly go to the root cause and prevent unnecessary alarm crowd.

In order for the trigger in our previous example to occur, the following trigger must be created.

```bash
min(/Linux CPU by Zabbix agent/system.cpu.load[all,avg1],5m)/last(/Linux CPU by Zabbix agent/system.cpu.num)>{$LOAD_AVG_PER_CPU.MAX.WARN} and \
 last(/Linux CPU by Zabbix agent/system.cpu.load[all,avg5])>0 and \
 last(/Linux CPU by Zabbix agent/system.cpu.load[all,avg15])>0
```

The explanation of this trigger is that if the 1 minute load average values of all CPU cores in the last 5 minutes divided by the total CPU cores, the remaining value is above the threshold value you set, and if the 5-minute and 15-minute load average values are also above zero.

## Low Level Discovery

Variable source types are also available on the systems. The ideal examples for these are disk mounts and ethernet interfaces. We call LLD (Low Level Discover) rules, which are special rules that actively scan such systems at certain intervals, define items and triggers within the framework of certain rules for new additions, and delete those that have been removed after a certain period.

The main logic in LLD rules is to parse the JSON response returned by a special item run on the system. For example, the item key below is predefined on the zabbix agent and returns the ethernet interfaces on the system as a JSON response.

```bash
net.if.discovery
```