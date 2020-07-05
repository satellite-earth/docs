# @satellite-earth/client

## Usage

``` js
const Satellite = require('@satellite-earth/client');
const { Earth } = require('@satellite-earth/core');

// Check that user has an Ethereum wallet installed
if (!window.ethereum) {
	alert('Please install MetaMask');
}

// Earth provides blockchain interface
const earth = new Earth();
await earth.connect();

// Create Satellite client, passing in the earth interface
// and a function describing how your application should
// react (if at all) as the client emits various events
const client = new Satellite(earth, (event, data, params) => {

	// Logic for handling client events goes here, e.g.

	if (event === 'contact') {

	} else if (event === 'torrent_added') {

	}

	// . . .
});
```

**Constructor**

1. `Earth` - Core API instance
2. `Function` - Event handler, called with three parameters: name of the event (e.g. `contact`), data specific to that event, and optional custom params. See the [*Client Events*](#client-events) section for details on each event.
3. `Object` - Options
	- `defaultTrackers` - `Array`: Default torrent tracker URIs (additional trackers can be specified on a per-torrent basis)
	- `getWebseed` - `Function`: Async function called with torrent model to return webseed for that torrent. Useful for creating temporary presigned webseeds on an as-needed basis.


## Properties

### earth

```  js
client.earth
```

`Earth` - Core API instance used to communicate with the blockchain and authenticate data. See docs for the core module.

---

### webtorrent

``` js
client.webtorrent
```

`WebTorrent` - The client contains an internal WebTorrent instance to share data with peers. See [WebTorrent docs](https://webtorrent.io/docs).

---

### worlds

``` js
client.worlds
```

`Object` - Worlds that the client has contacted.

## Methods

### contact

``` js
await client.contact('myworld', {
	endpoint: 'https://example.com/contact',
	tracking: [ 'mystate', 'myotherstate' ]
});

console.log(client.worlds);
// {
//   myworld: {
// 	   current: <Epoch>,
//	   endpoint: 'https://myexampleserver.com',
//	   history: [],
//	   tracking: [ 'mystate', 'myotherstate' ]
//   }
// }
```

Synchronize client with a world instance.

**Parameters**

1. `String` - Name of the world (ID of epoch signer)
2. `Object` - Options
	- `endpoint` - `String`: URI to contact the remote world instance
	- `tracking` - `Array`: Names of the states, if any, the client should build
	- `skipCache` - `Boolean`: If client should ignore locally cached signals (default `false`)
	- `drop` - `Boolean`: If client should comply with server requests to remove locally cached signals (default `true`)

**Returns**

`Promise`

---

### target

``` js
const target = await app.client.target('myworld', 'myaction');
```

Get signal params in preparation for sending a signal to a world. The value returned may be used as the second paramater of the `Signal` constructor.

!!! Note
	`target` is provided as a standalone method in case your application needs to sign data or send signals to the remote world instance in a non-standard way (i.e. you need access to the signal target before prompting user signature) but in most cases it's probably easier to allow the following `signal` method to automatically handle targeting, signing, and sending.

**Parameters**

1. `String` - Name of the world (ID of epoch signer)
2. `String` - Name of action being targeted
3. `Object` - Options
	- `confirm` - `Number`: Number of blocks behind the latest block to use as signal timestamp. Higher values provide more certainty that the targeted block will not become uncled. (Default `0`)

**Returns**

`Promise` returns `Object`: the signal target

---

### signal

``` js
await client.signal('myworld', 'myaction', {
	foo: 'This is the payload to be signed',
	bar: 'It can be whatever'
});
```

Target signal, prompt user signature, and send signal to remote world server.

**Parameters**

1. `String` - Name of the world (ID of epoch signer)
2. `String` - Name of action being targeted
3. `Object` - Payload to be signed
4. `Object` - *Options (same as target — see above)*

**Returns**

`Promise` returns `Object`: Http response from remote world server

---

### load

``` js
client.load({
	'@': "sbowman > publish > genesis > 0xcff80f6aeb7729329fa8be15adbcac1832f781dc1c7e5abf3a59c36acec71ffb"
	meta: "9441.3phpV41A3O1RINpu6qFtBj4NUgM=.QmXGvFGpbucGmZNPJyaQssMDvhHhMdpQjkfYE6tex1RJVu"
	subtitle: "Satellite is not just a decentralized blogging app. More generally, it's a framework for self-authenticating, publicly-hosted, platform-agnostic social data."
	title: "How Satellite Works: Decentralized Social Assets #fundamentals"
});
```

Download or start seeding a torrent. This method doesn't return anything — instead, the client should listen for events and react accordingly.

**Parameters**

1. `Torrent|Object` - Model of the torrent. You can pass a proper `Torrent` instance, or just the payload object. Either will work.
2. `Object` - Options
	- `announce` - `Array`: Additional trackers to use for this torrent
	- `eventParams` - `Object`: Custom parameters passed to the event handler. Useful for making the client react to torrent-specific events.

---

### remove

``` js
await client.remove(torrent.infoHash);
```

Stop downloading or seeding a torrent.

**Parameters**

1. `String` - Infohash of torrent to remove
2. `Object` - Options
	- `removeData` - `Boolean`: Also delete cached data for this torrent (default `false`)

**Returns**

`Promise`

---

### cacheTorrentMeta

``` js
await cacheTorrentMeta(torrent.infoHash, torrent.info);
```

This method is used internally by the client. Don't call it manually unless you know what you're doing.

**Parameters**

1. `String` - Torrent infohash
2. `Object` - Torrent infodict
	- `name` - `Uint8Array`: Torrent name
	- `pieces` - `Uint8Array`: Concatenated sha1 hashes of torrent pieces
	- `length` - `Number`: Torrent length in bytes
	- `piece length` - `Number`: Piece length in bytes 

**Returns**

`Promise`

---

### cacheTorrentData

``` js
await client.cacheTorrentData(blob, { infoHash: torrent.infoHash });
```

Save torrent data manually. Torrent chunks are automatically cached as they are downloaded, so this method is only necessary if you are loading data into cache from an external source and you already know the infohash of the torrent for the data.

**Parameters**

1. `Blob` - Data to cache
2. `Object` - Params
	- `infoHash` - `String`: Infohash of torrent data belongs to (required)

**Returns**

`Promise`

---

### traverseCachedDataKeys

``` js
const returned = await client.traverseCachedDataKeys(torrent.infoHash, (key, parsed) => {
	console.log(key); // '3226d40f4c197f7450e0c768d9dfc3f0a3f254d3.0'
	console.log(parsed.infoHash); // '3226d40f4c197f7450e0c768d9dfc3f0a3f254d3'
	console.log(parsed.index); // 0
	return key;
});
```

Iterate across every chunk of a given torrent in the local data store. To iterate across *all* chunks (for all torrents) pass `null` as the first parameter.

**Parameters**

1. `String|null` - Infohash of specific torrent (if any) for which to iterate across chunks
2. `Function` - Function to be called on each chunk, called with the chunk key as first parameter and an object with the `infoHash` and `index` values parsed from the key as the second parameter.

**Returns**

`Promise` returns `Array` - Returned values after calling function on each chunk

---

### getCachedData

``` js
const data = await getCachedData(infoHash, {
	length: 200000
});
```

Get a torrent's data from the local store as a single blob/buffer. If all chunks are not present (i.e. torrent is partially downloaded, return `undefined`).

**Parameters**

1. `String` - Infohash of torrent
2. `Object` - Params:
	- `length` - `Number` - Byte length of data (required)
	- `pieceLength` - `Number` - Torrent piece length (default `16384`)
	- `buffer` - `Boolean` - If true, data is returned as `Buffer`, else `Blob` (default `false`)

**Returns**

`Promise` returns `Blob|Buffer|undefined` - Torrent data

---

### listCache

``` js
const cached = await listCache();
console.log(cached[0]);
// {
// 	 cachedBytes: 4246111
// 	 cachedPieces: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, . . . ],
// 	 fileName: '@alice myvideo.mp4',
// 	 infoHash: 'cf8524a74b5dd020dc1e4f347b557528b12f2496',
// 	 lastPieceLength: 2655,
// 	 length: 4246111,
// 	 numPieces: 260,
// 	 pieceLength: 16384,
// 	 info: {
// 		 name: <Uint8Array(28)>,
// 		 length: 4246111,
// 		 'piece length': 16384,
// 		 pieces: <Uint8Array(5200)>
// 	 }
// }
```

Get an inventory of the data in the local cache.

**Returns**

`Promise` returns `Array` - Array of objects with information about each torrent in the local cache

---

### synchronizeDirectory

``` js
await client.synchronizeDirectory(10325047);
```

Build user directory from blockchain data. The client's user directory can be accessed at `client.earth.directory`.

**Parameters**

1. `Number` - Maximum block number to which directory should synchronize

**Returns**

`Promise`

---

### cacheTorrentChunk

``` js
await client.cacheTorrentChunk(data, {
	infoHash: '3226d40f4c197f7450e0c768d9dfc3f0a3f254d3',
	index: 0
});
```

Low-level method used to cache a torrent chunk.

**Parameters**

1. `Uint8Array` - Torrent chunk data
2. `Object` - Params
	- `infoHash` - `String` - Infohash of torrent chunk belongs to
	- `index` - `Number` Ordinal position of torrent chunk in data

**Returns**

`Promise`

---

### removeTorrentChunk

``` js
await client.removeTorrentChunk(data, {
	infoHash: '3226d40f4c197f7450e0c768d9dfc3f0a3f254d3',
	index: 0
});
```

Low-level method to delete a chunk from the store.

**Parameters**

1. `Object` - Params
	- `infoHash` - `String` - Infohash of torrent chunk belongs to
	- `index` - `Number` Ordinal position of torrent chunk in data

**Returns**

`Promise`

---

### getTorrent

``` js
const torrent = client.getTorrent('3226d40f4c197f7450e0c768d9dfc3f0a3f254d3');
```

Get model of an active torrent from the internal WebTorrent instance. If there are no active torrents with that infohash, return `null`.

**Parameters**

1. `String` - Infohash of torrent

**Returns**

`Object` - WebTorrent's torrent model

---

### getObjectUrl

``` js
const url = await client.getObjectUrl('3226d40f4c197f7450e0c768d9dfc3f0a3f254d3');
console.log(url); // 'blob:https://mysite.com/b4b33369-6216-40c9-91b0-0af15800d92e'
```

Get an object url to display a torrent's data in the browser.

!!! Note
	To avoid memory leaks, your application should revoke the object url when it's no longer needed.

**Parameters**

1. `String` - Infohash of torrent

**Returns**

`Promise` returns `String` - Object URL

## Client Events

### contact

Emitted when the client successfully initializes the current epoch upon receiving data from the remote world server.

**Data**

- `world` - `String` - Name of world client contacted
- `current` - `Epoch` - The world's current epoch
- `history` - `Array` - The world's previous epochs
- `tracking` - `Array` - List of states client will build
- `endpoint` - `String` - URL of the remote world server

---

### contact_failed

Emitted when the client fails to contact the remote world server, for whatever reason.

**Data**

- `world` - `String` - Name of world client attempted to contact
- `error` - `Object` - The error object

---

### state_initialized

Emitted when a state that the client is tracking is ready.

**Data**

- `world` - `String` - Name of the world in which state exists
- `state` - `State` - Model of the state
- `epochNumber` - `Number` -  World's current epoch number

---

### torrent_added

Emitted when a torrent becomes active.

**Data**

- `torrent` - `Object` - WebTorrent's model of the torrent
- `loaded` - `Number` - The number of bytes already loaded

---

### torrent_stopped

Emitted when a torrent is paused.

**Data**

- `infoHash` - `String` - Infohash of the torrent

---

### torrent_removed

Emitted when a torrent is stopped and its data removed.

**Data**

- `infoHash` - `String` - Infohash of the torrent

---

### torrent_complete

Emitted when a torrent is finished downloading. Note that this event is *always* emitted, even if all of the torrent's data was already cached locally when it was added.

**Data**

- `torrent` - `Object` - WebTorrent's model of the torrent
- `data` - `Uint8Array` - The torrent's data

---

### data_loaded

Emitted when the client receives a chunk of torrent data.

**Data**

- `torrent` - `Object` - WebTorrent's model of the torrent
- `bytes` - `Number` - The size of the data chunk that was loaded
- `loaded` - `Number` - The total bytes of this torrent that have been loaded

---

### data_cached

Emitted when a received chunk of torrent data has been saved in the local cache.

**Data**

- `torrent` - `Object` - WebTorrent's model of the torrent
- `bytes` - `Number` - The size of the data chunk that was loaded
- `index` - `Number` - The ordinal position of the chunk in torrent's data

---

### data_sent

Emitted when the client sends a chunk of torrent data to another peer.

**Data**

- `torrent` - `Object` - WebTorrent's model of the torrent
- `bytes` - `Number` - The size of the data chunk that was sent
