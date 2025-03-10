---
layout: article
tags: Beaker
title: restraint
mathjax: true
key: Linux
---

## Restraint
```
https://restraint.readthedocs.io/en/latest/commands.html

restraint:
# cat /var/lib/restraint/rstrnt-commands-env-8081.sh 
HARNESS_PREFIX=RSTRNT_
RSTRNT_URL=http://localhost:8081
RSTRNT_RECIPE_URL=http://localhost:8081/recipes/16343105
RSTRNT_TASKID=179082209

# cat /usr/lib/systemd/system/restraintd.service
[Unit]
Description=The restraint harness.
After=network-online.target time-sync.target
Requires=network-online.target

[Service]
Type=simple
StandardError=journal+console
ExecStartPre=/usr/bin/check_beaker
ExecStart=/usr/bin/restraintd --port 8081
KillMode=process
OOMScoreAdjust=-1000
OOMPolicy=continue

[Install]
WantedBy=multi-user.target

# cat /var/lib/restraint/config.conf 
[restraint]
recipe_url=http://lab-01.rhts.eng.pek2.redhat.com:8000//recipes/16343105/

[offsets_179082203]
logs/harness.log=144568

logs/taskout.log=109

[offsets_179082204]
logs/harness.log=3983064

logs/taskout.log=109

[offsets_179082205]
logs/harness.log=2722

logs/taskout.log=409

[offsets_179082206]
logs/taskout.log=3158
logs/harness.log=3245

[offsets_179082207]
logs/harness.log=102265

logs/taskout.log=28475

[offsets_179082208]
logs/taskout.log=73468
logs/harness.log=4669

[179082209]
started=true
reboots=1

remaining_time=15600

[offsets_179082209]
logs/taskout.log=90921
logs/harness.log=6293

# ll /usr/share/restraint/plugins/
total 12
drwxr-xr-x 2 root root   78 Jun 13 23:04 completed.d
-rwxr-xr-x 1 root root  639 Jun  3 03:42 helpers
drwxr-xr-x 2 root root   65 Jun 13 21:57 localwatchdog.d
drwxr-xr-x 2 root root   90 Jun 13 21:57 report_result.d
-rwxr-xr-x 1 root root 1105 Jun  3 03:42 run_plugins
-rwxr-xr-x 1 root root  430 Jun  3 03:42 run_task_plugins
drwxr-xr-x 2 root root  173 Jun 13 21:58 task_run.d
```
