# stacky

Command line interface to StackTach (https://github.com/rackerlabs/stacktach)

set STACKTACH_URL to point to your StackTach web server.

## Requires
pip install -r pip_requires.txt

## Help
```
$ ./stacky.py 
Usage: stacky <command>
    deployments - list stacktach deployments
    events      - list of unique event names
    watch   - watch <deployment id> <service-name> <event-name> <polling sec>
              deployment id :let this option be "all" if you want to watch all
              deployments or just a specific deployment-id
              service-name : name of whichever service you want to poll,
              i.e nova, glance or generic
              event-name :  = If not given a specific event, it will keep a
              watch on all events.
              polling : 2s default or can be specified as number of seconds
    show    - 'show <service-name> <event-id>'
              service-name :name of whichever service you want to poll,
              i.e nova, glance or generic, default will be nova
    uuid    - inspect events with uuid xxxxx
    summary - show summarized timings for all events
    timings - show timings for <event-name> (no .start/.end)
    request - show events with <request id>
    reports - list all reports created from <start> to <end>
              Default is today.
              All times 24-hr UTC in YYYY-MM-DD HH:MM format.
              Example: stacky reports 2013-02-28 00:00 2013-02-28 06:00
                       will return all reports created from midnight to 6am
                       on Feb 28th.
    report  - get report <id>
    kpi     - crunch KPIs
    hosts   - list all host names
    search  - 'search <service-name> <field> <value>'
              search for rawdata events belonging to any service - nova, glance
              or generic according to any field and its value.
              You can also pass limit on results, and duration of interest:
              'search <service-name> <field> <value> <limit> "<start datetime>" "<end datetime>"'
```

NOTE: *kpi* and *watch* commands are currently disabled. Will be fixed soon.

## Examples

### List event types
```
$ python stacky.py events
+---------------------------------------+
|               Event Name              |
+---------------------------------------+
|      compute.instance.create.end      |
|     compute.instance.create.start     |
|      compute.instance.delete.end      |
|     compute.instance.delete.start     |
|        compute.instance.exists        |
|  compute.instance.finish_resize.start |
|      compute.instance.reboot.end      |
|     compute.instance.reboot.start     |
|      compute.instance.rebuild.end     |
|     compute.instance.rebuild.start    |
|  compute.instance.resize.confirm.end  |
| compute.instance.resize.confirm.start |
|      compute.instance.resize.end      |
|    compute.instance.resize.prep.end   |
|   compute.instance.resize.prep.start  |
|     compute.instance.resize.start     |
|     compute.instance.shutdown.end     |
|    compute.instance.shutdown.start    |
|    compute.instance.snapshot.start    |
|        compute.instance.update        |
|       scheduler.run_instance.end      |
|    scheduler.run_instance.scheduled   |
|      scheduler.run_instance.start     |
+---------------------------------------+
```

### Lookup Nova instance by UUID
```
$ python stacky.py uuid bafe36de-aba8-46e8-9fe7-15b490e4cc01
Events related to bafe36de-aba8-46e8-9fe7-15b490e4cc01
+----------+---+----------------------------+--------------------+----------------------------------+------------------------------------------------------+----------+----------+----------------------+
|    #     | ? |            When            |     Deployment     |              Event               |                         Host                         |  State   |  State'  |        Task'         |
+----------+---+----------------------------+--------------------+----------------------------------+------------------------------------------------------+----------+----------+----------------------+
| 16373586 |   | 2013-04-09 14:31:50.345266 | cellA |     compute.instance.update      |   nova-api.foo.com    | building |   None   |         None         |
| 16373589 |   | 2013-04-09 14:31:54.211449 | cellB  | scheduler.run_instance.scheduled | nova-scheduler.foo.com |          |          |                      |
| 16373603 |   | 2013-04-09 14:32:11.773272 | cellB  |     compute.instance.update      |                   computehostA                    | building | building |      scheduling      |
... (and so on)
```

### Get details for an event
Take an event id from the above example and show its details:
```
$ python stacky.py show nova 16373586
+------------+-----------------------------------------------------+
|    Key     |                        Value                        |
+------------+-----------------------------------------------------+
|     #      |                       16373586                      |
|    When    |              2013-04-09 14:31:50.345266             |
| Deployment |                        cellA                        |
|  Category  |                     monitor.info                    |
| Publisher  |                  nova-api.foo.com                   |
|   State    |                       building                      |
|   Event    |               compute.instance.update               |
|  Service   |                         api                         |
|    Host    |                  nova-api.foo.com                   |
|    UUID    |         bafe36de-aba8-46e8-9fe7-15b490e4cc01        |
|   Req ID   |       req-5539174d-0cd9-40d9-a7d5-5aa883c20033      |
+------------+-----------------------------------------------------+
... (and so on)
```

### Get all events for request ID
Take the request id from above and get all the associated events:
```
$ python stacky.py request req-5539174d-0cd9-40d9-a7d5-5aa883c20033
+----------+---+----------------------------+--------------------+----------------------------------+------------------------------------------------------+----------+----------+----------------------+
|    #     | ? |            When            |     Deployment     |              Event               |                         Host                         |  State   |  State'  |        Task'         |
+----------+---+----------------------------+--------------------+----------------------------------+------------------------------------------------------+----------+----------+----------------------+
| 16373586 |   | 2013-04-09 14:31:50.345266 | cellA |     compute.instance.update      |   nova-api.foo.com    | building |   None   |         None         |
| 16373587 |   | 2013-04-09 14:31:51.196901 | cellB  |   scheduler.run_instance.start   | nova-scheduler.foo.com |          |          |                      |
| 16373589 |   | 2013-04-09 14:31:54.211449 | cellB  | scheduler.run_instance.scheduled | nova-scheduler.foo.com |          |          |                      |
| 16373592 |   | 2013-04-09 14:31:57.497054 | cellB  |    scheduler.run_instance.end    | nova-scheduler.foo.com |          |          |                      |
| 16373603 |   | 2013-04-09 14:32:11.773272 | cellB  |     compute.instance.update      |                   computehostA                    | building | building |      scheduling      |
... (and so on)
```

### List all deployments configured on stachtach to monitor for events
```
$ python stacky.py deployments
+---+--------------------+
| # |        Name        |
+---+--------------------+
| 1 | j-g2s4-kilo-1-4319 |
| 2 | j-g2s8-kilo-1-4547 |
+---+--------------------+
```

### Search for rawdata events belonging to any service for a given time frame
Example: Searching all events on nova service for deployment node 1, between '2017-07-27 23:31:38' and '2017-07-30 23:31:40', limiting 20 results.

```
$ python stacky.py search nova deployment_id 1 20 '2017-07-27 23:31:38' '2017-07-30 23:31:40'
+-----+---+----------------------------+--------------------+-------------------------------+--------------------+--------+--------+-------+
|  #  | ? |            When            |     Deployment     |             Event             |        Host        | State  | State' | Task' |
+-----+---+----------------------------+--------------------+-------------------------------+--------------------+--------+--------+-------+
| 145 |   | 2017-07-30 20:19:42.607541 | j-g2s4-kilo-1-4319 |  compute.instance.reboot.end  | j-g2s4-kilo-2-5327 | active |        |       |
| 144 |   | 2017-07-30 20:19:38.613726 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | active |        |       |
| 143 | E | 2017-07-27 23:31:38.236789 | j-g2s4-kilo-1-4319 |     compute.libvirt.error     | j-g2s4-kilo-2-5327 |        |        |       |
+-----+---+----------------------------+--------------------+-------------------------------+--------------------+--------+--------+-------+
```

Example: Searching all events on nova service for tenant ca83e6bc934d4d6caf1ac94f974ab9ae, between '2017-07-27 23:31:38' and '2017-07-30 23:31:40', limiting 20 results.
```
$ python stacky.py search nova tenant ca83e6bc934d4d6caf1ac94f974ab9ae  20 '2017-07-27 23:31:38' '2017-07-30 23:31:40'
+-----+---+----------------------------+--------------------+-------------------------------+--------------------+--------+--------+-------+
|  #  | ? |            When            |     Deployment     |             Event             |        Host        | State  | State' | Task' |
+-----+---+----------------------------+--------------------+-------------------------------+--------------------+--------+--------+-------+
| 145 |   | 2017-07-30 20:19:42.607541 | j-g2s4-kilo-1-4319 |  compute.instance.reboot.end  | j-g2s4-kilo-2-5327 | active |        |       |
| 144 |   | 2017-07-30 20:19:38.613726 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | active |        |       |
+-----+---+----------------------------+--------------------+-------------------------------+--------------------+--------+--------+-------+
```

Example: Searching all events on nova service for event 'compute.instance.reboot.start':
```
$ python stacky.py search nova event compute.instance.reboot.start
+-----+---+----------------------------+--------------------+-------------------------------+--------------------+--------+--------+-------+
|  #  | ? |            When            |     Deployment     |             Event             |        Host        | State  | State' | Task' |
+-----+---+----------------------------+--------------------+-------------------------------+--------------------+--------+--------+-------+
| 251 |   | 2017-08-15 05:32:37.625245 | j-g2s8-kilo-1-4547 | compute.instance.reboot.start | j-g2s8-kilo-2-6549 | active |        |       |
| 249 |   | 2017-08-10 05:22:55.653390 | j-g2s8-kilo-1-4547 | compute.instance.reboot.start | j-g2s8-kilo-2-6549 | active |        |       |
| 247 |   | 2017-08-10 05:22:48.999138 | j-g2s8-kilo-1-4547 | compute.instance.reboot.start | j-g2s8-kilo-2-6549 | active |        |       |
| 245 |   | 2017-08-10 05:21:37.574440 | j-g2s8-kilo-1-4547 | compute.instance.reboot.start | j-g2s8-kilo-2-6549 | active |        |       |
| 243 |   | 2017-08-10 05:21:07.403328 | j-g2s8-kilo-1-4547 | compute.instance.reboot.start | j-g2s8-kilo-2-6549 | active |        |       |
| 241 |   | 2017-08-10 05:02:24.202181 | j-g2s8-kilo-1-4547 | compute.instance.reboot.start | j-g2s8-kilo-2-6549 | active |        |       |
| 239 |   | 2017-08-10 04:41:58.422175 | j-g2s8-kilo-1-4547 | compute.instance.reboot.start | j-g2s8-kilo-2-6549 | active |        |       |
| 237 |   | 2017-08-10 04:10:12.768241 | j-g2s8-kilo-1-4547 | compute.instance.reboot.start | j-g2s8-kilo-2-6549 | active |        |       |
| 235 |   | 2017-08-10 00:47:32.034192 | j-g2s8-kilo-1-4547 | compute.instance.reboot.start | j-g2s8-kilo-2-6549 | active |        |       |
| 233 |   | 2017-08-10 00:39:27.910512 | j-g2s8-kilo-1-4547 | compute.instance.reboot.start | j-g2s8-kilo-2-6549 | active |        |       |
| 231 |   | 2017-08-10 00:37:59.212735 | j-g2s8-kilo-1-4547 | compute.instance.reboot.start | j-g2s8-kilo-2-6549 | active |        |       |
| 229 |   | 2017-08-09 23:43:47.315095 | j-g2s8-kilo-1-4547 | compute.instance.reboot.start | j-g2s8-kilo-2-6549 | active |        |       |
| 227 |   | 2017-08-09 23:41:41.292486 | j-g2s8-kilo-1-4547 | compute.instance.reboot.start | j-g2s8-kilo-2-6549 | active |        |       |
| 225 |   | 2017-08-09 23:32:07.371495 | j-g2s8-kilo-1-4547 | compute.instance.reboot.start | j-g2s8-kilo-2-6549 | active |        |       |
| 223 |   | 2017-08-09 23:28:11.373375 | j-g2s8-kilo-1-4547 | compute.instance.reboot.start | j-g2s8-kilo-2-6549 | active |        |       |
| 221 |   | 2017-08-09 23:20:54.732545 | j-g2s8-kilo-1-4547 | compute.instance.reboot.start | j-g2s8-kilo-2-6549 | active |        |       |
| 219 |   | 2017-08-09 23:14:16.520805 | j-g2s8-kilo-1-4547 | compute.instance.reboot.start | j-g2s8-kilo-2-6549 | active |        |       |
| 217 |   | 2017-08-09 23:11:48.224939 | j-g2s8-kilo-1-4547 | compute.instance.reboot.start | j-g2s8-kilo-2-6549 | active |        |       |
| 215 |   | 2017-08-09 23:10:31.182164 | j-g2s8-kilo-1-4547 | compute.instance.reboot.start | j-g2s8-kilo-2-6549 | active |        |       |
| 213 |   | 2017-08-09 23:09:10.657919 | j-g2s8-kilo-1-4547 | compute.instance.reboot.start | j-g2s8-kilo-2-6549 | active |        |       |
| 191 |   | 2017-08-08 21:52:00.192223 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | active |        |       |
| 189 |   | 2017-08-08 21:49:38.887607 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | active |        |       |
| 187 |   | 2017-08-08 21:32:32.096817 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | active |        |       |
| 185 |   | 2017-08-08 21:05:13.394918 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | active |        |       |
| 183 |   | 2017-08-08 21:02:07.845803 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | active |        |       |
| 181 |   | 2017-08-08 20:58:24.451998 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | active |        |       |
| 179 |   | 2017-08-08 20:37:16.692284 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | active |        |       |
| 177 |   | 2017-08-08 20:31:39.747477 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | active |        |       |
| 175 |   | 2017-08-08 19:25:04.886559 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | active |        |       |
| 173 |   | 2017-08-08 19:21:23.320767 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | active |        |       |
| 171 |   | 2017-08-08 18:18:22.291989 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | active |        |       |
| 169 |   | 2017-08-08 17:32:55.464050 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | active |        |       |
| 167 |   | 2017-08-08 00:29:47.512147 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | active |        |       |
| 165 |   | 2017-08-08 00:27:23.214272 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | active |        |       |
| 163 |   | 2017-08-08 00:18:52.455834 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | active |        |       |
| 161 |   | 2017-08-08 00:07:12.829193 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | active |        |       |
| 159 |   | 2017-08-08 00:02:42.862015 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | active |        |       |
| 157 |   | 2017-08-02 00:30:58.844547 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | active |        |       |
| 155 |   | 2017-08-02 00:05:35.758426 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | active |        |       |
| 153 |   | 2017-08-01 19:05:41.057068 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | active |        |       |
| 151 |   | 2017-08-01 18:25:20.860796 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | active |        |       |
| 146 |   | 2017-07-31 22:50:51.406844 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | active |        |       |
| 144 |   | 2017-07-30 20:19:38.613726 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | active |        |       |
| 127 |   | 2017-07-25 22:31:03.735723 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | active |        |       |
| 125 |   | 2017-07-25 19:15:14.541101 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | active |        |       |
| 119 |   | 2017-07-25 00:20:58.847064 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | active |        |       |
| 117 |   | 2017-07-24 23:37:18.258383 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | active |        |       |
| 111 |   | 2017-07-24 20:57:47.015922 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | active |        |       |
| 105 |   | 2017-07-21 17:14:04.253091 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | active |        |       |
|  99 |   | 2017-07-21 01:00:05.464092 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | error  |        |       |
+-----+---+----------------------------+--------------------+-------------------------------+--------------------+--------+--------+-------+
```

Example: Searching all events on nova service for event 'compute.instance.reboot.start' limiting 5 results:
```
$ python stacky.py search nova event compute.instance.reboot.start 5
+-----+---+----------------------------+--------------------+-------------------------------+--------------------+--------+--------+-------+
|  #  | ? |            When            |     Deployment     |             Event             |        Host        | State  | State' | Task' |
+-----+---+----------------------------+--------------------+-------------------------------+--------------------+--------+--------+-------+
| 251 |   | 2017-08-15 05:32:37.625245 | j-g2s8-kilo-1-4547 | compute.instance.reboot.start | j-g2s8-kilo-2-6549 | active |        |       |
| 249 |   | 2017-08-10 05:22:55.653390 | j-g2s8-kilo-1-4547 | compute.instance.reboot.start | j-g2s8-kilo-2-6549 | active |        |       |
| 247 |   | 2017-08-10 05:22:48.999138 | j-g2s8-kilo-1-4547 | compute.instance.reboot.start | j-g2s8-kilo-2-6549 | active |        |       |
| 245 |   | 2017-08-10 05:21:37.574440 | j-g2s8-kilo-1-4547 | compute.instance.reboot.start | j-g2s8-kilo-2-6549 | active |        |       |
| 243 |   | 2017-08-10 05:21:07.403328 | j-g2s8-kilo-1-4547 | compute.instance.reboot.start | j-g2s8-kilo-2-6549 | active |        |       |
+-----+---+----------------------------+--------------------+-------------------------------+--------------------+--------+--------+-------+
```
Example: Searching all events on nova service for event 'compute.instance.reboot.start' limiting 5 results between time range :'2017-07-27 23:31:38' '2017-07-30 23:31:40'
```
$ python stacky.py search nova event compute.instance.reboot.start 5 '2017-07-27 23:31:38' '2017-07-30 23:31:40'
+-----+---+----------------------------+--------------------+-------------------------------+--------------------+--------+--------+-------+
|  #  | ? |            When            |     Deployment     |             Event             |        Host        | State  | State' | Task' |
+-----+---+----------------------------+--------------------+-------------------------------+--------------------+--------+--------+-------+
| 144 |   | 2017-07-30 20:19:38.613726 | j-g2s4-kilo-1-4319 | compute.instance.reboot.start | j-g2s4-kilo-2-5327 | active |        |       |
+-----+---+----------------------------+--------------------+-------------------------------+--------------------+--------+--------+-------+
```
