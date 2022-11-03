# System Unit Examples for Exim

This directory contains several examples for Systemd units to manage an Exim installation.
There is room for improvement, so please share your ideas or setups that are proven to work
in your environment.

All the service units try to protect the system from unintentional
writes to locations outside of Exim's spool, and log directories.  You
may need to override specific settings, we recommend using Systemd's
override mechanism.

## Daemon

This is best suited for *average to high traffic systems*, as it engages
all built-in Exim facilities, as queue runner management, and system load
depended message processing.

It starts the Exim main process. This process listens on the ports
configured in the _runtime configuration_, and supervises all other
activities, including management of queue runner startup (`exim -odf
-q...`).

For regular maintenance tasks (log rotation, database cleanup)
additional units are required.

## Socket

This is best suited for *low traffic* systems, which experience a
message *burst* from time to time. Regular desktop, and edge systems fit this
pattern.

Exim's start is delayed until the first connection. Once a connection is
initiated, Exim starts a listener on the port configured in the [systemd
socket unit](./socket/exim.socket) and waits for more connections, and exits after being idle
(`exim -bw...`).

Additional [_queue runner_ timer and service units](#queue-runner) are required.

For regular maintenance tasks (log rotation, database cleanup)
additional units are required.

## Inetd-like Socket

This is best suited for systems with *low traffic*, if the
[socket](#socket) approach doesn't work.

For each incoming connection a new Exim instance starts, handling
exactly this connection and then exits. The listener port is configured
in the [systemd socket unit](./inetd/exim.socket).

Additional [_queue runner_ timer and service units](#queue-runner) are required.

For regular maintenance tasks (log rotation, database cleanup)
additional units are required.

## Queue Runner

This is a *timer*, and a service unit which starts Exim queue runner
processes. This is necessary, as the socket activated Exim instances
(from [socket](#socket) and [inetd](#inetd-like-socket))
do not care once the first delivery attempt is done.

## Maintenance

This is a *timer* and triggers regular cleanup tasks.
For security it is recommended to use the `User=` Systemd directive in a
local override file.
