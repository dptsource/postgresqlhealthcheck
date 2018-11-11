#############################################################################
# Don't make any changes to this script                                     #
# File :    postgreshealthchecks.sh                                         #
# Purpose : The purpose of this script is to report Database health check   #
#                                                                           #
# History:                                                                  #
# Name                   Date                                Version        #
# ***********************************************************************   #
# PostgreSQL Health Script                                          1.0     #
#dinfratechsource                                                           #
#Dheeraj Porayil Thekkinakathu                                              #
#############################################################################
#! /bin/bash
#Checking if this script is being executed as ROOT. For maintaining proper directory structure, this script must be run from a root user.
if [ $EUID != 0 ]
then
  echo "Please run this script as root so as to see all details! Better run with sudo."
  exit 1
fi
#Declaring variables
#set -x
MYUSER=postgres
MYPASS=XXXXXXX
dte=`date`
hstname=`hostname`
ip_add=`ifconfig | grep "inet" | head -2 | awk {'print$2'}| cut -f2 -d: `
UP1=$(service postgresql status);
if [ "$?" -gt "0" ]; then
INSTSTAT=("Not Running")
else
INSTSTAT=("Running")
fi
upt=`PGPASSWORD=$MYPASS psql -U $MYUSER -c "SELECT date_trunc('second', current_timestamp - pg_postmaster_start_time()) as uptime;" | awk 'FNR == 3'`
sr_version=`psql -V | awk '{print $2;print $3}'`
max_conn=`PGPASSWORD=$MYPASS psql -U $MYUSER -c "show max_connections" | awk 'FNR== 3'`
act_conn=`PGPASSWORD=$MYPASS psql -U $MYUSER -c "select count(*) from pg_stat_activity" | awk 'FNR== 3'`
idl_conn=`PGPASSWORD=$MYPASS psql -U $MYUSER -c "SELECT COUNT (*) FROM pg_stat_activity WHERE state= 'idle';"  | awk 'FNR== 3'` 
load_avg=`cat /proc/loadavg  | awk {'print$1,$2,$3'} | sed 's/ /,/g'`
ram_usage=`free -m | head -2 | tail -1 | awk {'print$3'}`
ram_total=`free -m | head -2 | tail -1 | awk {'print$2'}`
mem_pct=`free -m | awk 'NR==2{printf "%.2f%%\t\t", $3*100/$2 }'`
mnt_pnt=`df -h | awk 'FNR==4'`
#Creating a directory if it doesn't exist to store reports first, for easy maintenance.
if [ ! -d ${HOME}/health_reports ]
then
  mkdir ${HOME}/health_reports
fi
find ${HOME}/health_reports/ -mtime +1 -exec rm {} \;
html="${HOME}/health_reports/PostgreSQL-Health-Report-`hostname`-`date +%y%m%d`-`date +%H%M`.html"
#email_add="raj.kce2912@gmail.com"
for i in `ls /home`; do sudo du -sh /home/$i/* | sort -nr | grep G; done > /tmp/dir.txt
#Generating HTML file
echo "<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">" >> $html
echo "<html>" >> $html
echo "<link rel="stylesheet" href="https://unpkg.com/purecss@0.6.2/build/pure-min.css">" >> $html
echo "<body bgcolor="#80A8C3">" >> $html
echo "<fieldset>" >> $html
echo "<center>" >> $html
echo "<h2><u>PostgreSQL Server Health Report</u></h2>" >> $html
echo "<h4><legend>Version 1.0</legend></h4>" >> $html
echo "</center>" >> $html
echo "</fieldset>" >> $html
echo "<br>" >> $html
echo "<center>" >> $html
############################################PostgreSQL Instance Details#######################################################################
echo "<h3><u>PostgreSQL Instance Details</u> </h3>" >> $html
echo "<table class="pure-table">" >> $html
echo "<thead>" >> $html
echo "<tr>" >> $html
echo "<th>Hostname</th>" >> $html
echo "<th>IP Address</th>" >> $html
echo "<th>Instance Status</th>" >> $html
echo "<th>Server Version</th>" >> $html
echo "<th>Uptime</th>" >> $html
echo "<th>Date & Time</th>" >> $html
echo "</tr>" >> $html
echo "</thead>" >> $html
echo "<tbody>" >> $html
echo "<tr>" >> $html
echo "<td>$hstname</td>" >> $html
echo "<td>$ip_add</td>" >> $html
echo "<td><font color="Red">$INSTSTAT</font></td>" >> $html
echo "<td>$sr_version</td>" >> $html
echo "<td>$upt</td>" >> $html
echo "<td>$dte</td>" >> $html
echo "</tr>" >> $html
echo "</tbody>" >> $html
echo "</table>" >> $html
############################################PostgreSQL Connection Details#######################################################################
echo "<h3><u>PostgreSQL Connection Details</u> </h3>" >> $html
echo "<table class="pure-table">" >> $html
echo "<thead>" >> $html
echo "<tr>" >> $html
echo "<th>Max Connection</th>" >> $html
echo "<th>Active Connection</th>" >> $html
echo "<th>Idle Connection</th>" >> $html
echo "</tr>" >> $html
echo "</thead>" >> $html
echo "<tbody>" >> $html
echo "<tr>" >> $html
echo "<td>$max_conn</td>" >> $html
echo "<td>$act_conn</td>" >> $html
echo "<td>$idl_conn</td>" >> $html
echo "</tr>" >> $html
echo "</tbody>" >> $html
echo "</table>" >> $html
########################################### Resource Status #######################################################################
echo "<h3><u>Resource Utilization</u> </h3>" >> $html
echo "<br>" >> $html
echo "<table class="pure-table">" >> $html
echo "<thead>" >> $html
echo "<tr>" >> $html
echo "<th>Load Average</th>" >> $html
echo "<th>Used RAM(in MB)</th>" >> $html
echo "<th>Total RAM(in MB)</th>" >> $html
echo "<th>Memory Utilization %</th>" >> $html
echo "</tr>" >> $html
echo "</thead>" >> $html
echo "<tbody>" >> $html
echo "<tr>" >> $html
echo "<td><center>$load_avg</center></td>" >> $html
echo "<td><center>$ram_usage</center></td>" >> $html
echo "<td><center>$ram_total</center></td>" >> $html
echo "<td><center>$mem_pct</center></td>" >> $html
echo "</tr>" >> $html
echo "</tbody>" >> $html
echo "</table>" >> $html
########################################### Disk Utilization #######################################################################
echo "<h3><u>Disk Utilization</u> </h3>" >> $html
echo "<table class="pure-table">" >> $html
echo "<thead>" >> $html
echo "<tr>" >> $html
echo "<th><center>Mount Point Usage</center></th>" >> $html
echo "</tr>" >> $html
echo "</thead>" >> $html
echo "<tbody>" >> $html
echo "<tr>" >> $html
echo "<td>$mnt_pnt</td>" >> $html
echo "</tr>" >> $html
echo "</tbody>" >> $html
echo "</table>" >> $html
echo "<br />" >> $html
echo "</table>" >> $html
echo "</body>" >> $html
echo "</html>" >> $html
echo "Report has been generated in ${HOME}/health_reports with file-name = $html. Report has also been sent to $email_add."
#Sending Email to the user
mailx -a $html -s "PostgreSQL Health Report" dptsource@gmail.com < /dev/null

