NIP-XX
======

Probe
-----

`draft` `optional`

The `PROBE` command allows clients to find out whether a given relay would accept a given event.

```json
-> ["PROBE", "<event JSON>"]
<- ["OK", "<event id>", "restricted: user is not a member"]
```

To avoid unnecessary signer confirmations, the sent event MAY be unsigned but should be handled as if it were.
