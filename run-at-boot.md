To run *superplay* when the computer boots up, you'll need
to create an init script for it. The steps below have been
tested on the *Raspbian* image, which is based on Debian
Linux, and uses [init.d startup scripts](http://www.debian.org/doc/debian-policy/ch-opersys.html#s-sysvinit).

## Install

As the *root* user, copy the sample file below to
`/etc/init.d/superplay`, making sure the script is
executable (using `chmod 755 filename` should do the trick).
Now you can start and stop the service with these commands:

~~~
$ sudo /etc/init.d/superplay start
$ sudo /etc/init.d/superplay stop
$ sudo /etc/init.d/superplay restart
~~~

With the init script in place, install it in the
system boot sequence using the `update-rc.d` command:

~~~
$ sudo update-rc.d superplay defaults
~~~

Now when the system reboots, you should see a message that
the superplay Media Server is up and running.

You can uninstall `superplay` from the boot sequence by running:

~~~
$ sudo update-rc.d -f superplay remove
~~~

## Init Script

Here's an example init script for `/etc/init.d/superplay`.
You'll probably need to change the `USER`, `PATH`, `DAEMON`,
and `DAEMON_ARGS` variables to match your own system setup.

~~~
#! /bin/sh
### BEGIN INIT INFO
# Provides:          superplay
# Required-Start:    $remote_fs $syslog
# Required-Stop:     $remote_fs $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: superplay initscript
# Description:       superplay init at boot.
### END INIT INFO

USER=me
# node must be in PATH, or in the superplay directory
PATH=/sbin:/usr/sbin:/bin:/usr/bin:/home/$USER/bin
NAME=superplay
LONG_NAME="superplay Media Server"
DAEMON=/home/$USER/src/$NAME/bin/$NAME
DAEMON_ARGS="--background --add /home/$USER/vids"
PIDFILE=/var/run/$NAME.pid
SCRIPTNAME=/etc/init.d/$NAME

[ -x "$DAEMON" ] || exit 0

. /lib/lsb/init-functions

do_start() {
  if start-stop-daemon --start --quiet --oknodo --background \
    --chuid $USER --pidfile $PIDFILE --make-pidfile \
    --exec $DAEMON -- $DAEMON_ARGS; then
    return 0
  else
    return 1
  fi
}

do_stop() {
  if start-stop-daemon --stop --quiet \
    --retry=TERM/30/KILL/5 --pidfile $PIDFILE; then
    rm -f $PIDFILE
    return 0
  else
    rm -f $PIDFILE
    return 1
  fi
}

case "$1" in
  start)
    log_daemon_msg "Starting $LONG_NAME" $NAME || true
    if do_start; then
      log_end_msg 0 || true
    else
      log_end_msg 1 || true
    fi
    ;;
  stop)
    log_daemon_msg "Stopping $LONG_NAME" $NAME || true
    if do_stop; then
      log_end_msg 0 || true
    else
      log_end_msg 1 || true
    fi
    ;;
  restart)
    log_daemon_msg "Restarting $LONG_NAME" $NAME || true
    do_stop
    if do_start; then
      log_end_msg 0 || true
    else
      log_end_msg 1 || true
    fi
    ;;
  *)
    echo "Usage: /etc/init.d/$NAME {start|stop|restart}"
    exit 1
    ;;
esac

exit 0
~~~
