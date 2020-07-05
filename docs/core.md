# @satellite-earth/core

## Usage

**Browser**
``` js
const { Earth } = require('@satellite-earth/core');

// Check that user has an Ethereum wallet installed
if (!window.ethereum) {
	alert('Please install MetaMask');
}

const earth = new Earth();

// Connect using injected provider
await earth.connect();
```

**Node**

``` js
const { Earth } = require('@satellite-earth/core');

const earth = new Earth();

// Connect using HttpProvider (e.g. Infura)
await earth.connect({
	httpProviderUrl: PROVIDER_URL,
	httpBasicAuthParams: { password: PASSWORD }
});
```

## Properties

### clock

``` js
earth.clock
```

`Clock` - Internal Ethereum clock (see documentation for [clock](/clock))

---

### directory

``` js
earth.directory
```

`Directory` - Internal user directory (see documentation for [directory](/directory))

---

### web3

``` js
earth.web3
```

`Object` - Expose Web3 API for general-purpose blockchain ops (see documenation for [Web3](https://web3js.readthedocs.io/))

---

### contract

``` js
earth.contract
```

`Object` - Expose low-level contract model for advanced ops

---

### deployed

``` js
earth.deployed // 9975903
```

`Number` - The block number at which the contract was deployed to the blockchain

---

### address

``` js
earth.address // '0x7C9ed09cCb6723Fc42FBd9c5a83420a3D8fFCbE4'
```

`String` - The address of the contract on the blockchain

## Methods

### connect

``` js
await earth.connect();
```

Initialize the API and connect to database of users. In the browser, earth automatically uses `window.ethereum` if it's available. In a Node environment, you'll need to supply a provider endpoint (see [*Usage*](#usage) above).

**Parameters**

1. `Object` - Options
	- `useOwnProvider` - `Boolean` - Don't use `window.ethereum` provider, even if it's available in the browser (default `false`)
	- `httpProviderUrl` - `String` - endpoint of your provider
	- `httpBasicAuthParams` - `Object` - Object with `username` and `password` if provider requires HTTP Basic Auth

**Returns**

`Promise`

---

### createID

``` js
earth.createID({
	alias: 'alice',
	primary: '0x19646E56d36615A1A723650a2c65E4311D84bE70',
	recovery: '0x85D8Ae333D2e4CFDF478d891658B1e23DF924103'
}, (event) => {
	// Handle tx events
});
```

Create a new ID. `alias` name and `primary` address are required, while `recovery` is optional.

**Parameters**

1. `Object` - Object containing `alias`, `primary`, and optionally `recovery` (can be set by user later)
2. `Function` - Function to handle transaction events. See [*Transaction Events*](#transaction-events).

**Returns**

`Promise`

---

### setPrimary

``` js
earth.setPrimary({
	alias: 'alice',
	newPrimary: '0x7034412De72956e1105134a35b2f23f6b071010d'
}, (event) => {
	// Handle tx events
});
```

Set a new primary address. Must be sent from user's current primary address.

**Parameters**

1. `Object` - Object containing `alias`, `newPrimary`, and optionally `from` (earth will attempt to auto-detect user's current address)
2. `Function` - Function to handle transaction events. See [*Transaction Events*](#transaction-events).

**Returns**

`Promise`

---

### setRecovery

``` js
earth.setRecovery({
	alias: 'alice',
	newRecovery: '0x7034412De72956e1105134a35b2f23f6b071010d'
}, (event) => {
	// Handle tx events
});
```

Set a new recovery address. Must be sent from user's current primary address.

**Parameters**

1. `Object` - Object containing `alias`, `newRecovery`, and optionally `from` (earth will attempt to auto-detect user's current address)
2. `Function` - Function to handle transaction events. See [*Transaction Events*](#transaction-events).

**Returns**

`Promise`

---

### recover

``` js
earth.recover({
	alias: 'alice',
	recovery: '0x85D8Ae333D2e4CFDF478d891658B1e23DF924103',
	newRecovery: '0x7034412De72956e1105134a35b2f23f6b071010d'
}, (event) => {
	// Handle tx events
});
```

Recover user's ID (current recovery address becomes primary address). Must be sent from user's current recovery address. User has the option to set a new recovery address in the same transaction.

**Parameters**

1. `Object` - Object containing `alias`, `recovery`, and optionally `newRecovery`
2. `Function` - Function to handle transaction events. See [*Transaction Events*](#transaction-events).

**Returns**

`Promise`

---

### nameAvailable

``` js
const available = await earth.nameAvailable('alice');
console.log(available); // false
```

Check if a name is available for registration

**Parameters**

1. `String` - Alias ID

**Returns**

`Promise` returns `Boolean` - If name is available

---

### addressAvailable

``` js
const available = await earth.addressAvailable('0x19646E56d36615A1A723650a2c65E4311D84bE70');
console.log(available); // false
```

Check if an address may be associated with an ID (addresses may only ever be associated with one ID)

**Parameters**

1. `String` - Address

**Returns**

`Promise` returns `Boolean` - If address is available

---

### lookupName

``` js
const user = earth.lookupName('jenny');
console.log(user);
// {
// 	primary: '0x3DC747f75CDF06FEDAd5B1AE69ED3AF3328a1CF2',
// 	recovery: '0x017761369D08ad71166849b771a301B3e2079BC8',
// 	joined: 1593201439
// 	number: 8675309
// }
```

Lookup user info by alias ID. Returns `null` if user does not exist.

**Parameters**

1. `String` - Alias ID

**Returns**

`Promise` returns `Object` - User

- `primary` - `String` - User's primary address
- `recovery` - `String` - User's recovery address (empty string if no recovery)
- `joined` - `Number` - Timestamp of block containing transaction that created user
- `number` - `Number` - Alias number (unique ordinal position among all users)

---

### lookupNumber

``` js
const user = await earth.lookupNumber(8675309);
console.log(user);
// {
// 	primary: '0x3DC747f75CDF06FEDAd5B1AE69ED3AF3328a1CF2',
// 	recovery: '0x017761369D08ad71166849b771a301B3e2079BC8',
// 	joined: 1593201439
// 	name: 'jenny'
// }
```

Lookup user info by number. Returns `null` if user does not exist.

**Parameters**

1. `Number` - Alias number
2. `Object` - Options
	- `hex` - Boolean - If `name` should be returned as hex string (default `false`)

**Returns**

`Promise` returns `Object` - User

- `primary` - `String` - User's primary address
- `recovery` - `String` - User's recovery address (empty string if no recovery)
- `joined` - `Number` - Timestamp of block containing transaction that created user
- `name` - `String` - Alias name

---

### lookupAddress

``` js
const name = await earth.lookupAddress('0x3DC747f75CDF06FEDAd5B1AE69ED3AF3328a1CF2');
console.log(name); // 'jenny'
```

Lookup the name for which given address is currrently primary. If `includePast` option is true, returns name that address has *ever* been associated with.

**Parameters**

1. `String` - Address
2. `Object` - Options
	- `includePast` - `Boolean` - Return name that address has *ever* been associated with (default `false`)
	- `hex` - `Boolean` -  If name should be returned as hex string (default `false`)

**Returns**

`Promise` returns `String` - Alias name

---

### packData

``` js
const packed = earth.packData({
	foo: 'bar'
});
```

Pack data to be signed as per [EIP-712](https://eips.ethereum.org/EIPS/eip-712)

**Parameters**

1. `Object` - Data to to be packed
2. `Array` - (optional) EIP-712 domain expressed as array of objects containing `name`, `type`, and `value`

**Returns**

`Object` - Packed data

---

### addressData

``` js
// The data object must contain the top 
// level keys "_signed_" and "_params_"
const authorAddress = earth.addressData({
	_signed_: {
		foo: 'bar'
	}
	_params_: {
		sig: '3b58748191ff2bbf98cdfbb63710babe030bc1226462b6d1d01c54e581fea3ee43f4b5cda4f68c86795d2ba3e6d480fc2155fbe86839ad2855df389f593bef971c'
	}
});

console.log(authorAddress); // '0x19646E56d36615A1A723650a2c65E4311D84bE70'
```

Decode signature to find the address that signed data.

**Parameters**

1. `Object` - Data object must contain `_signed_` and `_params_` (including signature `sig`)
2. `Array` - (optional) EIP-712 domain expressed as array of objects containing `name`, `type`, and `value`

**Returns**

`String` - Address that signed data

---

!!! Note
	For the following three methods (`verifyData`, `verifyDataSync`, and `signData`) the user's `alias` name is represented as a hex string. You can convert to utf8 using `web3.utils.hexToUft8('0x' + hex)`

### verifyData

``` js
// The data object must contain the top 
// level keys "_signed_" and "_params_"
const author = await earth.verifyData({
	_signed_: {
		foo: 'bar'
	}
	_params_: {
		sig: '3b58748191ff2bbf98cdfbb63710babe030bc1226462b6d1d01c54e581fea3ee43f4b5cda4f68c86795d2ba3e6d480fc2155fbe86839ad2855df389f593bef971c'
	}
});

console.log(author); // { alias: '616c6963650a', address: '0x19646E56d36615A1A723650a2c65E4311D84bE70' }
```

Decode signature and reference blockchain to find the data's author

**Parameters**

1. `Object` - Data object must contain `_signed_` and `_params_` (including signature `sig`)
2. `Array` - (optional) EIP-712 domain expressed as array of objects containing `name`, `type`, and `value`

**Returns**

`Promise` returns `Object` - Object containing author's `address` and `alias` name hex encoded

---

### verifyDataSync

``` js
const data = {
	_signed_: {
		foo: 'bar'
	}
	_params_: {
		sig: '3b58748191ff2bbf98cdfbb63710babe030bc1226462b6d1d01c54e581fea3ee43f4b5cda4f68c86795d2ba3e6d480fc2155fbe86839ad2855df389f593bef971c'
	}
};

const blockNumber = 10190780;

// The data object must contain the top 
// level keys "_signed_" and "_params_"
const author = await earth.verifyData(data, blockNumber);

console.log(author); // { alias: '616c6963650a', address: '0x19646E56d36615A1A723650a2c65E4311D84bE70' }
```

Synchronously verify data by referencing internal alias directory. The second parameter (block number) tells the directory at which block (i.e. which point in time) to find the alias linked to the signing address. This is necessary because a user may have changed their address since signing the message.

**Parameters**

1. `Object` - Data object must contain `_signed_` and `_params_` (including signature `sig`)
2. `Number` - Block number at which to verify authorship
3. `Array` - (optional) EIP-712 domain expressed as array of objects containing `name`, `type`, and `value`

**Returns**

`Object` - Object containing author's `address` and `alias` name (hex encoded) at given block number

---

### signData

``` js
let data;

try {

	data = await earth.signData({ foo: 'bar' });

	// . . . waiting for user to confirm signature prompt

	console.log(data);
	// _signed_: {
	// 	foo: 'bar'
	// }
	// _params_: {
	// 	alias: '616c6963650a',
	// 	address: '0x19646E56d36615A1A723650a2c65E4311D84bE70',
	// 	sig: '3b58748191ff2bbf98cdfbb63710babe030bc1226462b6d1d01c54e581fea3ee43f4b5cda4f68c86795d2ba3e6d480fc2155fbe86839ad2855df389f593bef971c'
	// }

} catch (err) {
	// User rejected signature request
}
```

Sign data the browser with the key linked to user's ID

**Parameters**

1. `Object` - Data to sign
2. `Array` - (optional) EIP-712 domain expressed as array of objects containing `name`, `type`, and `value`

**Returns**

`Promise` returns `Object` - Signed data payload with params

---

### getDirectoryLog

``` js
const logs = await earth.getDirectoryLog();
console.log(logs[0]);
// {
// 	transactionIndex,
// 	transactionHash,
// 	blockNumber,
// 	blockHash,
// 	timestamp,
// 	event,
// 	data
// }
```

Get contract event logs

**Parameters**

1. `Object` - Object `fromBlock` (default `9975903`) and `toBlock` (default `'latest'`) expressing interval from which to return logs

**Returns**

`Promise` returns `Object` - Array of objects representing contract events

---

### synchronizeDirectory

``` js
await earth.synchronizeDirectory();
```

Build internal alias directory from contract event logs

**Parameters**

1. `Number` - Maximum block number to which directory should be synchronized (default `'latest'`)

**Returns**

`Promise` returns `Object` - Array of objects representing contract events

---

### synchronizeClock

``` js
await earth.synchronizeClock({ startBlock, toBlock });
```

Populate internal clock with block data directly from the blockchain. By default, the clock starts synchronizing from it's last synced position, or the latest block available.

**Parameters**

1. `Core` - Earth core instance (used to communicate with blockchain)
2. `Object` - Options
	- `startBlock` - `Number` - Block number of last block to sync
	- `toBlock` - `Number` - Block number of first block to sync

**Returns**

`Promise` returns `Object` - New block data

---

### getActiveAddress

``` js
const active = await earth.getActiveAddress();
console.log(active); // '0x19646E56d36615A1A723650a2c65E4311D84bE70'
```

Get currently selected address from browser Ethereum provider

**Returns**

`Promise` returns `String` - User's currently selected Ethereum address

---

### getActiveAlias

``` js
const active = await earth.getActiveAlias();
console.log(active); // 'alice'
```

Get alias linked to active address in browser. Useful for implementing UI showing a user that their alias name has been recognized. Returns empty string if currently selected address does not exists or is not linked to any alias.

**Returns**

`Promise` returns `String` - User's alias name

## Transaction Events

When writing data to the blockchain via the identity API (i.e. calling `createID`, `setPrimary`, `setRecovery`, or `recover`) the second parameter of the method allows you to pass an event handler function so your application can react to various events as the transaction is confirmed by the network. This function is always called with an object containing the `name` of the event and sometimes extra data (see possible events listed below).

!!! Note
	Although implementing a handler for these events is optional, it makes for better UX when the user is able to track the status of their transaction.

### hash

Emitted when the transaction data becomes known. At this point, the tx can be considered "pending".

**Data**

- `name` - `String` - Name of the event
- `data` - `String` - Transaction hash

---

### confirmed

Emitted when the transaction has been confirmed.

**Data**

- `name` - `String` - Name of the event
- `data` - `Object` - Transaction receipt

---

### failed

Emitted if the transaction fails (due to an execution error or being dropped from the tx pool).

**Data**

- `name` - `String` - Name of the event
- `data` - `Object` - Transaction receipt

---

### error

Emitted if the transaction fails for an unknown reason (probably a network error)

**Data**

- `name` - `String` - Name of the event

---

### slow

Emitted after 200 seconds has passed and the transaction has still not been confirmed (probably network congestion or gas price too low)

**Data**

- `name` - `String` - Name of the event

## Exports

In addition the the core API and contract constants, this module exports the raw Application Binary Interface (ABI) as well as a low-level function to get a plain Web3 model of the Satellite contract.

### abi

`String` - JSON encoded Application Binary Interface to the Satellite contract

### Contract

`Object` - Contract constants including `address` and `deployed`

### Earth

`Object` - High-level core API

### getContractInstance

`Function` - Return plain Web3 contract instance
