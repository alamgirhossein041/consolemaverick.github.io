---
title: "Policing applications with long running queries"
layout: post
subtitle: "Why won't my process top 😭"
description: "If you are faced with an application that experiences long running queries and are unable to terminate it via your monitoring tool here is what you do."
image: /assets/img/uploads/62l6qe.jpg
date: '2022-01-25 07:28:05 +0900'
optimized_image: /assets/img/uploads/62l6qe.jpg
category: fundementals
tags:
  - linux
  - start-stop-daemon
  - shell
author: Pawel Stoklosa
---

If you are faced with an application that experiences long running queries and are unable to terminate it via your monitoring tool here is what you do.

In order to diagnose the problem look for this in your syslog

```
[CEST Jul  9 03:10:21] error    : ‘application_puma' failed to stop (exit status -1)
```

In this case we are using monit to monitor the application and it's complaining that it failed to stop the process. The culprit was a long running query which prevents the stop command issued by monit from killing the process.

The stop command was using the builtin puma stop command:

```
bundle exec pumactl -S $STATE_FILE stop
```

Unix philosophy dictates using purpose built tools in unison for the desired effect, piping the output of one into another. Processes like puma and sidekiq are doing away with daemonization support. Builtin daemonization support is contrary to the Unix philosophy. Instead a dedicated tool should be used for daemonization. One of those tools is the start-stop-daemon.

An additional benefit of this tool is the ability to more aggressively terminate processes. The start-stop-daemon can issue increasingly aggressive terminte command like so:

```
start-stop-daemon --stop --quiet --retry=TERM/30/KILL/5 --pidfile $PID\_DIR/$NAME.pid
```

This will send SIGTERM to that process, wait 30 seconds, then SIGKILL and wait 5 seconds. The SIGKILL is the equivalent of kill -9.
Putting this all together into a single script for a hypothetical Ruby on Rails application leveraging puma looks like this:

```
case "$1" in
  start)
    echo "Starting puma"
    start-stop-daemon --start --pidfile $PID_FILE --make-pidfile --background --chdir $DEPLOY_TO/current --startas /bin/bash -- -c "bundle exec puma -e $RAILS_ENV -b $TCP -S $STATE_FILE --control-url $CONTROL_SOCKET --redirect-stderr  $DEPLOY_TO/shared/log/puma_err.log"
    echo "Done!"
    ;;
  stop)
    echo "Stopping puma"
    start-stop-daemon --stop --quiet --retry=TERM/30/KILL/5 --pidfile $PID_FILE
    echo "Done"
    ;;
  *)
    echo "Usage: puma {start|stop} {server_index}"
    exit 1
    ;;
esac
```

Starting uses the start-stop-daemon and so does the stopping. puma does only 1 job: serving the application. start-stop-daemon does only 1 job, daemonizing and the 2 tools work in unison.
