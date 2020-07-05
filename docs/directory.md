# @satellite-earth/directory

An in-memory directory of alias IDs built from Etheruem log data to faciliate the synchronous verification of signed messages.

!!! Note
	The `Directory` instance contained within the `Core` API is preferred for general usage — this standalone module is provided to support composability in other use cases.

## Usage

``` js
const Directory = require('@satellite-earth/directory');
const { Earth } = require('@satellite-earth/core');

// Construct a directory instance
const directory = new Directory();

// Earth provides blockchain interface
const earth = new Earth();
await earth.connect();

// Read blockchain logs to populate alias IDs
await directory.synchronize(earth);
```

**Constructor**

1. `Function` - (optional) If provided, function will be called with the `alias` name of each record, and only upon returning a truthy value will that alias record be included. Useful for building directory for a subset of users.

## Properties

### initialized

``` js
directory.initialized
```

`Boolean` - If directory has been populated

---

### blockNumber

``` js
directory.blockNumber
```

`Number|null` - Latest block number to which directory has synchronized. `null` if directory is uninitialized.

---

### compressed

``` js
directory.compressed
```

`Uint8Array` - Compressed directory data. Useful for caching state so it doesn't have to rebuilt on every restart.

## Methods

### inflate

``` js
directory.inflate(data);
```

Repopulate alias records with previously cached data

**Parameters**

1. `Uint8Array` - The compressed data

---

### synchronize

``` js
await directory.synchronize(earth);
```

Populate alias records from contract log data

**Parameters**

1. `Core` - Core API instance user to access contract data
2. `Number` - (optional) Max block number from which to populate records

**Returns**

---

### list

``` js
const list = directory.list({ utf8: true });
console.log(list[0]);
// {
// 	alias: 'alice'
// 	primary: '0x19646E56d36615A1A723650a2c65E4311D84bE70',
// 	recovery: '0x85D8Ae333D2e4CFDF478d891658B1e23DF924103'
// }
```

Get a list of users in the directory based on most up-to-data records

**Parameters**

1. `Object` - Options
	`utf8` - `Boolean` - If alias names should be returned as utf8 instead of hex string (default `false`)

**Returns**

---

### getAlias

``` js
const name = directory.getAlias('0x19646E56d36615A1A723650a2c65E4311D84bE70', {
	at: 10190780,
	utf8: true
});

console.log(name); // 'alice'
```

Get alias name that address was linked to at given block number

**Parameters**

1. `String` - Primary address
2. `Object` - Options
	- `at` - `Number` - Block number at which to select record (default `this.blockNumber`)
	- `utf8` - `Boolean` - If alias name should be returned as utf8 instead of hex string (default `false`)

**Returns**

`String` - Alias name

---

### getPrimary

``` js
const primary = directory.getPrimary('alice', { utf8: true });
console.log(primary); // '0x19646E56d36615A1A723650a2c65E4311D84bE70'
```

Get alias name's primary address, if any

**Parameters**

1. `String` - Alias name
2. `Object` - Options
	- `at` - `Number` - Block number at which to select record (default `this.blockNumber`)
	- `utf8` - `Boolean` - If alias name should be returned as utf8 instead of hex string (default `false`)

**Returns**

`String` - Primary address

---

### getRecovery

``` js
const recovery = directory.getRecovery('alice', { utf8: true });
console.log(recovery); // '0x85D8Ae333D2e4CFDF478d891658B1e23DF924103'
```

Get alias name's recovery address, if any

**Parameters**

1. `String` - Alias name
2. `Object` - Options
	- `at` - `Number` - Block number at which to select record (default `this.blockNumber`)
	- `utf8` - `Boolean` - If alias name should be returned as utf8 instead of hex string (default `false`)

**Returns**

`String` - Recovery address

---

### getRecords

``` js
const records = directory.getRecords('alice', { utf8: true });

console.log(records[0]);
// alice created id at block 10190780
// {
// 	blockNumber: 10190780
// 	primary: '0x19646E56d36615A1A723650a2c65E4311D84bE70',
// 	recovery: '0x85D8Ae333D2e4CFDF478d891658B1e23DF924103'
// }

console.log(records[1]);
// alice changed primary address at block 10194321
// {
// 	blockNumber: 10194321
// 	primary: '0x578EC0C952554f26de5aa80DC3D853F9f76ec81D'
// }
```

Get entire address history for a given alias

**Parameters**

1. `String` - Alias name
2. `Object` - Options
	- `utf8` - `Boolean` - If alias name being queried is utf8 (default `false` i.e. hex)

**Returns**

`Array` - Array of objects representing new values of alias name's `primary` or `recovery` address at `blockNumber`
