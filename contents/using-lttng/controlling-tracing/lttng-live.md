---
id: lttng-live
---

We have seen how trace files may be produced by LTTng out of generated
application and Linux kernel events. We have seen that those trace files
may be either recorded locally by consumer daemons or remotely using
a relay daemon. And we have seen that the maximum size and count of
trace files is configurable for each channel. With all those features,
it's still not possible to read a trace file as it is being written
because it could be incomplete and appear corrupted to the viewer.
There is a way to view events as they arrive, however: using
_LTTng live_.

LTTng live is implemented, in LTTng, solely on the relay daemon side.
As trace data is sent over the network to a relay daemon by a (possibly
remote) consumer daemon, a _tee_ may be created: trace data will be
recorded to trace files _as well as_ being transmitted to a
connected live viewer:

<div class="img img-90">
    <object data="/images/docs/lttng-live-relayd.svg" type="image/svg+xml">
        <img src="/images/docs/lttng-live-relayd.svg">
    </object>
</div>

In order to use this feature, a tracing session must created in live
mode on the target system:

<pre class="term">
lttng create --live
</pre>

An optional parameter may be passed to `--live` to set the interval
of time (in microseconds) between flushes to the network
(1&nbsp;second is the default):

<pre class="term">
lttng create --live 100000
</pre>

will flush every 100&nbsp;ms.

If no network output is specified to the `create` command, a local
relay daemon will be spawned. In this very common case, viewing a live
trace is easy: enable events and start tracing as usual, then use
`lttng view` to start the default live viewer:

<pre class="term">
lttng view
</pre>

The correct arguments will be passed to the live viewer so that it
may connect to the local relay daemon and start reading live events.

You may also wish to use a live viewer not running on the target
system. In this case, you should specify a network output when using
the `create` command (`--set-url` or `--ctrl-url`/`--data-url` options).
A distant LTTng relay daemon should also be started to receive control
and trace data. By default, `lttng-relayd` listens on 127.0.0.1:5344
for an LTTng live connection. Otherwise, the desired URL may be
specified using its `--live-port` option.

The
<a href="http://www.efficios.com/babeltrace" class="ext">`babeltrace`</a>
viewer supports LTTng live as one of its input formats. `babeltrace` is
the default viewer when using `lttng view`. To use it manually, first
list active tracing sessions by doing the following (assuming the relay
daemon to connect to runs on the same host):

<pre class="term">
babeltrace -i lttng-live net://localhost
</pre>

Then, choose a tracing session and start viewing events as they arrive
using LTTng live, e.g.:

<pre class="term">
babeltrace -i lttng-live net://localhost/host/hostname/my-session
</pre>
