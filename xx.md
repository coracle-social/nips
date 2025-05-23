NIP-XX
======

Virtual Relays
--------------

`draft` `optional`

This NIP defines virtual relays, which are a way of subdividing a single relay into multiple spaces. Virtual relays share their parent relay's URL when relevant, and inherit [NIP 42](./42.md) AUTH state. They are identified by a short string of random characters, for example `8SB0`.

## VCMD

The `VCMD` command accepts the virtual relay's ID along with the arguments for any other command. `VCMD` can be used in either direction, either client -> relay or relay -> client.

```yaml
-> ["VCMD", "8SB0", "REQ", {"kinds": [1]}]
-> ["VCMD", "8SB0", "EVENT", <event data>]
```

## VLIST

The `VLIST` command allows clients to ask a relay what virtual relays exist. Relays must respond with `VITEM` with a single virtual relay ID.

```yaml
-> ["VLIST"]
<- ["VITEM", "8SB0"]
<- ["VITEM", "7BTA"]
```

## VINFO

The `VINFO` command allows clients to ask for the [NIP 11](./11.md) relay information document associated with the given virtual relay.

```yaml
-> ["VINFO", "8SB0"]
<- ["VINFO", {"name": "My virtual relay", "version": "1.0"}]
```

## VADD

The `VADD` command allows for the creation of user-defined virtual relays. The first argument MUST be a partial NIP 11 relay information document to be merged into the parent relay's information document. The relay MUST respond with the virtual relay's ID if the relay was added. The relay MAY respond with an empty ID if the request was denied. No response indicates lack of support.

```yaml
-> ["VADD", {"name": "My virtual relay", "version": "1.0"}]
<- ["VADD", "8SB0"]
```

## VDEL

The `VDEL` command allows for the deletion of user-defined virtual relays. If the relay is not deleted, the relay MAY respond with an empty ID. No response indicates lack of support.

```yaml
-> ["VDEL", "8SB0"]
<- ["VDEL", "8SB0"]
```
