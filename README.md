# zabbix_one-liners

Active Monitoring.
"Active" means when Zabbix server received some data from monitored hosts but not initiate these checks.
Apache access log. Monitor HTTP status codes
A script running from cron every minute. It calculates offset and  store it in file. Every time called tail processed only last portion of access.log which is good for performance. Also we use system unilities which improve parsing speed to 0.7 sec per 100MB (grep -E takes 3 sec). Comparing with zbal scripts this one calculates statistics all at one shot and agent don't have to run it again for each type of HTTP status code.
Log rotation friendly. It recognises that log has been rotated and reset offset to zero.
~# crontab -l
ACCESSLOG="/var/log/httpd/access_log"
OFFSET="/var/lib/zabbix/offset"
* * * * * tail -c  +$(K=`cat $OFFSET || echo 0`; [ $K -gt `stat -c \%s $ACCESSLOG| tee $OFFSET` ] && echo 0 || echo $K) $ACCESSLOG |cut -d' ' -f9| cut -c1| sort |uniq -c| while read n c; do if [ -n "$c" ]; then zabbix_sender -z zabbix_host -s $(hostname -f) -k active.webcode[${c}XX] -o $n; fi; done
(Note) Some implementations of CRON doesn't understand variables. So just change them to real files and paths. You can check what is going on by checking /var/log/cron
In Zabbix create items as type: zabbix trapper, key:  active.webcode[2XX] etc.; type: numeric
Create triggers:
Create graphs:
 
Monitor MySQL database status.
This script will send all the MySQL status variables (its a WIP so might be non functioning).
#!/bin/bash 
echo "show global status" | mysql -N -uroot -pMYSQL_PASSWORD | awk -F'\t' '{ printf("- "); printf("mysql.status[\"%s\"]",$1); printf(" \"%s\"",$2); print ""; }' | zabbix_sender -c /etc/zabbix/zabbix_agentd.conf -i - | grep sent | awk '{ print $2 }' | sed 's/;$//' echo "show variables" | mysql -N -pMYSQL_PASSWORD | awk -F'\t' '{ printf("- "); printf("mysql.var[\"%s\"]",$1); printf(" \"%s\"",$2); print ""; }' | zabbix_sender -c /etc/zabbix/zabbix_agentd.conf -vv -i -
 
This script uses zabbix_sender to send the values for all mysql variables and status variables to the monitoring server, it is then possible to create items to receive this data and triggers to act upon it. Additional data that does not have a matching key will be discarded.
This script should be run on a cron, or even via a system.run item (or UserParameter). In which case you can also monitor its status.
