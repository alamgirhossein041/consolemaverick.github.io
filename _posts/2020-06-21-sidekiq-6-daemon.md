---
title: Running Sidekiq 6 as a Daemon
layout: post
subtitle: via start-stop-daemon
description: Fixing sidekiq after upgrading from 5
image: /assets/img/uploads/sidekiq.png
date: '2020-06-21 13:28:05 +0900'
optimized_image: /assets/img/uploads/sidekiq.png
category: fundementals
tags:
  - sidekiq
  - daemon
  - linux
  - background jobs
author: Pawel
---

You may have noticed after upgrading to Sidekiq 6 that your background jobs no longer run.

You peruse the error log to find

```
failed to start (exit status 0) -- '/var/www/{env}/{app_name}/shared/scripts/sidekiq.sh start 1': invalid option: --index
```

When you look at (Github)[https://github.com/mperham/sidekiq/blob/master/6.0-Upgrade.md] you see that daemonization has been removed from Sidekiq 6 so it's time to find an alternative method of keeping the sidekiq process running.

You no longer can do something like this

```
bundle exec sidekiq --index 0 --pidfile /tmp/pids/sidekiq.pid --environment {env} --logfile /var/log/sidekiq.log --queue notification --concurrency 10 --daemon
```

So how do you get monit running as a background process and make sure it stays up?

First create a `sidekiq.sq` script inside your application folder `/var/www/{env}/{app_name}/shared/scripts/sidekiq.sh`

Inside if define the following:

```
#!/bin/bash
DIR=$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )
source $DIR/env.sh

export CURRENT_DIR=$DEPLOY_TO/current
cd $DEPLOY_TO/current

if [ -z "${3}" ]
then
  export QUEUES='-q default -q mailers'
else
  export QUEUES="-q ${3}"
fi

export CONCURRENCY=4
/etc/monit/scripts/sidekiq.sh $1 $2
```

What this will allow you to do is pass in the ACTION, INDEX number and QUEUE name via command line like so to start the process

```
/var/www/{env}/{app_name}/shared/scripts/sidekiq.sh start 1 {optional_queue}
```

In this case we're asking it to start sidekiq process 1 without any optional queue name thus defaulting to mailers and default queues.

You use this inside of your monit script at `/etc/monit/conf-enabled/{env}/sidekiq` like so

```
check process sidekiq with pidfile /var/www/{env}/{app_name}/shared/pids/sidekiq.1.pid
    start program = "/var/www/{env}/{app_name}/shared/scripts/sidekiq.sh start 1" as uid www-data and gid www-data
    stop program = "/var/www/{env}/{app_name}/shared/scripts/sidekiq.sh stop 1"
    if cpu > 60% for 2 cycles then alert
    if cpu > 80% for 5 cycles then restart
    if 5 restarts within 5 cycles then timeout
```

If you wanted to specify a custom queue name then your monit configuration would look like this

```
check process sidekiq with pidfile /var/www/{env}/{app_name}/shared/pids/sidekiq.1.pid
    start program = "/var/www/{env}/{app_name}/shared/scripts/sidekiq.sh start 1 high_priority" as uid www-data and gid www-data
    stop program = "/var/www/{env}/{app_name}/shared/scripts/sidekiq.sh stop 1 high_priority"
    if cpu > 60% for 2 cycles then alert
    if cpu > 80% for 5 cycles then restart
    if 5 restarts within 5 cycles then timeout
```

where `high_priority` is the name of the queue.

The finally the key element is the actual daemonization. This is done using the `/etc/monit/scripts/sidekiq.sh` script specified above. In this file drop in the following:

```
#!/bin/bash
PATH=/usr/local/rvm/gems/ruby-$RVM/bin:/usr/local/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
DAEMON=/usr/local/rvm/gems/ruby-$RVM/bin/bundle
DAEMON_OPTS="exec sidekiq -c $CONCURRENCY -t $2 $QUEUES >> $DEPLOY_TO/shared/log/sidekiq.$2.log"
NAME="sidekiq"."$2"
DESC="$APP $APP_ENV sidekiq"
PID_DIR=$DEPLOY_TO/shared/pids

export VERBOSE=1
export PATH=$PATH
export HOME=$DEPLOY_TO

cd $CURRENT_DIR
source /usr/local/lib/rvm
rvm $RVM

case "$1" in
  start)
  echo -n "Starting $DESC: "
        start-stop-daemon --start --pidfile $PID_DIR/$NAME.pid --background --make-pidfile --chdir $CURRENT_DIR --startas /bin/bash -- -c "exec $DAEMON $DAEMON_OPTS"
        echo "$NAME."
        ;;
  stop)
  echo -n "Stopping $DESC: "
  echo `cat $PID_DIR/$NAME.pid`
  kill `cat $PID_DIR/$NAME.pid`
  rm $PID_DIR/$NAME.pid
  echo "$NAME."
  ;;
  restart|force-reload)
  echo -n "Restarting $DESC: "
  kill `cat $PID_DIR/$NAME.pid`
  rm $PID_DIR/$NAME.pid
  sleep 1
        start-stop-daemon --start --quiet --pidfile \
            $PID_DIR/$NAME.pid --background --make-pidfile --chdir $CURRENT_DIR --exec $DAEMON -- $DAEMON_OPTS || true
        echo "$NAME."
        ;;
  reload)
        echo -n "Reloading $DESC configuration: "
        test_nginx_config
        start-stop-daemon --stop --signal HUP --quiet --pidfile $PID_DIR/$NAME.pid \
            --exec $DAEMON || true
        echo "$NAME."
        ;;
  *)
  echo "Usage: $NAME {start|stop|restart|force-reload}" >&2
  exit 1
  ;;
esac

exit 0
```

What this is doing is allowing you to start/stop/restart the sidekiq process. It creates a PIDFILE so monit can track if it's up or not. It also generates a log file which you can use to track what your sidekiq process is doing. 

You'll notice I'm using an `env.sh` file above. If you're curious what's inside there here's a sample one:

```
#!/bin/bash
export APP_NAME=your_app
export APP_ENV=your_app_env
export RUBY_VERSION=your_ruby_version
export DEPLOY_TO=/var/www/production/your_app_directory
export RVM=your_ruby_version@your_app_your_app_env
export RVM_SOURCE=/usr/local/lib/rvm
export USER=www-data
export GROUP=www-data
export GEM_PATH=/usr/local/rvm/gems/$RUBY_VERSION:/usr/local/rvm/gems/$RUBY_VERSION@global:/usr/local/rvm/gems/$RVM
export GEM_HOME=/usr/local/rvm/gems/$RVM
export PATH=/usr/local/bin:/usr/local/sbin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/bin/:$GEM_PATH
export HOME=/var/www
export RAILS_ENV=$APP_ENV
export VERBOSE=true

cd $DEPLOY_TO/current
source $RVM_SOURCE
rvm $RVM
umask 0002
```
