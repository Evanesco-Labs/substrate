# Substrate WhiteNoise-based RPC

This implementation adds a WhiteNoise-based rpc component to Substrate, so that clients and Substrate nodes can
make rpc requests and responses through the WhiteNoise network. The rpc messages will pass through the WhiteNoise
routing node and reach the dest address through multi-hop connection. This can protect the network privacy of both the
rpc client and the node. Here is more details
about [WhiteNoise protocol](https://github.com/Evanesco-Labs/WhiteNoise.rs).

## Getting Started

Follow the steps below to get started substrate with the WhiteNoise-based RPC.

### Rust Setup

First, complete the [basic Rust setup instructions](./bin/node-template/docs/rust-setup.md).

### Build

The `cargo build` command will perform an initial build. Use the following command to build the node without launching
it:

```sh
cargo build --release
```

### WhiteNoise RPC Component

We have added a new parameter for substrate node  `--whitenoise-bootstrap`. It specifies the bootstrap node address of
the WhiteNoise Network that you want to connect to. WhiteNoise Network is a decentralized network like TOR network, and user
needs to access the network through the bootstrap node. After the Substrate node is started, it will register to the
WhiteNoise network and get a WhiteNoiseID. Then a rpc client is able to generate a privacy connection by dialing the
WhiteNoiseID, and sends an rpc request on this connection. After Substrate nodes receive the request, they also response
on the same connection.

Notice that the WhiteNoiseID of a substrate node is derived from the node key.
The WhiteNoiseID should be unique in the WhiteNoise network, otherwise sending and receiving messages may be affected for different clients with the same WhiteNoiseID.
So do not use the same node key to start up multiple substrate nodes with WhiteNoise RPC.

If parameter `--whitenoise-bootstrap` is filled, substrate node will start with WhiteNoise-based rpc, otherwise it will not.

You can also see information about this param with this command:

```sh
./target/release/node-template -h | grep whitenoise
```

## Example

### Start a Local WhiteNoise Network

Follow the [instructions](https://github.com/Evanesco-Labs/WhiteNoise.rs#start-local-whitenoise-network) to start a local
WhiteNoise Network.

For better description, we assume that the bootstrap node address we start is the following:

`/ip4/127.0.0.1/tcp/6661/p2p/12D3KooWMNFaCGrnfMomi4TTMvQsKMGVwoxQzHo6P49ue6Fwq6zU`

### Start substrate node-template

Generate a new directory and copy node-template into this directory with this command:

```shell
mkdir rpctest
cp ./target/release/node-template ./rpctest/
```

Start a node-template with WhiteNoise based rpc.

```shell
./rpctest/node-template \
--base-path /tmp/alice \
  --chain local \
  --alice \
  --port 30333 \
  --ws-port 9945 \
  --rpc-port 9933 \
  --validator \
--whitenoise-bootstrap /ip4/127.0.0.1/tcp/6661/p2p/12D3KooWMNFaCGrnfMomi4TTMvQsKMGVwoxQzHo6P49ue6Fwq6zU
```

The identity of this WhiteNoise rpc server called WhiteNoiseID, it is shown in log. Remember this WhiteNoiseID printed in your log,
rpc client have to dial this id to request. The WhiteNoiseID
is `07sYJEC6MiSP6PZBuhq6KJUwgHhJNvwVWipySMR8peVJs` in the following log:

```shell
2021-06-28T05:39:36.426Z INFO  whitenoisers::network::node] [WhiteNoise] local whitenoise id:07sYJEC6MiSP6PZBuhq6KJUwgHhJNvwVWipySMR8peVJs
```

### Client Request

We also implement a WhiteNoise rpc client to send raw json rpc requests and print out response. Clone and
build [WhiteNoise-RPC](https://github.com/Evanesco-Labs/WhiteNoise-RPC#rpc-client).

Then generate a new file `./insert_request.json` and copy a json request to this file. In this test, we first call
insertKey method. Copy the following json request to `./insert_request.json`:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "author_insertKey",
  "params": [
    "aura",
    "clip organ olive upper oak void inject side suit toilet stick narrow",
    "0x9effc1668ca381c242885516ec9fa2b19c67b6684c02a8a3237b6862e5c8cd7e"
  ]
}
```

Send the first request with this command:

```shell
./target/release/whitenoise-rpc --bootstrap /ip4/127.0.0.1/tcp/6661/p2p/12D3KooWMNFaCGrnfMomi4TTMvQsKMGVwoxQzHo6P49ue6Fwq6zU --id 06ASf5EcmmEHTgDJ4X4ZT5vT6iHVJBXPg5AN5YoTCpGWt --json ./insert_request.json
```

You can see the response:

```
response: {"jsonrpc":"2.0","result":null,"id":1}
```

We can send another request to check if it has the Key we insert. Generate a new file `./has_request.json` and copy the
following json request to `./has_request.json`:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "author_hasKey",
  "params": [
    "0x9effc1668ca381c242885516ec9fa2b19c67b6684c02a8a3237b6862e5c8cd7e",
    "aura"
  ]
}
```

Send the second request with this command:

```shell
./target/release/whitenoise-rpc --bootstrap /ip4/127.0.0.1/tcp/6661/p2p/12D3KooWMNFaCGrnfMomi4TTMvQsKMGVwoxQzHo6P49ue6Fwq6zU --id 06ASf5EcmmEHTgDJ4X4ZT5vT6iHVJBXPg5AN5YoTCpGWt --json ./has_request.json
```

You can see the second response, the Key is successfully inserted:

```
response: {"jsonrpc":"2.0","result":true,"id":1}
```
