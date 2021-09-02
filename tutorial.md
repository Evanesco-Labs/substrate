# WhiteNoise Tutorial

WhiteNoise is a privacy network protocol like Tor but with more attractive features.
You can learn more details about WhiteNoise protocol in this [Specification](https://github.com/Evanesco-Labs/WhiteNoise.rs/blob/main/docs/whitenoise_spec.md).

We implement the WhiteNoise protocol in Rust and integrate it with Substrate RPC to better protect Substrate node and user's privacy.


In the following, We will introduce how to use Substrate RPC with WhiteNoise network.

## 1. Start a WhiteNoise Network
WhiteNoise network is a decentralized network consists of multiply nodes, they can be divided into two categories, namely **Bootstrap Node** and **Routing Node**.

Bootstrap node is used for node discovery and Routing node is used to relay messages.

Follow these steps to startup a local WhiteNoise network with one bootstrap node and 4 routing nodes.

### Build WhiteNoise.rs
Building requires Rust toolchain. First complete the basic [Rust setup instructions](https://github.com/Evanesco-Labs/substrate/blob/master/bin/node-template/docs/rust-setup.md).
Then clone [WhiteNoise.rs](https://github.com/Evanesco-Labs/WhiteNoise.rs) to your local device.

Use the following command to build the WhiteNoise node:

```shell
cargo build --release
```

After building success, you can see a new executable file `whitenoisers` in directory `./target/release`.

Make 5 new directories and copy `whitenoisers` into these directories.

```shell
mkdir boot node1 node2 node3 node4    
cp ./target/release/whitenoisers boot 
cp ./target/release/whitenoisers node1
cp ./target/release/whitenoisers node2
cp ./target/release/whitenoisers node3
cp ./target/release/whitenoisers node4
```

### Start Bootstrap Node
This command will start a WhiteNoise node as a Bootstrap, listening to port "3331":

```shell
cd boot
 ./whitenoisers start --port 3331
```

After running this command, the local **MultiAddress** of Bootstrap is shown in log like the following:
```shell
2021-08-23T07:41:34.207Z INFO  whitenoisers] [WhiteNoise] bootstrap_addr:None
[2021-08-23T07:41:34.207Z INFO  whitenoisers] [WhiteNoise] port str:Some("3331")
[2021-08-23T07:41:34.209Z INFO  libp2p_gossipsub::behaviour] Subscribed to topic: noise_topic
[2021-08-23T07:41:34.210Z INFO  whitenoisers::sdk::host] [WhiteNoise] local Multiaddress: /ip4/127.0.0.1/tcp/3331/p2p/12D3KooWRcCCUorgKZqAaJwKAYQbdEk3AiseXTcmZMzmhPkJyiZF
```
Remember the value of bootstrap node's MultiAddress printed in log, in this example `/ip4/127.0.0.1/tcp/3331/p2p/12D3KooWRcCCUorgKZqAaJwKAYQbdEk3AiseXTcmZMzmhPkJyiZF`. We need this MultiAddress to startup the routing nodes.

### Start Routing Nodes
Start another terminal and run the following command in node1 directory.

This command will start a WhiteNoise node as routing node, which listens to port "3332". Make sure the port is
available and fill in the bootstrap node's MultiAddress in the `--bootstrap` flag:
```shell
./whitenoisers start --port 3332 --bootstrap /ip4/127.0.0.1/tcp/3331/p2p/12D3KooWRcCCUorgKZqAaJwKAYQbdEk3AiseXTcmZMzmhPkJyiZF
```
Log will show like the following:
```shell
[2021-08-23T07:46:37.554Z INFO  whitenoisers] [WhiteNoise] bootstrap_addr:Some("/ip4/127.0.0.1/tcp/3331/p2p/12D3KooWRcCCUorgKZqAaJwKAYQbdEk3AiseXTcmZMzmhPkJyiZF")
[2021-08-23T07:46:37.554Z INFO  whitenoisers] [WhiteNoise] port str:Some("3332")
[2021-08-23T07:46:37.557Z INFO  libp2p_gossipsub::behaviour] Subscribed to topic: noise_topic
[2021-08-23T07:46:37.557Z INFO  whitenoisers::sdk::host] [WhiteNoise] local Multiaddress: /ip4/127.0.0.1/tcp/3332/p2p/12D3KooWKLSRXB1tKTRDnUZUiQQw7tWSGNXeEBDjqHaUiCdiRodn
[2021-08-23T07:46:37.558Z INFO  whitenoisers::network::whitenoise_behaviour] [WhiteNoise] routing updated,peer:PeerId("12D3KooWRcCCUorgKZqAaJwKAYQbdEk3AiseXTcmZMzmhPkJyiZF"),addresses:["/ip4/127.0.0.1/tcp/3331"]
[2021-08-23T07:46:37.560Z INFO  libp2p_gossipsub::behaviour] New peer connected: 12D3KooWRcCCUorgKZqAaJwKAYQbdEk3AiseXTcmZMzmhPkJyiZF
[2021-08-23T07:46:37.561Z INFO  whitenoisers::network::whitenoise_behaviour] [WhiteNoise] routing updated,peer:PeerId("12D3KooWRcCCUorgKZqAaJwKAYQbdEk3AiseXTcmZMzmhPkJyiZF"),addresses:["/ip4/127.0.0.1/tcp/3331", "/ip4/127.0.0.1/tcp/3331/p2p/12D3KooWRcCCUorgKZqAaJwKAYQbdEk3AiseXTcmZMzmhPkJyiZF"]
```

Like node1, go to the other routing node directories and run the same command, but with a new `--port` parameter each.

Don't close the terminals running bootstrap node and four terminals running routing nodes, when continue the next steps.

## 2. Start Substrate with WhiteNoise RPC
### Build Substrate Fork
First, clone our [Substrate fork](https://github.com/Evanesco-Labs/substrate). Use the following command to build the node without launching it:
```shell
cargo build --release
```
After building success, you can see a new executable file `node-template` in directory `./target/release`.

### Start node-template
Generate a new directory and copy node-template into this directory with this command:
```shell
mkdir rpctest
cp ./target/release/node-template ./rpctest/
```

Start a node-template with WhiteNoise based rpc, with the following command. Fill in the bootstrap node's MultiAddress in the `--whitenoise-bootstrap` flag.
```shell
./rpctest/node-template \
--base-path /tmp/bob \
  --chain local \
  --bob \
  --port 30333 \
  --ws-port 9945 \
  --rpc-port 9933 \
  --validator \
--whitenoise-bootstrap /ip4/127.0.0.1/tcp/3331/p2p/12D3KooWRcCCUorgKZqAaJwKAYQbdEk3AiseXTcmZMzmhPkJyiZF
```

Log prints like the following:
```shell
2021-08-23 16:01:28 [WhiteNoise] local whitenoise id:039YdNbebFWZBGYE2DNsE21MRHFqLgUsXheL6xj7e2Bmk    
2021-08-23 16:01:28 [WhiteNoise] finished get mainnets this turn:5MUeAz3UwWjkreXzL8seC87dJwb3qv4NgeabouMyHq3U    
2021-08-23 16:01:28 [WhiteNoise] mainnets address:["/ip4/127.0.0.1/tcp/3332", "/ip4/192.168.1.14/tcp/3332"],peer_id:PeerId("12D3KooWKLSRXB1tKTRDnUZUiQQw7tWSGNXeEBDjqHaUiCdiRodn")    
2021-08-23 16:01:28 choose id:PeerId("12D3KooWKLSRXB1tKTRDnUZUiQQw7tWSGNXeEBDjqHaUiCdiRodn") to register    
2021-08-23 16:01:28 [WhiteNoise] finished register proxy this turn:Ack { command_id: "E1TzYuBSQozgkEVPUCAa4m2JhbDtku8juP7JFHLQrWhk", result: true, data: [] }    
2021-08-23 16:01:28 Listening for new connections on 127.0.0.1:9945.
```

The identity of this WhiteNoise rpc server is called WhiteNoiseID, and it is shown in log.

Remember this WhiteNoiseID printed in your log, rpc client have to dial this ID to send request.
In this example the WhiteNoiseID is `039YdNbebFWZBGYE2DNsE21MRHFqLgUsXheL6xj7e2Bmk`.

Notice that if you what to start multiple node-templates do not use the same node key or account.

Also don't close the terminal running node-template, when continue the next steps.

## 3. Request Substrate WhiteNoise RPC

### Build WhiteNoise RPC Client
First clone [WhiteNoise-RPC](https://github.com/Evanesco-Labs/WhiteNoise-RPC). Use the following command to build the WhiteNoise RPC client:
```shell
cargo build --release
```
After building success, you can see a new executable file `whitenoise-rpc` in directory `./target/release`.

Make a new directory `rpc-client` and copy `whitenoise-rpc` into this directory:
```shell
mkdir rpc-client
cp ./target/release/whitenoise-rpc ./rpc-client
cd ./rpc-client
```

### New Json Requests
Here we will try two substrate rpc requests through WhiteNoise network as an example. We will make two json files for each request.

In directory `rpc-client`, make new json file named `insert_request.json` and copy the following to this file.
This request will call the `insertKey` method of node-template.
```shell
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

Make another json file named `has_request.json` and copy the following to this file.
This request will call the `hasKey` method of node-template.
```shell
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

### Send WhiteNoise RPC Requests
Send the hasKey request with the following command. Fill in the boostrap MultiAddress in the `--bootstrap` flag, and the node-template's WhiteNoiseID in the `--id` flag.
```shell
./whitenoise-rpc --bootstrap /ip4/127.0.0.1/tcp/3331/p2p/12D3KooWRcCCUorgKZqAaJwKAYQbdEk3AiseXTcmZMzmhPkJyiZF --id 039YdNbebFWZBGYE2DNsE21MRHFqLgUsXheL6xj7e2Bmk --json ./has_request.json
```

As there is no such key, the response will be printed as the following:
```shell
{"jsonrpc":"2.0","result":false,"id":1}
```

Then call the insertKey method with this command:
```shell
./whitenoise-rpc --bootstrap /ip4/127.0.0.1/tcp/3331/p2p/12D3KooWRcCCUorgKZqAaJwKAYQbdEk3AiseXTcmZMzmhPkJyiZF --id 039YdNbebFWZBGYE2DNsE21MRHFqLgUsXheL6xj7e2Bmk --json ./insert_request.json
```

The response will be printed as the following:
```shell
response: {"jsonrpc":"2.0","result":null,"id":1}
```
Finally call the hasKey method again to check if insert succeed:
```shell
./whitenoise-rpc --bootstrap /ip4/127.0.0.1/tcp/3331/p2p/12D3KooWRcCCUorgKZqAaJwKAYQbdEk3AiseXTcmZMzmhPkJyiZF --id 039YdNbebFWZBGYE2DNsE21MRHFqLgUsXheL6xj7e2Bmk --json ./has_request.json
```
Finally this response will be printed:
```shell
response: {"jsonrpc":"2.0","result":true,"id":1}
```

## 4. WhiteNoise Testnet
Instead of starting your local WhiteNoise network to try and test, you can also use WhiteNoise Testnet.

WhiteNoise Testnet is a remote WhiteNoise network with multiple nodes.
The bootnode MultiAddress of WhiteNoise testnet is `/ip4/122.9.132.178/tcp/3331/p2p/12D3KooWJahyhDUv7KkqNqdY9cK1WpdiSE2tX5ckNawj87AYB5Nw`.

You can use this testnet bootstrap MultiAddress to replace your local bootstrap MultiAddress and try step three and four again. 


