---
layout: post
title: SystemChangeTracker
subtitle: Get Email Alerts When Your System Files are Changed
gh-repo: lleevveell66/lleevveell66.github.io
gh-badge: [star, fork, follow]
tags: [cron, script, blue team, alerting]
---

## Installation

```
cat <<'EOF'>/etc/cron.hourly/CheckForSystemChanges
#!/bin/sh

EMAIL_FROM=SysChangeTracker@myhost.com
EMAIL_TO=alerts@my-alert-host.com
EMAIL_FILE=/tmp/SysChangeTracker.email

OLDREPORT=/tmp/SysChangeTracker.old
NEWREPORT=/tmp/SysChangeTracker.new
DIFFREPORT=/tmp/SysChangeTracker.diff

function send_alert()
{
 HOSTNAME=$(/usr/bin/hostname)
 DATETIME=$(/usr/bin/date -Is)

 /usr/bin/echo "From: $EMAIL_FROM" > $EMAIL_FILE
 /usr/bin/echo "To: $EMAIL_TO" >> $EMAIL_FILE
 /usr/bin/echo "Subject: $HOSTNAME System Changes Detected" >> $EMAIL_FILE
 /usr/bin/echo "MIME-Version: 1.0" >> $EMAIL_FILE
 /usr/bin/echo "Content-Type: text/html" >> $EMAIL_FILE
 /usr/bin/echo "Content-Disposition: inline" >> $EMAIL_FILE
 /usr/bin/echo "<html>" >> $EMAIL_FILE
 /usr/bin/echo "<body>" >> $EMAIL_FILE
 /usr/bin/echo "<pre style="font: monospace">" >> $EMAIL_FILE
 /usr/bin/echo "" >> $EMAIL_FILE
 /usr/bin/echo "The following system file differences where found on $HOSTNAME at $DATETIME:" >> $EMAIL_FILE
 /usr/bin/echo "----------------------------------------------------------------------------" >> $EMAIL_FILE
 /usr/bin/echo "" >> $EMAIL_FILE
 /bin/cat $DIFFREPORT >> $EMAIL_FILE
 /usr/bin/echo "" >> $EMAIL_FILE
 /usr/bin/echo "----------------------------------------------------------------------------" >> $EMAIL_FILE
 /usr/bin/echo "" >> $EMAIL_FILE
 /usr/bin/echo "# SM5DLUGT [c|g] file" >> $EMAIL_FILE
 /usr/bin/echo "# ||||||||  | |  +- file which failed verification" >> $EMAIL_FILE
 /usr/bin/echo "# ||||||||  | +---- ghost" >> $EMAIL_FILE
 /usr/bin/echo "# ||||||||  +------ config file" >> $EMAIL_FILE
 /usr/bin/echo "# |||||||+--------- modification time" >> $EMAIL_FILE
 /usr/bin/echo "# ||||||+---------- group" >> $EMAIL_FILE
 /usr/bin/echo "# |||||+----------- owner" >> $EMAIL_FILE
 /usr/bin/echo "# ||||+------------ symbolic link contents" >> $EMAIL_FILE
 /usr/bin/echo "# |||+------------- major and minor numbers" >> $EMAIL_FILE
 /usr/bin/echo "# ||+-------------- MD5 checksum" >> $EMAIL_FILE
 /usr/bin/echo "# |+--------------- mode" >> $EMAIL_FILE
 /usr/bin/echo "# +---------------- size" >> $EMAIL_FILE
 /usr/bin/echo "</pre></body></html>" >> $EMAIL_FILE

 /bin/cat $EMAIL_FILE | /usr/sbin/sendmail $EMAIL_TO
 logger -t "SysChangeTracker" "WARNING! System configuration differences were found! See $EMAIL_FILE for details."
}

/bin/cp -f $NEWREPORT $OLDREPORT

/usr/bin/rpm -Va > $NEWREPORT

/usr/bin/diff $OLDREPORT $NEWREPORT > $DIFFREPORT

LINES=$(/bin/cat $DIFFREPORT | /usr/bin/wc -l)

if (( $LINES > 0 ));then
  send_alert
fi

exit 0

EOF

chmod 755 /etc/cron.hourly/CheckForSystemChanges
/usr/bin/rpm -Va > /tmp/SysChangeTracker.new
```

Now, edit /etc/cron.hourly/CheckForSystemChanges and change the EMAIL_FROM and EMAIL_TO values to suit your needs.
