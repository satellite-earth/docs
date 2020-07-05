# @satellite-earth/state

!!! Note
	The `State` class, in addition to the following, inherits the functionality of the `Torrent` and `Message` class.

## Usage

``` js
const state = new State({
	name: 'mystate',
	established: 10195635
});
```

**Constructor**

1. `Object` - State meta data
	- `name` - `String` - Name of the state
	- `established` - `Number` - Block number at which state was created

## Properties

### store

``` js
state.store
```

`Object` - Arbitrary data model

---

### nucleus

``` js
state.nucleus
```

`Object` - Contains state's `get` and `set` logic

---

### record

``` js
state.record
```

`Object` - Map of signal `uuid` to `blockNumber` at which that signal updated the state

---

### established

``` js
state.established // 1591149129
```

`Number` - Block number at which state was created

---

### updated

``` js
state.updated // 10202992
```

`Number` - Last block number at which state was modified

---

### compressed

``` js
state.compressed
```

`Uint8Array` - Compressed state data

---

### initialized

``` js
state.initialized // true
```

`Boolean` - If `store` has been created

## Methods

### data

``` js
await state.data(data);
```

Build state from compressed data. Overrides method on [Torrent](/torrent) superclass.

**Parameters**

1. `Uint8Array` - State torrent data

**Returns**

`Promise` returns `this` - Reference to self allows method chaining

---

### build

``` js
await state.build();
```

Recompute data params for present state

**Returns**

`Promise` returns `this` - Reference to self allows method chaining

---

### set

``` js
state.set(signal, epoch);
```

Modify state's data store method with signal data as interpreted by the state's internal logic.

**Parameters**

1. `Signal` - Signal object
2. `Object` - Epoch context
	- `signer` - `String` - Alias name of epoch signer
	- `number` - `Number` - Ordinal epoch number
	- `alpha` - `Number` - Block number from which epoch began
3. `Object` - Options
	- `safe` - `Boolean` - If copy of the store should be made prior to modification to prevent unintentional partial modification of the data by setter logic. The tradeoff for using `safe` mode is decreased performance since a copy of the store has to made on every update. Not recommended for production, but useful for debugging.

---

### get

``` js
const myValue = state.get('foo', { bar: 'baz' });
```

Read data from the state's internal store by interfacing with the state's internal getter logic.

**Parameters**

1. `String` - Name of thing to get (passed to internal getter)
2. `Object` - Arbitary query params (passed to internal getter)
3. `Object` - Options
	- `safe` - Boolean - If copy of the store should be made prior to reading data to prevent unintentional modification by getter logic. Comes with a performance hit (see above)

**Returns**

`?` - Unknown return type (depends on internal getter)

---

### on

``` js
state.on('myevent', (data) => {
	// . . .
});
```

Attach an event handler to react to custom events triggered by the internal setter function.

**Parameters**

1. `String` - Name of event
2. `Function` - Handler called with data provided by internal setter
