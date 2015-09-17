{
  "title": "How to monitor logrotate",
  "shortTitle": "logrotate",
  "date": "2015-06-27"
}
---
Logrotate is a tool that manages activities like automatic rotation, removal and compression of log files in in Unix-like systems. We can use WDT to be notified when logrotate failed.

1. Sign up on WDT.io if you haven't already.
2. Create a new inbound timer.
3. Use a name that reminds you of the log file we want to monitor.
4. Select a schedule of **every 1 day** and a small precision like **2 minutes**.
6. Save the new timer.
7. Copy the URL of this new timer.
8. Edit the config file for your logfile under `/etc/logrotate.d/'.
9. Add or extend the postrotate section to send a kick to the URL copied from step 7.

Now every time logrotate runs, it'll also send a kick to WDT. This regular kick prevents WDT from sending an alert to you. If, for whatever reason logrotate fails, WDT won't get the kick and will send an alert.


### Example

On a server named 'www' with nginx installed, we have `/etc/logrotate.d/nginx`:

```bash
/var/log/nginx/*.log {
        daily
        missingok
        rotate 52
        compress
        delaycompress
        notifempty
        create 640 nginx adm
        sharedscripts
        postrotate
                [ -f /var/run/nginx.pid ] && kill -USR1 `cat /var/run/nginx.pid`
        endscript
}
```
Every day the log files under `/var/log/nginx/` are being rotated.

So we name our new inbound timer **www/logrotate/nginx**, set the schedule to **every 1 day** and the precision to **2 minutes**. The URL for this new timer will look something like **k.wdt.io/123abc/www/logrotate/nginx**. With that, we edit the config using `vim /etc/logrotate.d/nginx` and change the postrotate line to:

```bash
/var/log/nginx/*.log {
        daily
        missingok
        rotate 52
        compress
        delaycompress
        notifempty
        create 640 nginx adm
        sharedscripts
        postrotate
                [ -f /var/run/nginx.pid ] && kill -USR1 `cat /var/run/nginx.pid` && curl -s -m 30 http://k.wdt.io/123abc/www/logrotate/nginx
        endscript
```
Now we'll be notified when the rotation fails and can fix whatever caused the problem.

### More info

- [man page for logrotate](http://linux.die.net/man/8/logrotate)
- [man page for curl](http://linux.die.net/man/1/curl)