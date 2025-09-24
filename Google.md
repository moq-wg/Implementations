# Google MoQT Implementation

These instructions explain how to run Google's pure MoQT relay.

## Download and Build instructions

Follow the [instructions](https://bazel.build/install) to install Bazel.

```
sudo apt-get install libicu-dev
cd <directory that will be the root of your quiche implmentation>
git clone https://github.com/google/quiche.git
cd quiche
bazel build -c opt -cxxopt=”-w” quiche:moqt_relay
```

## Run the relay

If you haven't already, you'll need to generate a certificate and key file. You can
do so with openssl or letsencrypt.org.

Minimum command:

```
./bazel-bin/quiche/moqt_relay –certificate_file=”<certfile>” –key_file=”<keyfile>”
```

Optional command line flags:

```
--disable_certificate_verification={true, false}

default: false
If true, ignores the certificate of any attaching client.

--bind_address="<string>"

default: "127.0.0.1"
A string represenation of the IP address the relay will listen on.

--port=<uint16>

default: 9667
The port the relay will listen on.

--default_upstream="URL"

default: ""
If not "", on startup the relay will attempt to connect to this URL. If successful,
it will forward all requests in a namespace that has not been PUBLISHed to this
peer.
```

## Connect to the relay

The relay does not care what Path you use to connect to it.

The relay only supports WebTransport; it does not run over Raw QUIC.

draft-14 only.

## Supported functions

- Accepts PUBLISH_NAMESPACE and adds the entry to its routing table.

- Forward SUBSCRIBE to one peer in the routing table for an inclusive namespace, or
the default if not in the table. *Does NOT* support relay of PUBLISH_DONE.

## Public Instances

Google does not operate any public instances at this time.

## Bug reports

Email martin.h.duke@gmail.com, or message Martin Duke on the quicdev Slack.
