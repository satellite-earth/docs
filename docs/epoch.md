# @satellite-earth/epoch

!!! Note
	The `Epoch` class, in addition to the following, inherits the functionality of the `Torrent` and `Message` class.

## Usage

``` js
const Epoch = require('@satellite-earth/epoch');

// . . .

const epoch = new Epoch(meta);
```

**Constructor**

1. `Object` - Epoch meta data
	- `name` - `String` - typeof options.name === 'undefined' ? this.name : options.name,
	- `number` - `Number` - Ordinal epoch number
	- `ancestor` - `String` - Previous epoch's `uuid`
	- `alpha` - `Number` - Ethereum block number at which epoch began
	- `omega` - `Number` - Ethereum block number at which epoch ended (only defined if loading previously finalized epoch)

## Properties

### signals

``` js
epoch.signals
```

`Array` - `Signal` objects contained within the epoch

---

### states

``` js
epoch.states
```

`Array` - `State` objects built from epoch's signals

---

### initial

``` js
epoch.initial
```

`Array` - `State`s that existed at the beginning of epoch

---

### ancestor

``` js
epoch.ancestor // '4ab983d73d502bd027e97cdba7fcad7e695fd514'
```

`String` -  Previous epoch's `uuid` (`'genesis'` if epoch number `0`)

---

### signer

``` js
epoch.signer // 'alice'
```

`String` - Alias name of epoch signer

---

### number

``` js
epoch.number // 3
```

`Number` - Ordinal epoch number

---

### alpha

``` js
epoch.alpha // 10190026
```

`Number` - Ethereum block number at which epoch began

---

### omega

``` js
epoch.omega // 10363972
```

`Number` - Ethereum block number at which epoch ended

---

### finalized

``` js
epoch.finalized // true
```

`Boolean` - If the epoch has ended

---

### compressed

``` js
epoch.compressed
```

`Uint8Array` - Compressed epoch data

## Methods

### data

``` js
await epoch.data(data);
```

Inflate epoch with compressed data. Overrides method on [Torrent](/torrent) superclass.

**Parameters**

1. `Uint8Array` - Epoch torrent data

**Returns**

`Promise` returns `this` - Reference to self allows method chaining

---

### include

``` js
epoch.include(signal);
```

Include a signal in the epoch.

**Parameters**

1. `Signal` - Signal to be included

---

### loadState

``` js
await epoch.loadState(meta, get);
```

Create a new state model or initialize an existing state from compressed data.

**Parameters**

1. `Object` - Meta data for state as defined by `payload` on [State](/state) class
2. `Function` - (optional) async function, if provided, is called with state model and expected to return compressed data to initialize state.

**Returns**

`Promise` returns `this` - Reference to self allows method chaining

---

### build

``` js
await epoch.build({
	initial: [ myState, myOtherState, ... ],
	signals: [ signal1, signal2, ... ]
});
```

Build epoch from initial states and signals.

**Parameters**

1. `Object` - Content of the epoch
	- `initial` - `Array` - Meta data for epochs initial states
	- `signals` - `Array` - Signals to include in the epoch
2. `Function` - (optional) async function, if provided, is called with state model and expected to return compressed data to initialize state.

**Returns**

`Promise` returns `this` - Reference to self allows method chaining

---

### drop

``` js
const dropped = await epoch.drop('1cf5fc421006a70d06a11fe8c5441f1b8a97a28d');
```

Remove a signal from the epoch. States are automatically rebuilt without dropped signal.

**Parameters**

1. `Signal|Array` - `uuid`(s) of signal(s) to drop
2. `Function` - (optional) async function, if provided, is called with state model and expected to return compressed data to initialize state.

**Returns**

`Promise` returns `Array` - Signals that were dropped

---

### finalize

``` js
await epoch.finalize(10363972);
```

Finalize the epoch, build states and compute data params in preparation for a new epoch.

**Parameters**

1. `Number` - Final block number

**Returns**

`Promise` returns `this` - Reference to self allows method chaining

---

### next

``` js

console.log(epoch.number); // 3
const nextEpoch = await epoch.next();
console.log(nextEpoch.number); // 4
```

Create a successive epoch.

**Parameters**

1. `Object` - Options
	- `name` - `String` - Alias name of the new epoch singer. By default, inherit from immediate ancestor.

**Returns**

`Promise` returns `Epoch` - The new epoch
