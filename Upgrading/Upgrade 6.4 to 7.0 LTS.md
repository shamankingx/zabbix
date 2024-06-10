Upgrade Zabbix from 6.4 to 7.0 LTS

This procedure using below environment
OS: Ubuntu 22.04.3 LTS
Zabbix: 6.4
Mysql: 8.0.36
Nginx: 1.18.0

1 Stop Zabbix processes

- Stop Zabbix server to make sure that no new data is inserted into database.

```
sudo service zabbix-server stop
```
or
```
sudo systemctl stop zabbix-server.service
```

If upgrading Zabbix proxy, stop proxy too.

```
sudo service zabbix-proxy stop
```
or
```
sudo systemctl stop zabbix-proxy
```

2 Back up the existing Zabbix database
- This is a very important step. Make sure that you have a backup of your database. It will help if the upgrade procedure fails (lack of disk space, power off, any unexpected problem).

3 Back up configuration files, PHP files and Zabbix binaries
- Make a backup copy of Zabbix binaries, configuration files and the PHP file directory.

Configuration files:

If using Apache2
```
mkdir /opt/zabbix-backup/
cp /etc/zabbix/zabbix_server.conf /opt/zabbix-backup/
cp /etc/apache2/conf-enabled/zabbix.conf /opt/zabbix-backup/
```

If using Nginx
```
mkdir /opt/zabbix-backup/
cp /etc/zabbix/zabbix_server.conf /opt/zabbix-backup/
cp /etc/nginx/conf.d/zabbix.conf /opt/zabbix-backup/
```

PHP files and Zabbix binaries:

```
cp -R /usr/share/zabbix/ /opt/zabbix-backup/
cp -R /usr/share/zabbix-* /opt/zabbix-backup/
```

4 Update repository configuration package
- To proceed with the update your current repository package has to be uninstalled.

```
rm -Rf /etc/apt/sources.list.d/zabbix.list.dpkg-dist
rm -Rf /etc/apt/sources.list.d/zabbix.list
cp /etc/apt/sources.list.d/zabbix.list.dpkg-dist /etc/apt/resources.list.d/zabbix.list
```

Then install the new repository configuration package.

On Ubuntu 22.04 run:

```
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.0-1+ubuntu22.04_all.deb
dpkg -i zabbix-release_7.0-1+ubuntu22.04_all.deb
```

Update the repository information.

```
sudo apt-get update
```

Upgrade process
To complete a successful Zabbix server upgrade on MySQL/MariaDB, you may require to set GLOBAL log_bin_trust_function_creators = 1 in MySQL if binary logging is enabled, there are no superuser privileges and log_bin_trust_function_creators = 1 is not set in MySQL configuration file.

To set the variable using the MySQL console, run:


mysql> ```SET GLOBAL log_bin_trust_function_creators = 1;```

Once the upgrade has been successfully completed, this option can be disabled:

mysql> ```SET GLOBAL log_bin_trust_function_creators = 0;```

5 Upgrade Zabbix components
- To upgrade Zabbix components you may run something like:

```
sudo apt-get install --only-upgrade zabbix-server-mysql zabbix-frontend-php zabbix-agent zabbix-get zabbix-nginx-conf zabbix-sql-scripts zabbix-web-service zabbix-release
```

If using PostgreSQL, substitute mysql with pgsql in the command. If upgrading the proxy, substitute server with proxy in the command. If upgrading the Zabbix agent 2, substitute zabbix-agent with zabbix-agent2 in the command.

Recheck installed package
```
sudo apt list --installed | grep zabbix
```

If you installed all packages will show such as below.

zabbix-agent2/zabbix,now 1:7.0.0-1+ubuntu22.04 amd64 [installed]
zabbix-frontend-php/zabbix,now 1:7.0.0-1+ubuntu22.04 all [installed]
zabbix-get/zabbix,now 1:7.0.0-1+ubuntu22.04 amd64 [installed]
zabbix-nginx-conf/zabbix,now 1:7.0.0-1+ubuntu22.04 all [installed]
zabbix-release/zabbix,now 1:7.0-1+ubuntu22.04 all [installed]
zabbix-server-mysql/zabbix,now 1:7.0.0-1+ubuntu22.04 amd64 [installed]
zabbix-sql-scripts/zabbix,now 1:7.0.0-1+ubuntu22.04 all [installed]
zabbix-web-service/zabbix,now 1:7.0.0-1+ubuntu22.04 amd64 [installed]

Database upgrade
```
sudo zabbix_server -c /etc/zabbix/zabbix_server.conf
```

