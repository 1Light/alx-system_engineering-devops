# Postmortem: The Case of the Phantom "p"

## Incident Overview
Upon the release of ALX's System Engineering & DevOps project 0x19 at approximately 00:07 Pacific Standard Time (PST), an outage occurred on an isolated Ubuntu 14.04 container running an Apache web server. GET requests on the server led to `500 Internal Server Error` responses, when the expected response was an HTML file defining a simple Holberton WordPress site.

## Debugging Process: A Detective Story

The issue was discovered upon opening the project and instructions were given to address it at roughly 19:20 PST. The following steps were taken to solve the mystery:

1. **The Usual Suspects**:
    - Checked running processes using `ps aux`. Two `apache2` processes - `root` and `www-data` - were properly running. Both had alibis.

2. **Clues in the Folder**:
    - Examined the `sites-available` folder in the `/etc/apache2/` directory and determined that the web server was serving content from `/var/www/html/`. So far, so good.

3. **The Strace Adventure - Root**:
    - Ran `strace` on the PID of the `root` Apache process while curling the server in another terminal. This did not yield useful information. A wild goose chase.

4. **The Strace Adventure - www-data**:
    - Repeated step 3 on the PID of the `www-data` process. This revealed an `-1 ENOENT (No such file or directory)` error upon an attempt to access the file `/var/www/html/wp-includes/class-wp-locale.phpp`. Bingo!

5. **The Typo Terror**:
    - Examined files in the `/var/www/html/` directory one-by-one using Vim pattern matching to locate the erroneous `.phpp` file extension. The typo was found in the `wp-settings.php` file (Line 137, `require_once( ABSPATH . WPINC . '/class-wp-locale.php' );`).

6. **The Fix**:
    - Removed the trailing `p` from the line.

7. **Success!**:
    - Tested another `curl` on the server, which resulted in a `200 OK` response. Victory!

8. **Automation to the Rescue**:
    - Wrote a Puppet manifest to automate fixing the error. Because who wants to deal with a phantom "p" again?

### Flowchart: Debugging Journey
```plaintext
+--------------------+
| Discover the Issue |
+---------+----------+
          |
          v
+---------+----------+
| Check Apache2      |
| Processes          |
+---------+----------+
          |
          v
+---------+----------+
| Examine Sites-     |
| Available Folder   |
+---------+----------+
          |
          v
+---------+----------+
| Run Strace on Root |
+---------+----------+
          |
          v
+---------+----------+
| Run Strace on      |
| www-data Process   |
+---------+----------+
          |
          v
+---------+----------+
| Identify Typo in   |
| wp-settings.php    |
+---------+----------+
          |
          v
+---------+----------+
| Fix Typo and Test  |
+---------+----------+
          |
          v
+---------+----------+
| Write Puppet       |
| Manifest           |
+--------------------+
```

## Summation: The Moral of the Story

In summary, the issue was caused by a pesky typo. The WordPress application encountered a critical error in `wp-settings.php` when trying to load the file `class-wp-locale.phpp`. The correct file name, located in the `wp-content` directory of the application folder, was `class-wp-locale.php`.

The patch involved a simple fix of removing the trailing `p` from the file extension.

## Prevention: Lessons Learned

This outage was an application error, not a web server error. To prevent such outages in the future, consider the following:

* **Test Thoroughly**: Testing the application before deployment would have brought this error to light earlier and allowed it to be addressed promptly. Remember, testing is your friend!
* **Implement Status Monitoring**: Enable an uptime-monitoring service such as [UptimeRobot](https://uptimerobot.com/) to provide instant alerts upon website outages.

In response to this error, a Puppet manifest [0-strace_is_your_friend.pp](https://github.com/bdbaraban/holberton-system_engineering-devops/blob/master/0x17-web_stack_debugging_3/0-strace_is_your_friend.pp) was written to automate fixing any such identical errors should they occur in the future. The manifest replaces any `phpp` extensions in the file `/var/www/html/wp-settings.php` with `php`.

While this particular error has been addressed, it serves as a reminder of the importance of diligent testing and monitoring to maintain robust and reliable systems. And remember, even a single character can make a world of difference! 🕵️‍♂️🔍

---
