#### THE GOAL

    Consolidate main PostgreSQL and system health metrics in one report.
    Useful for routine daily and weekly PostgreSQL checks.

    Report provides:
    - Basic OS-level parameters: disk space, hostname,IP Address, Date & Time ,load average, memory utilization 
    - Important PostgreSQL statistics : Instance Status , Uptime, version,max_connection,threads_connected

    Report example: 

#### CONFIGURING AND RUNNING

    1. Edit PostgreSQL credentials in postgresqlhealth.sh
    2. Run ./postgresqlhealth.sh
    3. Enjoy!

    Script will automatically send report to email address given in script.
  
    Report will be emailed to you. Useful when running script via cron.

#### DOCUMENTATION

 version 1.0

#### DOWNLOAD LATEST