View log while upgrading
```
sudo tail -f /var/log/zabbix/zabbix_server.log
```
Example of log
2878035:20240610:080407.429 completed 18% of database upgrade
2878035:20240610:080407.453 completed 19% of database upgrade
2878035:20240610:080407.549 completed 20% of database upgrade
2878035:20240610:080407.671 completed 21% of database upgrade
2878035:20240610:080407.853 completed 22% of database upgrade
2878035:20240610:080408.102 completed 23% of database upgrade
2878035:20240610:080408.232 completed 24% of database upgrade
2878035:20240610:080408.435 completed 25% of database upgrade
2878035:20240610:080408.580 completed 26% of database upgrade
2878035:20240610:080408.723 completed 27% of database upgrade
2878035:20240610:080418.702 slow query: 9.978859 sec, "update items set timeout='' where type not in (19,21)"
2878035:20240610:080419.724 completed 28% of database upgrade
2878035:20240610:080420.030 completed 29% of database upgrade
2878035:20240610:080420.164 completed 30% of database upgrade
2878035:20240610:080420.382 completed 31% of database upgrade
2878035:20240610:080420.670 completed 32% of database upgrade
2878035:20240610:080420.689 completed 33% of database upgrade
2878035:20240610:080420.751 completed 34% of database upgrade
2878035:20240610:080420.774 completed 35% of database upgrade
2878035:20240610:080420.937 completed 36% of database upgrade
2878035:20240610:080420.964 completed 37% of database upgrade
2878035:20240610:080421.004 completed 38% of database upgrade
2878035:20240610:080421.033 completed 39% of database upgrade
2878035:20240610:080421.143 completed 40% of database upgrade
2878035:20240610:080425.796 slow query: 4.652906 sec, "alter table `items` modify `query_fields` text not null"
2878035:20240610:080425.922 completed 41% of database upgrade
2878035:20240610:080426.066 completed 42% of database upgrade
2878035:20240610:080427.419 completed 43% of database upgrade
2878035:20240610:080427.775 completed 44% of database upgrade
2878035:20240610:080427.803 completed 45% of database upgrade
2878035:20240610:080428.144 completed 46% of database upgrade
2878035:20240610:080428.343 completed 47% of database upgrade
2878035:20240610:080428.530 completed 48% of database upgrade
2878035:20240610:080428.679 completed 49% of database upgrade
2878035:20240610:080428.712 completed 50% of database upgrade
2878035:20240610:080428.823 completed 51% of database upgrade
2878035:20240610:080428.911 completed 52% of database upgrade
2878035:20240610:080429.103 completed 53% of database upgrade
2878035:20240610:080429.415 completed 54% of database upgrade
2878035:20240610:080429.647 completed 55% of database upgrade
2878035:20240610:080429.840 completed 56% of database upgrade
2878035:20240610:080430.143 completed 57% of database upgrade
2878035:20240610:080430.358 completed 58% of database upgrade
2878035:20240610:080430.632 completed 59% of database upgrade
2878035:20240610:080430.842 completed 60% of database upgrade
2878035:20240610:080431.137 completed 61% of database upgrade
2878035:20240610:080431.199 completed 62% of database upgrade
2878035:20240610:080431.288 completed 63% of database upgrade
2878035:20240610:080431.381 completed 64% of database upgrade
2878035:20240610:080431.552 completed 65% of database upgrade
2878035:20240610:080431.971 completed 66% of database upgrade
2878035:20240610:080432.220 completed 67% of database upgrade
2878035:20240610:080432.308 completed 68% of database upgrade
2878035:20240610:080432.793 completed 69% of database upgrade
2878035:20240610:080433.052 completed 70% of database upgrade
2878035:20240610:080433.217 completed 71% of database upgrade
2878035:20240610:080433.422 completed 72% of database upgrade
2878035:20240610:080433.744 completed 73% of database upgrade
2878035:20240610:080434.130 completed 74% of database upgrade
2878035:20240610:080434.299 completed 75% of database upgrade
2878035:20240610:080434.431 completed 76% of database upgrade
2878035:20240610:080434.603 completed 77% of database upgrade
2878035:20240610:080442.997 slow query: 8.393859 sec, "update items set lifetime='7d' where flags in (0,2,4)"
2878035:20240610:080444.219 completed 78% of database upgrade
2878035:20240610:080445.152 completed 79% of database upgrade
2878035:20240610:080445.196 completed 80% of database upgrade
2878035:20240610:080445.277 completed 81% of database upgrade
2878035:20240610:080445.362 completed 82% of database upgrade
2878035:20240610:080445.459 completed 83% of database upgrade
2878035:20240610:080445.586 completed 84% of database upgrade
2878035:20240610:080445.893 completed 85% of database upgrade
2878035:20240610:080445.950 completed 86% of database upgrade
2878035:20240610:080446.205 completed 87% of database upgrade
2878035:20240610:080446.445 completed 88% of database upgrade
2878035:20240610:080447.217 completed 89% of database upgrade
2878035:20240610:080447.404 completed 90% of database upgrade
2878035:20240610:080447.469 completed 91% of database upgrade
2878035:20240610:080447.781 completed 92% of database upgrade
2878035:20240610:080447.973 completed 93% of database upgrade
2878035:20240610:080448.175 completed 94% of database upgrade
2878035:20240610:080448.295 completed 95% of database upgrade
2878035:20240610:080448.438 completed 96% of database upgrade
2878035:20240610:080448.502 completed 97% of database upgrade
2878035:20240610:080448.545 completed 98% of database upgrade
2878035:20240610:080448.577 completed 99% of database upgrade
2878035:20240610:080448.597 completed 100% of database upgrade
2878035:20240610:080453.545 slow query: 4.948335 sec, "delete from changelog"
2878035:20240610:080453.545 database upgrade fully completed
2878123:20240610:080453.651 starting HA manager
2878123:20240610:080453.672 HA manager started in active mode
2878035:20240610:080453.678 server #0 started [main process]
2878124:20240610:080453.680 server #1 started [service manager #1]
2878125:20240610:080453.681 server #2 started [configuration syncer #1]
2878128:20240610:080456.233 server #3 started [alert manager #1]
2878129:20240610:080456.235 server #4 started [alerter #1]
2878130:20240610:080456.236 server #5 started [alerter #2]
2878131:20240610:080456.237 server #6 started [alerter #3]
2878132:20240610:080456.240 server #7 started [preprocessing manager #1]
2878133:20240610:080456.241 server #8 started [lld manager #1]
2878134:20240610:080456.242 server #9 started [lld worker #1]
2878135:20240610:080456.244 server #10 started [lld worker #2]
2878136:20240610:080456.246 server #11 started [housekeeper #1]
2878137:20240610:080456.250 server #12 started [timer #1]
2878142:20240610:080456.252 server #16 started [history syncer #1]
2878138:20240610:080456.252 server #13 started [http poller #1]
2878144:20240610:080456.253 server #17 started [history syncer #2]
2878149:20240610:080456.257 server #20 started [escalator #1]
2878145:20240610:080456.266 server #18 started [history syncer #3]
2878140:20240610:080456.267 server #14 started [browser poller #1]
2878141:20240610:080456.267 server #15 started [discovery manager #1]
2878150:20240610:080456.270 server #21 started [snmp trapper #1]
2878152:20240610:080456.271 server #22 started [proxy poller #1]
2878161:20240610:080456.271 server #30 started [poller #5]
2878154:20240610:080456.275 server #24 started [vmware collector #1]
2878158:20240610:080456.277 server #28 started [poller #3]
2878156:20240610:080456.278 server #26 started [poller #1]
2878171:20240610:080456.278 server #36 started [trapper #5]
2878159:20240610:080456.279 server #29 started [poller #4]
2878155:20240610:080456.282 server #25 started [task manager #1]
2878167:20240610:080456.291 server #35 started [trapper #4]
2878153:20240610:080456.293 server #23 started [self-monitoring #1]
2878157:20240610:080456.293 server #27 started [poller #2]
2878162:20240610:080456.293 server #31 started [unreachable poller #1]
2878183:20240610:080456.304 server #47 started [trigger housekeeper #1]
2878148:20240610:080456.310 server #19 started [history syncer #4]
2878141:20240610:080456.311 for a discovery process with 5 workers, the user limit of 1024 file descriptors is insufficient. The maximum number of concurrent checks per worker has been reduced to 122
2878164:20240610:080456.315 server #32 started [trapper #1]
2878177:20240610:080456.318 server #41 started [history poller #3]
2878181:20240610:080456.320 server #45 started [report manager #1]
2878175:20240610:080456.321 server #40 started [history poller #2]
2878180:20240610:080456.321 server #44 started [availability manager #1]
2878174:20240610:080456.323 server #39 started [history poller #1]
2878165:20240610:080456.350 server #33 started [trapper #2]
2878172:20240610:080456.357 server #37 started [icmp pinger #1]
2878173:20240610:080456.359 server #38 started [alert syncer #1]
2878178:20240610:080456.361 server #42 started [history poller #4]
2878166:20240610:080456.364 server #34 started [trapper #3]
2878132:20240610:080456.366 [1] thread started [preprocessing worker #1]
2878189:20240610:080456.368 server #50 started [agent poller #1]
2878189:20240610:080456.371 thread started
2878190:20240610:080456.371 server #51 started [snmp poller #1]
2878190:20240610:080456.375 thread started
2878179:20240610:080456.375 server #43 started [history poller #5]
2878187:20240610:080456.383 server #48 started [odbc poller #1]
2878188:20240610:080456.387 server #49 started [http agent poller #1]
2878188:20240610:080456.391 thread started
2878193:20240610:080456.393 server #53 started [internal poller #1]
2878194:20240610:080456.393 server #54 started [proxy group manager #1]
2878132:20240610:080456.394 [2] thread started [preprocessing worker #2]
2878132:20240610:080456.394 [3] thread started [preprocessing worker #3]
2878192:20240610:080456.396 server #52 started [configuration syncer worker #1]
2878182:20240610:080456.407 server #46 started [report writer #1]
2878141:20240610:080456.409 thread started [discovery worker #2]
2878141:20240610:080456.413 thread started [discovery worker #3]
2878141:20240610:080456.421 thread started [discovery worker #1]
2878141:20240610:080456.421 thread started [discovery worker #5]
2878141:20240610:080456.422 thread started [discovery worker #4]
