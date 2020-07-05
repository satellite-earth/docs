# @satellite-earth/torrent

!!! Note
	The `Torrent` class, in addition to the following, inherits the functionality of the `Message` class.

## Usage

``` js
const Torrent = require('@satellite-earth/torrent');

// Create a torrent model
const torrent = new Torrent({
	name: 'mytorrent.png', 
	foo: 'bar'
});

// Compute torrent "meta" data 
await torrent.data(file);

console.log(torrent.payload);
// {
// 	_signed_: {
// 		name: 'mytorrent', 
// 		foo: 'bar',
// 		meta: '9437.nTrcKwmq8R39tFoR0DNvYhcLsq4=.QmaXEemWbXdysM3E9pj6sdM2LBcoW2NUUp1GzgAsNgrFCp'
// 	},
// 	_params_: {}
// }
```

**Constructor**

1. `Object|String` - Message payload. Can be object or uri-encoded.
2. `Object` - Options
	- `deriveName` - `Function` - If provided, called with `this` and expected to return `name` property. Useful for creating torrent models in which the name is derived from other properties instead of being explicitly contained in the signed data.

## Properties

### name

``` js
torrent.name; // 'mytorrent'
```

`String` - Torrent's name, as set in constructor. Default `Untitled`.

---

### extension

``` js
torrent.extension; // 'png'
```

`String` - File extension of torrent data, if any (based on `name`)

---

### length

``` js
torrent.length; // 255124
```

`Number` - Byte length of torrent data

---

### pieces

``` js
torrent.pieces; // [ '77412139e061b76cbe626d806bc748b350b390ef', ... ]
```

`Array` - SHA1 hashes of each piece of torrent data

---

### numPieces

``` js
torrent.numPieces; // 16
```

`Number` - Number of pieces torrent data was split into

---

### pieceLength

``` js
torrent.pieceLength; // 16384
```

`Number` - Byte length of torrent pieces

---

### lastPieceLength

``` js
torrent.lastPieceLength; // 9364
```

`Number` - Byte length of last torrent piece

---

### info

``` js
torrent.info
```

`Object` - BitTorrent info dict

- `name` - `Uint8Array` - Torrent's name
- `length` - `Number` - Byte length of torrent data
- `piece length` - `Number` - Byte length of torrent pieces
- `pieces` - `Uint8Array` - SHA1 hashes of each piece of torrent data

---

### infoHash

``` js
torrent.infoHash; // 'd92cc55bd412a34f3bb3caf2878707963954fd52'
```

`String` - Torrent's infohash

---

### ipfsHash

``` js
torrent.ipfsHash; // 'QmaXEemWbXdysM3E9pj6sdM2LBcoW2NUUp1GzgAsNgrFCp'
```

`String` - IPFS multihash of torrent's data

## Methods

### data

``` js
await torrent.data(file);
console.log(torrent._signed_.meta);
// 9437.nTrcKwmq8R39tFoR0DNvYhcLsq4=.QmaXEemWbXdysM3E9pj6sdM2LBcoW2NUUp1GzgAsNgrFCp
```

Populate torrent model with computed torrent `meta` data, concatenated as `<length>.<SHA1>.<IPFS>` and saved in the torrent's `_signed_` object.

**Parameters**

1. `Uint8Array|Blob` - Torrent data

**Returns**

`Promise` returns `this` - Reference to self allows method chaining

---

### clearTorrentParams

``` js
torrent.clearTorrentParams();
console.log(torrent._signed_.meta); // undefined
```

Remove existing torrent meta data from model

---

### getParsedTorrent

``` js
const parsed = getParsedTorrent();
console.log(parsed);
// {
// 	length,
// 	info,
// 	infoHash,
// 	infoBuffer,
// 	name,
// 	infoHashBuffer,
// 	announce,
// 	urlList,
// 	lastPieceLength,
// 	pieceLength,
// 	pieces,
// 	files: [ ... ]
// }
```

Get object representing the torrent in the format expected by WebTorrent

**Parameters**

1. `Object` - Options
	- `announce` - `Array` - Array of torrent trackers
	- `urlList` - `Array` - Array of http webseeds

**Returns**

`Object` - The torrent object

---

### getTorrentFile

``` js
const torrentFile = torrent.getTorrentFile();
```

Get standard bencoded `.torrent` file in the format expected by any torrent client

**Parameters**

1. `Object` - Options
	- `announce` - `Array` - Array of torrent trackers
	- `urlList` - `Array` - Array of http webseeds

**Returns**

`Uint8Array` - The `.torrent` file
