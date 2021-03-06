---
author:
  name: Linode
  email: skleinman@linode.com
description: 'Use the "logrotate" tool to manage logfiles.'
keywords: 'logrotate,log rotation,log files,access logs,linux systems administration,basic systems administration'
license: '[CC BY-ND 3.0](http://creativecommons.org/licenses/by-nd/3.0/us/)'
alias: ['linux-tools/utilities/logrotate/']
modified: Tuesday, May 17th, 2011
modified_by:
  name: Amanda Folson
published: 'Monday, October 11th, 2010'
title: Use logrotate to Manage Log Files
---

`logrotate` is a tool for managing log files created by system processes. This tool automatically compresses and removes logs to maximize the convenience of logs and conserve system resources, and allows users extensive control over how log rotation is processed.

Using Log Rotate
----------------

`logrotate`'s behavior is determined by options set in a configuration file, typically located at `/etc/logrotate.conf`.

    logrotate /etc/logrotate.conf

Beyond the system wide log rotation configuration, you may run `logrotate` in a per-user setup with distinct configuration files if your deployment requires that non-privileged users be able to rotate their own logs.

### Running Log Rotate

Running `logrotate` as a cronjob ensures that logs will be rotated regularly as configured. Logs will only be rotated when `logrotate` runs, regardless of configuration. For example, if you configure `logrotate` to rotate logs every day but `logrotate` only runs every week, the logs will only be rotated every week.

For most daemon processes, logs should be rotated by the root user. In most cases, `logrotate` is invoked from a script in the `/etc/cron.daily/` directory. If one does not exist, create a script that resembles the following in the `/etc/cron.daily/` folder:

{: .file }
/etc/cron.daily/logrotate
:   ~~~ bash
    #!/bin/sh
    logrotate /etc/logrotate.conf
    ~~~

You may also use an entry in the root user's `crontab`.

### Understanding logrotate.conf

The configuration file for log rotation begins with a number global directives that control how log rotation is applied globally. Most configuration of log rotation does not occur in the `/etc/logrotate.conf` file, but rather in files located in the `/etc/logrotate.d` directory. Every daemon process or log file will have its own file for configuration in this directory. The `/etc/logrotate.d` configurations are loaded with the following directive in `logrotate.conf`

{: .file-excerpt }
logrotate.conf

> include /etc/logrotate.d

Configuration settings for rotation of specific logs is instantiated in a block structure:

{: .file-excerpt }
logrotate.conf

> /var/log/mail.log { weekly rotate 5 compress compresscmd xz create 0644 postfix postfix

> }

The size and rotation of `/var/log/mail.log` is managed according to the directives instantiated between the braces. The above configuration rotates logs every week, saves the last five rotated logs, compresses all of the old log files with the `xz` compression tool, and recreates the log files with permissions of 0644\` and `postfix` as the user and group owner. These specific configuration options override global configuration options which are described below.

Configure Log Rotation
----------------------

{: .file-excerpt }
logrotate configuration

> rotate 4

The `rotate` directive controls how many times a log is rotated before old logs are removed. If you specify a rotation number of `0`, logs will be removed immediately after they are rotated. If you specify an email address using the `mail` directive as file, logs are emailed and removed.

{: .file-excerpt }
logrotate configuration

> mail <squire@example.com>

Your system will need a functioning [MTA](/docs/email/) to be able to send email.

Configure Rotation Intervals
----------------------------

To rotate logs every week, set the following configuration directive:

{: .file-excerpt }
logrotate configuration

> weekly

When `weekly` is set, logs are rotated if the current week day is lower than the week day of the last rotation (i.e. Monday is less than Friday) or if the last rotation occurred more than a week before the present.

To configure monthly log rotation, use the following directive:

{: .file-excerpt }
logrotate configuration

> monthly

Logs with this value set will rotate every month on the first day of the month that `logrotate` runs, which is often the first day of the month.

For annual rotation:

{: .file-excerpt }
logrotate configuration

> yearly

Logs are rotated when the current year differs from the date of the last rotation.

To rotate based on size, use the following directive:

{: .file-excerpt }
logrotate configuration

> size [value]

The `size` directive forces log rotation when a log file grows bigger than the specified `[value]`. By default, `[value]` is assumed to be in bytes. Append a `k` to `[value]` to specify a size in kilobytes, or use `M` or `G` for megabytes or gigabytes.

Configure Log Compression
-------------------------

{: .file-excerpt }
logrotate configuration

> compress

The `compress` directive compresses all logs after they have been rotated. If this directive is placed in the global configuration, all logs will be compressed. If you want to disable a globally enabled compression directive for a specific log, use the `nocompress` directive.

{: .file-excerpt }
logrotate configuration

> compresscmd xz

By default, `logrotate` compresses files using the `gzip` command. You can replace this with another compression tool such as `bzip2` or `xz` as an argument to the `compresscmd` directive.

Delay Log File Compression
--------------------------

{: .file-excerpt }
logrotate configuration

> delaycompress

In some situations it is not ideal to compress a log file immediately after rotation when the log file needs additional processing. The `delaycompress` directive above postpones the compression one rotation cycle.

Maintain Log File Extension
---------------------------

In typical operation, `logrotate` will append a number to a file name so the `access.log` file would be rotated to `access.log.1`. To ensure that an extension is maintained, use the following directive.

{: .file-excerpt }
logrotate configuration

> extension log

This ensures that `access.log` will be rotated to `access.1.log`. If you enable compression, the compressed log will be located at `access.1.log.gz`.

Control Log File Permissions
----------------------------

If your daemon process requires that a log file exist to function properly, `logrotate` may interfere when it rotates logs. As a result, it is possible to have `logrotate` create new empty log files after rotation. Consider the following example:

{: .file-excerpt }
logrotate configuration

> create 640 www-data users

In this example, a blank file is created with the permissions `640` (owner read/write, group read, other none) owned by the user `www-data` and in the `users` group. This directive specifies options in the form: `create [mode(octal)] [owner] [group]`.

Running Commands
----------------

`logrotate` can run commands before and after rotation to ensure that routine tasks associated with log ration are run such as restarting or reloading daemons and passing other kinds of signals. To run a command before rotation begins, use a directive similar to the following:

{: .file-excerpt }
logrotate configuration

> prerotate
> :   touch /srv/www/example.com/application/tmp/stop
>
> endscript

The command `touch /srv/www/example.com/application/tmp/stop` runs before rotating the logs. Ensure that there are no errant directives or commands on the lines that contain `prerotate` and `endscript`. Also be aware that all lines *between* these directives will be executed. To run a command or set of commands after log rotation, consider the following example:

{: .file-excerpt }
logrotate configuration

> postrotate
> :   touch /srv/www/example.com/application/tmp/start
>
> endscript

`postrotate` is operationally and functionally identical to `prerotate` except that the commands are run after log rotation.



