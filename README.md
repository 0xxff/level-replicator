
![img](/replicator.png)

# SYNOPSIS
A simple eventually consistent master-master replication module 
for leveldb.

# BUILD STATUS
[![build-status](https://www.codeship.io/projects/0d604520-6cc1-0131-203c-22ccfa4c21c9/status)](https://www.codeship.io/projects/13128)

## METHOD
### Change Logs
Each database maintains a `CHANGE LOG` in a separate sub-level.
Each entry in the change log has a key that matches each key in
the database appended by a lamport timestamp.

```
some-random-key-name!2342341
```

```
put
```

![img](/closeup.png)

A replicating database will query a remote database's change log in reverse 
until it finds either a matching key in its own change log or the first key 
in the remote server's change log.

![img](/faraway.png)

### Optimizations
Keeping a changes log and applying the changes in reverse means the latest
changes will be applied to the appropriate key values. If there is more than
one operation against a key,

  1. the change entry is removed by the server (*Not yet implemented*) or
  2. it is ignored by the client.

The changes log can be truncated over time to save disk space.

## EXAMPLE: MORE THAN TWO SERVERS

### Server 1
```js
var level = require('level')
var rep = require('level-replicator')

var config = {
  servers: { // list of servers to replicate with
    "127.0.0.1:8001": {}, // a serer id and its meta data
    "127.0.0.1:8002": {} // a serer id and its meta data
  },
  port: 8000 // port for this server
}

var db = rep(level('/tmp/db'), config) 

// put something into the database
db.put('some-key', 'some-value', function(err) {
})
```

### Server 2

```js
var level = require('level')
var lrep = require('level-replicator')

var config = {
  servers: {
    "127.0.0.1:8000": {},
    "127.0.0.1:8002": {}
  },
  port: 8001
}

var db = rep(level('/tmp/db'), config) 

db.put('some-key', 'some-value', function(err) {
})
```

### Server 3...

```js
var level = require('level')
var lrep = require('level-replicator')

var config = {
  servers: { 
    "127.0.0.1:8000": {},
    "127.0.0.1:8001": {} 
  },
  port: 8002
}

var db = lrep(level('/tmp/db'), config) 

db.put('some-key', 'some-value', function(err) {
})
```


## EXAMPLE: SECURE PEERS
Secure peers require a key pair; if you want to be fancy you could use 
[`this`][0] or [`this`][1] to crete a JSON file that includes the generated 

```js

var config = {
  servers: {
    "127.0.0.1:8001": {} 
  },
  port: 8000,
  pems: __dirname + '/pems.json'
}

config.identify = function (id) {

  //
  // you can asynchronously verify that the key matches the known value here
  //
  id.accept()
}
```

[0]:https://github.com/hij1nx/selfsigned
[1]:https://github.com/substack/rsa-json
