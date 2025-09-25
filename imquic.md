# imquic (Meetecho MoQT Implementation)

[imquic](https://github.com/meetecho/imquic) is a QUIC/WebTransport
library with native and experimental support for RoQ and MoQT. It comes
with a few sample applications to showcase how the library works: for
MoQT, this means a basic relay, a publisher, a subscriber and a
`moq-test` implementation.

The library doesn't currently implement congestion controller, which
means at the moment it's mostly meant as a functional testbed for MoQT.


## Download and Build instructions

At the time of writing, imquic can only be built on Linux and, with a
bit of tweaking, MacOS. The [README.md](https://github.com/meetecho/imquic/blob/main/README.md)
contains info on which dependencies are needed, and how to install them.
Most of the dependencies should be straightforward to install. QuicTLS may
be slightly trickier, since it usually is not directly available in distros repos.

Once you have installed the dependencies, setup the configuration script:

```
./autogen.sh
```

After that, configure and compile as usual to start the whole compilation process:

```
./configure --prefix=/usr/local --enable-moq-examples
make
make install
```

Should you be interested in QLOG support, add `--enable-qlog`. To generate
a documentation for the library, add `--enable-docs`. A documentation for
the library is also available [online](https://imquic.conf.meetecho.com/),
even though it may not always be aligned to the latest version of the code.


## Run the demos

The main MoQT demos that are currently available are documented in the
[examples README.md](https://github.com/meetecho/imquic/blob/main/examples/README.md):

* `imquic-moq-relay`, a basic MoQT relay;
* `imquic-moq-pub`, a basic MoQT publisher (basically a clone on `moq-clock`in [moq-rs](https://github.com/kixelated/moq-rs));
* `imquic-moq-sub`, a basic MoQT subscriber (with support for a few different kinds of media);
* `imquic-moq-test`, a basic MoQT publisher/subscriber that implements the [testing protocol](https://afrind.github.io/moq-test/draft-afrind-moq-test.html) draft (still WIP).

All those demos should in principle be interoperable with other MoQT
implementations. The library currently supports MoQT versions from v06
to v14 (v14 currently WIP). Demos can advertise support for specific
versions (or version groups) using the `-M` flag:

* `-M any`: advertise support from v11 to v14;
* `-M legacy`: advertise support from v06 to v10;
* `-M <num>`: advertise support only for a single specific version.

All demos support both raw QUIC (`-q`) and WebTransport (`-w`). If both
flags are enabled at the same time, the demo will advertise support for
both and use what's eventually negotiated.

All demos support QLOG, if built in the library. Check the
[examples README.md](https://github.com/meetecho/imquic/blob/main/examples/README.md)
for more info on how to selectively enable the logging you're interested in.

### Running the relay

The `imquic-moq-relay` demo is a basic (and not very performant) relay
implementation with support for most of the MoQT features. Use the `-h`
flag for a complete list of supported command line arguments.

If you haven't already, you'll need to generate a certificate and key file.
You can do so with openssl, letsencrypt.org or mkcert. Once available,
they can be passed to the relay using the `-c` and `-k` flags.

This snippet launches a MoQ relay that can be reached both via raw QUIC
and WebTransport (`-q -w`), and that only accepts connections negotiating
version -07 of the draft (`-M`):

```
./examples/imquic-moq-relay -p 9000 -c localhost.crt -k localhost.key -q -w -M 7
```

The relay, while not very efficient (it relies on a global lock for
managing sessions) is supposed to support all of the implemented MoQT
verbs. Some of the known limitations are:

* prioritization is currently not supported;
* incoming `TRACK_STATUS` requests are only handled for currently active
tracks; the request is never forwarded upstream, at the moment;
* incoming `FETCH` requests only return objects that are already
available and cached; the relay doesn't currently go upstream to retrieve
missing objects;
* `SUBSCRIBE_UPDATE` can only be used to change the `forward` policy,
and nothing else, at the moment.

### Running the publisher

The `imquic-moq-pub` demo is a basic publisher implementation. It only
publishes text, in a format compatible with `moq-clock`. It supports
both `ANNOUNCE/PUBLISH_NAMESPACE` + incoming `SUBSCRIBE`, and `PUBLISH`,
as a way to publish content to a relay. Use the `-h` flag for a complete
list of supported command line arguments.

This sample snippet creates a MoQT publisher using WebTransport (`-w`)
that publishes the current time to the `clock` namespace (`-n`; multiple
instances of `-n` add more namespace levels) and `now` (`-N`) track;
since `-M` is not provided, support for multiple versions of MoQT is offered:

```
./examples/imquic-moq-pub -r 127.0.0.1 -R 9000 -w -n clock -N now
```

### Running the subscriber

The `imquic-moq-sub` demo is a basic subscriber implementation. It won't
do anything with objects, except printing info about them depending on
how they should be interpreted. It supports both subscribing and fetching:

* for subscribing, it supports both `SUBSCRIBE` and `SUBSCRIBE_NAMESPACE` + incoming `PUBLISH` as a way to subscribe to a live track;
* for fetching, it supports both Standalone and Joining `FETCH`.

In case any live subscription is active, the demo can optionally be
configured to issue a `SUBSCRIBE_UPDATE` as well. Use the `-h` flag for
a complete list of supported command line arguments.

This snippet creates a MoQT subscriber for the namespace/track showed
in the previous section, using `-t text` to interpret the objects as text.
Raw QUIC (`-q`) is used to connect to the relay:

```
./examples/imquic-moq-sub -r 127.0.0.1 -R 9000 -q -n clock -N now -t text
```

Notice that the `loc` type the subscriber supports is not the latest
version of the LOC specification, as it was meant to test interoperability
with `moq-encoder-player` many months ago, and so is probably outdated.

## Public Instances

Meetecho does not operate any public instances at this time.

## Bug reports

Open an issue on the imquic repo, or message Lorenzo Miniero on the quicdev Slack.
