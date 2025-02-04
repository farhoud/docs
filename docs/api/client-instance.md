---
title: Client Instance
id: client-instance
---
import WorkInProgress from '../components/WorkInProgress.mdx'

# Client Instance
FULA client package provides a method `createClient` that returns client instance. You can use this object to call File and Data APIs.

The client instance is basically a wrapper around a libp2p node. It abstracts away the protocol layer and makes it easier to communicate over libp2p with FULA's [File](https://github.com/functionland/fula/tree/0766d2e4ab56b423a00f2c28a132ea1a8f871164/protocols/file) and [Graph](https://github.com/functionland/fula/tree/0766d2e4ab56b423a00f2c28a132ea1a8f871164/protocols/graph) protocols.

In addition to File and Data APIs, the client instance provides access to the underlying libp2p node. It also maintains the connection between client and the Box and provides a reconnecting mechanism.

You can find basic methods for working with the client instance here on this page.

## Create a new client instance
You can use the `createClient` method to create a new client instance.

`async createClient(config?, pKey=undefined)`

Arguments are:
- `config?`: The configuration object that gets passed directly to the underlying libp2p node. See [here](https://github.com/libp2p/js-libp2p/blob/master/doc/CONFIGURATION.md) for more information
- `pkey`: The private key used for securing the connection. FULA client is currently using `NOISE` as the connection encryptor, if you are changing that in `config` make sure you provide appropriate arguments.

The method resolves to a FULA instance.

Example:
```javascript
import {createClient} from '@functionland/fula'
const client = await createClient()
```

## Connect to Box
In order to use different APIs provided by the FULA client, you must connect a client instance to a Box app. For that you can use the `connect` method.

`async connect(serverId)`

Arguments are:
- `serverId`: The base58 PeerID string provided by Box app or any other libp2p node supporting File and Graph protocols.

Example:
```javascript
const serverId = '12D3KooWBFCDpMyEmyAAAAY6PiQw2vaM35ChTZ8ZmVUe8GFRMUrt' // copied from box app
await client.connect(serverId)
```

## Close a connection
You can disconnect an already connected client instance with `close` method. This will close the connection and also stops the libp2p node.

`async close()`

Example:
```javascript
await client.close()
```


## Access to Libp2p node
As mentioned before, the FULA client is a wrapper around a libp2p node abstracting away the protocol layer and providing APIs to communicate over File and Graph protocols. If you want to do more specific configurations or access libp2p cache or other interfaces, you can use `getNode` method.

`getNode()`

The method returns a Libp2p node.

Example:
```javascript
const libp2pNode = client.getNode()
console.log(libp2pNode.multiaddrs)
// [ <Multiaddr 047f00000106f9ba - /ip4/127.0.0.1/tcp/63930> ]
```


<WorkInProgress />
