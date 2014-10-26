.. post:: Apr 10, 2013
    :tags: perl, anyevent, websockets

    Background information on websockets and an example walkthrough building an
    evented web application with websockets

Plack, AnyEvent, Redis, and SockJS
==================================

I thought I would share some of what I've been doing with SockJS and Redis in
Perl lately. This is an example application and some background information on
websockets that will be part of a talk on AnyEvent and Plack/Twiggy at
`PDX.pm`_ this week.

.. _PDX.pm: http://pdx.pm.org

WebSockets
----------

A websocket is a bi-directional socket connecting a client and the server. It
allows for communication similar to comet or ajax, without polling overhead.
A couple websocket wrapper implementations have popped up in the Node.js
community: `Socket.IO`_ and `SockJS`_.

.. _Socket.IO: http://socket.io
.. _SockJS: http://github.com/sockjs/sockjs-client

Both implementations use `alternative transports`_ to create a cross-browser
websocket, or websocket-like, connection. Cross-browser support makes use of a
proper WebSocket protocol initially, and failing that, backs down to a
`supported transport`_.

.. _alternative transports: https://github.com/sockjs/sockjs-client#supported-transports-by-browser-html-served-from-http-or-https
.. _supported transport: https://github.com/sockjs/sockjs-client#supported-transports-by-name

SockJS vs Socket.IO
-------------------

`SockJS`_ was a project started to address bloat in `Socket.IO`_.  In the
relative past, SockJS has appeared to be a more consistent project, and has had
more active development. In response to a competing project, `Socket.IO`_ was
trimmed down to reduce bloat and given a different name: `Engine.IO`_.

.. _Engine.IO: https://github.com/LearnBoost/engine.io

WebSocket Server
----------------

These websocket implementations require a server-side component to run. The Node
SocketIO module is the server module that also distributes the client side
module, and SockJS is broken into a server module and client side javascript.

WebSockets in Perl
------------------

We can use them in Perl!

* `Perl's SockJS`_ is the SockJS server to compliment the javascript SockJS
  client
* `PocketIO`_ is the Perl Socket.IO server

.. _Perl's SockJS: https://github.com/vti/sockjs-perl
.. _PocketIO: https://github.com/vti/pocketio.git

But, it's not, *not* painful.

Problems
--------

* Make sure to turn chunking off on the SockJS instance, otherwise::

    4f
    {"origins":["*:*"],"entropy":3165083279,"websocket":true,"cookie_needed":false}
    0

Will be your output on the websocket. This is chunked return from Twiggy, which
is returned because Twiggy returns via HTTP1.1 by default.
