Here is the ready-to-use `README.md` for today's new repo.

# cPanel LiteSpeed Website Slow Troubleshooting

A practical guide for diagnosing slow websites on cPanel servers using LiteSpeed, PHP-FPM, CloudLinux, MySQL/MariaDB, WordPress, and related server components.

This guide focuses on identifying the actual bottleneck before making configuration changes. The goal is to troubleshoot website performance systematically without applying unnecessary or destructive fixes.

---

## Table of Contents

* [Common Causes of Slow Websites](#common-causes-of-slow-websites)
* [Start With Basic Checks](#start-with-basic-checks)
* [Check Server Load](#check-server-load)
* [Check CPU Usage](#check-cpu-usage)
* [Check Memory Usage](#check-memory-usage)
* [Check Disk Usage](#check-disk-usage)
* [Check Disk I/O](#check-disk-io)
* [Check LiteSpeed](#check-litespeed)
* [Check PHP-FPM](#check-php-fpm)
* [Check PHP Workers](#check-php-workers)
* [Check MySQL/MariaDB](#check-mysqlmariadb)
* [Check CloudLinux Resources](#check-cloudlinux-resources)
* [Check WordPress](#check-wordpress)
* [Check LiteSpeed Cache](#check-litespeed-cache)
* [Check DNS and Network](#check-dns-and-network)
* [Check HTTP Response Time](#check-http-response-time)
* [Check Website Logs](#check-website-logs)
* [Check Cron Jobs](#check-cron-jobs)
* [Check External Requests](#check-external-requests)
* [Common Symptoms](#common-symptoms)
* [Safe Troubleshooting Workflow](#safe-troubleshooting-workflow)
* [Useful Commands](#useful-commands)
* [Troubleshooting Checklist](#troubleshooting-checklist)
* [Important Notes](#important-notes)

---

## Common Causes of Slow Websites

A slow website can be caused by many different layers.

Common causes include:

* High server load
* CPU saturation
* Memory pressure
* Swap usage
* High disk I/O
* Slow storage
* PHP-FPM worker exhaustion
* Too many PHP processes
* MySQL/MariaDB slow queries
* Large database tables
* WordPress plugins
* WooCommerce background tasks
* Object cache problems
* LiteSpeed Cache configuration
* CloudLinux resource limits
* Cron jobs
* External API requests
* DNS problems
* Network latency
* Large images or assets
* Poorly optimized application code
* Security scans consuming resources

Do not assume that the web server itself is responsible just because the website is slow.

---

# Start With Basic Checks

Before changing anything, collect basic server information.

Check uptime and load:

```bash
uptime
```

Check CPU and processes:

```bash
top
```

Check memory:

```bash
free -h
```

Check disk:

```bash
df -h
```

Check inode usage:

```bash
df -i
```

Check mounted filesystems:

```bash
lsblk
```

These initial checks can quickly indicate whether the problem is server-wide or isolated to a particular account.

---

# Check Server Load

Run:

```bash
uptime
```

Example:

```text
load average: 2.10, 1.85, 1.70
```

The three values represent approximately:

```text
1 minute
5 minutes
15 minutes
```

Also check:

```bash
top
```

Look for:

* CPU saturation
* High system CPU
* High I/O wait
* Memory pressure
* Large numbers of PHP processes
* MySQL activity
* LiteSpeed processes
* Unexpected processes

Load should be interpreted relative to the number of CPU cores and the type of workload.

---

# Check CPU Usage

Use:

```bash
top
```

For a more focused process list:

```bash
ps aux --sort=-%cpu | head -20
```

This can identify processes consuming significant CPU.

Common causes include:

* PHP scripts
* WordPress requests
* MySQL queries
* Backup processes
* Malware/security scans
* Cron jobs
* Image processing
* Search/indexing tasks

Do not immediately terminate a process simply because it has high CPU usage. First determine what the process is doing.

---

# Check Memory Usage

Run:

```bash
free -h
```

Example:

```text
               total        used        free
Mem:            16Gi        11Gi        1.2Gi
Swap:            2Gi        500Mi        1.5Gi
```

Also check:

```bash
ps aux --sort=-%mem | head -20
```

Look for:

* PHP-FPM
* MySQL/MariaDB
* LiteSpeed
* ClamAV
* Imunify
* Backup processes
* Large application processes

If memory is exhausted, the kernel may terminate processes through the OOM mechanism.

Check:

```bash
dmesg | grep -i -E 'oom|out of memory|killed process'
```

---

# Check Disk Usage

Run:

```bash
df -h
```

Then:

```bash
df -i
```

A filesystem can become problematic because of either:

* Disk space exhaustion
* Inode exhaustion

Find large directories:

```bash
du -xhd1 / | sort -h
```

For `/home`:

```bash
du -xhd1 /home | sort -h
```

For `/var`:

```bash
du -xhd1 /var | sort -h
```

High disk usage can affect database operations, logging, caching, temporary files, and application performance.

---

# Check Disk I/O

If CPU usage is not particularly high but the website is still slow, investigate I/O.

Check:

```bash
iostat -xz 1 5
```

If `iostat` is unavailable, install the appropriate system package rather than assuming the command exists.

You can also inspect processes with:

```bash
iotop
```

Look for:

* High `%util`
* High await time
* Large read/write activity
* Database writes
* Backup activity
* Log activity
* Security scans

High I/O wait can make a website appear slow even when CPU usage is relatively low.

---

# Check LiteSpeed

Check LiteSpeed service status:

```bash
systemctl status lsws
```

Restart only when necessary:

```bash
systemctl restart lsws
```

Check the service after restarting:

```bash
systemctl status lsws
```

Check LiteSpeed processes:

```bash
ps aux | grep -i litespeed
```

Check listening ports:

```bash
ss -lntp | grep -E ':80|:443'
```

If LiteSpeed is running normally, continue investigating PHP, database, application, or resource limits instead of assuming LiteSpeed is the problem.

---

# Check LiteSpeed Logs

Depending on the installation, LiteSpeed logs may be available under:

```text
/usr/local/lsws/logs/
```

List the directory:

```bash
ls -lah /usr/local/lsws/logs/
```

Check the error log:

```bash
tail -100 /usr/local/lsws/logs/error.log
```

For a specific virtual host, check the site's configured log location.

Look for:

* Connection errors
* PHP errors
* Backend timeouts
* Proxy errors
* Worker problems
* Permission problems
* Repeated application failures

Repeated errors are often more useful than isolated errors.

---

# Check PHP-FPM

PHP-FPM is a common bottleneck for WordPress and WooCommerce sites.

Check PHP-FPM processes:

```bash
ps aux | grep php-fpm
```

Count PHP-FPM processes:

```bash
ps -C php-fpm --no-headers | wc -l
```

The actual process name may vary depending on the PHP installation.

On cPanel systems with multiple PHP versions, identify the relevant PHP version before changing configuration.

---

# Check PHP Workers

A website can become slow when PHP workers are exhausted.

Look for:

```text
pm.max_children
pm.max_requests
pm.start_servers
pm.min_spare_servers
pm.max_spare_servers
```

The exact configuration depends on the PHP-FPM pool.

Check the active PHP-FPM configuration before changing values.

Increasing workers without considering available RAM can make the server slower or cause memory exhaustion.

The correct approach is:

```text
Check RAM
    ↓
Check PHP process memory
    ↓
Check current worker usage
    ↓
Check request volume
    ↓
Adjust carefully if required
```

---

# Check MySQL/MariaDB

Database performance is frequently involved in slow WordPress and WooCommerce websites.

Check the database service:

```bash
systemctl status mariadb
```

or:

```bash
systemctl status mysql
```

Check active processes:

```bash
mysqladmin processlist
```

If authentication requires credentials, use the appropriate administrative authentication method for the server.

You can also enter MySQL:

```bash
mysql
```

Then:

```sql
SHOW PROCESSLIST;
```

Look for:

* Long-running queries
* Locked tables
* Waiting queries
* Repeated queries
* Large numbers of connections

---

# Check Database Size

For MySQL/MariaDB:

```sql
SELECT
    table_schema AS database_name,
    ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS size_mb
FROM information_schema.tables
GROUP BY table_schema
ORDER BY size_mb DESC;
```

Large databases are not automatically bad, but large tables and inefficient queries can affect performance.

For WordPress, investigate tables related to:

* Post metadata
* Options
* WooCommerce sessions
* Action Scheduler
* Logs
* Plugin-specific data

Do not delete database tables simply because they are large.

---

# Check Slow Queries

If slow-query logging is enabled, inspect the configured slow-query log.

Check MariaDB configuration:

```bash
grep -Rni "slow_query" /etc/my.cnf /etc/my.cnf.d/ 2>/dev/null
```

Check for active configuration:

```bash
mysql -e "SHOW VARIABLES LIKE 'slow_query%';"
```

Also:

```bash
mysql -e "SHOW VARIABLES LIKE 'long_query_time';"
```

Slow-query logging should be used carefully because logging itself can generate additional I/O.

---

# Check CloudLinux Resources

On CloudLinux servers, an individual account can become slow because it has reached its resource limits even when the entire server looks healthy.

Common CloudLinux limits include:

* CPU
* RAM
* IO
* EP
* NPROC
* Number of processes
* Entry Processes

Check the affected account through CloudLinux tools or the cPanel/WHM interface.

The important distinction is:

```text
Server overloaded
```

versus:

```text
One account reached its resource limits
```

These require different troubleshooting approaches.

---

# Check Entry Processes

Entry Processes (EP) are particularly important for websites receiving many simultaneous dynamic requests.

A site can become slow if requests are waiting because available entry processes are exhausted.

Investigate:

* Traffic spikes
* WordPress admin requests
* WooCommerce requests
* Bots
* Slow PHP requests
* External API calls
* Long-running PHP scripts

Do not simply increase EP without finding why requests are taking too long.

---

# Check WordPress

For WordPress websites, first identify whether the problem affects:

* Frontend
* Admin dashboard
* Login
* WooCommerce pages
* Checkout
* Cart
* Product pages
* Search
* AJAX requests

If WP-CLI is available:

```bash
wp plugin list
```

Check WordPress version:

```bash
wp core version
```

Check active theme:

```bash
wp theme list
```

Check cron:

```bash
wp cron event list
```

Use:

```bash
wp profile stage --all
```

only if the WP-CLI profile command is available.

A slow WordPress site should be investigated at the PHP, database, plugin, theme, cache, and server layers.

---

# Check LiteSpeed Cache

If LiteSpeed Cache is installed, check whether caching is actually working.

For WordPress:

```bash
wp plugin list | grep -i litespeed
```

Review:

* Page cache
* Object cache
* Browser cache
* Guest mode
* Guest optimization
* Cache exclusions
* Logged-in user behavior

Do not change multiple cache settings simultaneously.

After a configuration change, test:

1. Anonymous frontend
2. Logged-in frontend
3. Product page
4. Cart
5. Checkout
6. WordPress admin

WooCommerce pages should be handled carefully because dynamic pages cannot always be cached in the same way as normal content pages.

---

# Check Object Cache

If Redis or another object cache is being used, verify that it is actually available and connected.

Check Redis:

```bash
systemctl status redis
```

or:

```bash
systemctl status redis-server
```

Test:

```bash
redis-cli ping
```

Expected response:

```text
PONG
```

A broken object-cache configuration can sometimes create additional errors instead of improving performance.

---

# Check DNS and Network

Check DNS:

```bash
dig example.com
```

Check response time:

```bash
dig example.com
```

Test HTTP:

```bash
curl -I https://example.com
```

Test connection timing:

```bash
curl -o /dev/null -s -w \
'DNS: %{time_namelookup}\nConnect: %{time_connect}\nTLS: %{time_appconnect}\nTTFB: %{time_starttransfer}\nTotal: %{time_total}\n' \
https://example.com/
```

This helps separate:

* DNS delay
* TCP connection delay
* TLS negotiation
* Server processing time
* Total response time

---

# Check HTTP Response Time

Use:

```bash
curl -I https://example.com
```

For more detailed timing:

```bash
curl -o /dev/null -s -w \
'TTFB: %{time_starttransfer}\nTotal: %{time_total}\n' \
https://example.com/
```

Run the test multiple times.

One slow request does not necessarily indicate a server problem.

Compare:

```text
Homepage
Product page
Static file
WordPress admin
API endpoint
```

This can help identify whether the problem is application-specific.

---

# Check Website Logs

Logs are often the fastest way to identify application problems.

Check cPanel log directory:

```bash
ls -lah /usr/local/cpanel/logs/
```

Check cPanel errors:

```bash
tail -100 /usr/local/cpanel/logs/error_log
```

Check LiteSpeed logs:

```bash
tail -100 /usr/local/lsws/logs/error.log
```

For a specific website, identify its configured access and error logs.

Look for:

* PHP fatal errors
* 500 errors
* 502 errors
* 503 errors
* Timeouts
* Repeated requests
* Missing files
* Permission errors
* Database errors

---

# Check Cron Jobs

Background tasks can consume CPU, RAM, database connections, and disk I/O.

List system cron jobs:

```bash
crontab -l
```

Check system cron directories:

```bash
ls -lah /etc/cron.d/
ls -lah /etc/cron.daily/
ls -lah /etc/cron.hourly/
```

For WordPress:

```bash
wp cron event list
```

Look for:

* Very frequent tasks
* Failed tasks
* Large queues
* Repeated plugin tasks
* WooCommerce Action Scheduler tasks
* Backup jobs
* Security scans

---

# Check External Requests

A website may appear slow because PHP is waiting for another service.

Examples:

* Payment gateway
* Shipping API
* Currency API
* Remote image service
* Marketing API
* Social API
* External authentication
* License verification

Symptoms may include:

* Slow TTFB
* PHP workers remaining busy
* Normal CPU usage
* Normal database performance

Application-level profiling may be necessary to identify the exact external request.

---

# Check for Traffic Spikes

A sudden increase in requests can make a previously healthy website slow.

Check active connections:

```bash
ss -Htan | wc -l
```

Check established connections:

```bash
ss -Htan state established | wc -l
```

Inspect connections to HTTP/HTTPS:

```bash
ss -Htan '( sport = :80 or sport = :443 )' | head
```

For web-server access logs, look for:

* Repeated IP addresses
* Repeated URLs
* Login attempts
* XML-RPC requests
* `wp-login.php`
* `wp-cron.php`
* Search requests
* Expensive API endpoints

High traffic should be analyzed before applying blocking rules.

---

# Common Symptoms

| Symptom                                     | Possible Cause                                    |
| ------------------------------------------- | ------------------------------------------------- |
| Entire server is slow                       | CPU, RAM, I/O, network, or system-wide workload   |
| One website is slow                         | Account/application/resource issue                |
| TTFB is high                                | PHP, database, external API, or server processing |
| Static files are fast but WordPress is slow | PHP/database/application                          |
| Admin is slow                               | Plugin, database, cron, or PHP                    |
| WooCommerce checkout is slow                | PHP, database, payment/shipping API               |
| CPU is high                                 | PHP, MySQL, cron, traffic, scans                  |
| RAM is exhausted                            | PHP workers, MySQL, applications, processes       |
| High I/O wait                               | Storage, database, backups, logs                  |
| EP limit reached                            | Too many simultaneous dynamic requests            |
| CPU limit reached                           | Account/application workload                      |
| MySQL processes are waiting                 | Slow queries or locks                             |
| Website slows during backups                | Backup I/O/resource contention                    |
| Website slows periodically                  | Cron/background jobs                              |

---

# Server-Wide vs Account-Specific

One of the most important troubleshooting steps is determining the scope.

### Server-wide problem

Check:

```bash
uptime
free -h
df -h
iostat -xz 1 5
top
```

If several websites are slow at the same time, investigate the server.

### Account-specific problem

If only one account is affected, inspect:

* CloudLinux limits
* PHP-FPM
* Website logs
* Database
* WordPress plugins
* Theme
* Cron
* Traffic
* Cache

Do not modify global server settings for a problem isolated to one account unless there is evidence that the global configuration is responsible.

---

# Safe Troubleshooting Workflow

Use the following sequence.

```text
1. Confirm the affected website
          ↓
2. Determine whether the issue is intermittent or constant
          ↓
3. Check server load
          ↓
4. Check CPU and RAM
          ↓
5. Check disk and I/O
          ↓
6. Check CloudLinux account limits
          ↓
7. Check LiteSpeed
          ↓
8. Check PHP-FPM
          ↓
9. Check MySQL/MariaDB
          ↓
10. Check website logs
          ↓
11. Check WordPress/plugins/theme
          ↓
12. Check cache
          ↓
13. Check DNS/network
          ↓
14. Measure response time
          ↓
15. Apply one targeted fix
          ↓
16. Test again
```

---

# Useful Commands

### Server load

```bash
uptime
top
```

### CPU

```bash
ps aux --sort=-%cpu | head -20
```

### Memory

```bash
free -h
ps aux --sort=-%mem | head -20
```

### Disk

```bash
df -h
df -i
```

### I/O

```bash
iostat -xz 1 5
```

### Network connections

```bash
ss -Htan | wc -l
```

### LiteSpeed

```bash
systemctl status lsws
```

### PHP processes

```bash
ps aux | grep php-fpm
```

### MariaDB

```bash
systemctl status mariadb
```

### DNS

```bash
dig example.com
```

### HTTP

```bash
curl -I https://example.com
```

### Detailed HTTP timing

```bash
curl -o /dev/null -s -w \
'DNS: %{time_namelookup}\nConnect: %{time_connect}\nTLS: %{time_appconnect}\nTTFB: %{time_starttransfer}\nTotal: %{time_total}\n' \
https://example.com/
```

---

# Troubleshooting Checklist

## Server

* [ ] Check uptime and load
* [ ] Check CPU
* [ ] Check RAM
* [ ] Check swap
* [ ] Check disk usage
* [ ] Check inode usage
* [ ] Check I/O wait

## cPanel/CloudLinux

* [ ] Check account resource limits
* [ ] Check EP
* [ ] Check CPU limit
* [ ] Check memory limit
* [ ] Check I/O limit
* [ ] Check NPROC

## LiteSpeed

* [ ] Verify LiteSpeed is running
* [ ] Check LiteSpeed logs
* [ ] Check listening ports
* [ ] Check virtual-host errors

## PHP

* [ ] Check PHP-FPM
* [ ] Check worker usage
* [ ] Check PHP errors
* [ ] Check PHP version
* [ ] Check long-running requests

## Database

* [ ] Check MariaDB status
* [ ] Check active queries
* [ ] Check long-running queries
* [ ] Check database size
* [ ] Check locks
* [ ] Check slow queries

## WordPress

* [ ] Check plugins
* [ ] Check theme
* [ ] Check cron
* [ ] Check WooCommerce tasks
* [ ] Check object cache
* [ ] Check page cache
* [ ] Check application errors

## Network

* [ ] Check DNS
* [ ] Check HTTP response
* [ ] Check TTFB
* [ ] Check external API requests
* [ ] Check traffic spikes

---

# Important Notes

### Do not blindly increase limits

Increasing:

```text
PHP workers
CPU limits
RAM limits
Entry Processes
MySQL connections
```

does not automatically solve performance problems.

Higher limits can increase resource consumption and make an overloaded server worse.

---

### Do not restart everything at once

Avoid running a sequence such as:

```bash
systemctl restart lsws
systemctl restart mariadb
systemctl restart php-fpm
```

without identifying the problem.

Restarting services can temporarily hide the underlying issue.

---

### Do not delete cache or database data blindly

Before cleaning:

* Identify what the data is
* Verify whether it is safe to remove
* Take a backup when appropriate
* Use the application's supported cleanup method

---

# Final Principle

A slow website should be treated as a **diagnostic problem**, not simply a configuration problem.

Start by determining:

```text
Is the server slow?
        ↓
Is the account limited?
        ↓
Is LiteSpeed slow?
        ↓
Is PHP slow?
        ↓
Is MySQL slow?
        ↓
Is WordPress slow?
        ↓
Is an external service slow?
        ↓
Is the network/DNS slow?
```

Measure first, identify the bottleneck, make one targeted change, and measure again.

---

## Disclaimer

This guide is intended for system administrators, hosting providers, developers, and server operators.

Commands and configuration paths can vary depending on the operating system, cPanel/WHM version, CloudLinux configuration, LiteSpeed installation, PHP version, database configuration, and hosting environment.

Always verify commands and configuration before applying changes to a production server.
