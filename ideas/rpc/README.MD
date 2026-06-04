# RPC — Remote Procedure Call

## Overview

A lightweight RPC system where local function calls are transparently serialized and dispatched over a network channel.

## Basic Usage

Define a function normally:

```
user createUser(string name) {
    user u = new user();
    u.name = name;
    u.id = users.size++;
    users.push(u);
    return u;
}
```

The RPC layer serializes the call into a binary frame:

```
struct rpc {
    int functionId;
    args args;
}

struct args {
    int size;
    stream *data;
}
```

A router maps function IDs to handlers and handles (de)serialization:

```
val routes = { ::createUserController }

fun handle(rpc *rpc) {
    return routes[rpc.functionId](rpc.args);
}
```

## Callbacks / Back-Calls

For functions that accept callbacks (e.g. streaming or error handling), the callback is itself turned into an RPC call sent back over the same channel:

```
processBigData(user[] users, fun(user, error) onError) {
    for (user in users) {
        try {
            handle(user);
        } catch (e) {
            onError(user, e);
        }
    }
}
```

The `onError` callback is implemented as a reverse RPC frame sent to the caller:

```
fun onError(user u, error e) {
    channel.send(args {
        functionId = context.functionId;
        data = serializeUserError(u, e);
    });
}
```

This allows bidirectional communication over a single channel without extra infrastructure.
