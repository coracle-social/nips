NIP-XX
======

Rooms
-----

`draft` `optional`

This NIP defines rooms, which are a way of subdividing a single relay into multiple spaces. Rooms with the same ID on different relays SHOULD be considered different rooms.

This spec is purposely anarchic, for example the same room may be defined by multiple people. Client-side resolution of these inconsistencies is left intentionally undefined. See the (Relay support)[#relay-support] section for ways in which relays may resolve these inconsistencies by means of progressive enhancement.

## Defining a room

A room is identified by a short random string, for example `9S6F1A`. Anyone who wishes to create a room SHOULD publish a `kind 30300` room metadata event:

- `d` is the room id
- `name` is the name of the room
- `about` is a description of the room
- `picture` is an image url representing the room
- If a `hidden` tag is present, it indicates that the room should not be advertised to non-members
- If a `private` tag is present, it indicates that non-members are not allowed to view room content
- If a `closed` tag is present, it indicates that non-members are not allowed to post to the room

```yaml
{
  "kind": 30300,
  "content": "",
  "tags": [
    ["d", "9S6F1A"],
    ["name", "Pizza Lovers"],
    ["picture", "https://pizza.com/pizza.png"],
    ["about", "a room for people who love pizza"],
    ["closed"]
  ],
}
```

## Membership

A `kind 30301` event indicates room membership. Clients should request this event in order to determine the user's membership status. The membership event's author MUST be the same as the room metadata's author.

```yaml
{
  "kind": 30301,
  "content": "",
  "tags": [
    ["d", "9S6F1A"],
    ["p", "6cdb0d971e488334e1fec57fff22e3615f98fea060c9eb5a57d248b46b06999a"],
    ["p", "62f11f9fb616a5fdd83a4694da1eb676e226cc8861d446f455be344d2eb46f01"]
  ]
}
```

Users MAY request access to a room using a `kind 301` event:

```yaml
{
  "kind": 301,
  "content": "",
  "tags": [
    ["h", "9S6F1A"]
  ]
}
```

Users MAY request to be removed from room membership (or cancel an access request) using a `kind 302` event:

```yaml
{
  "kind": 302,
  "content": "",
  "tags": [
    ["h", "9S6F1A"]
  ]
}
```

## Membership lists

Users MAY track rooms they are a member of using a [NIP 51](./51.md) `kind 10300` list.

- A `room` tag with room ID and relay url SHOULD be included for each room the user is (or intends to be) a member of
- An `r` tag SHOULD be included for each relay the user is (or intends to be) a member of

These should not be treated as canonical member lists, but are intended to allow users to customize navigation in clients, and provide a social signal for rooms the user wants to recommend to people in their social graph.

```yaml
{
  "kind": 10300,
  "content": "",
  "tags": [
    ["room", "9S6F1A", "wss://relay.example.com/"],
    ["room", "82HLOO", "wss://relay.example.com/"],
    ["r", "wss://relay.example.com/"]
  ]
}
```

## Posting to a room

Events posted to a room MUST use special event kinds to avoid being matched by queries that do not specify an `h` tag.

Room events MUST:

- Use kind `310`, `10310`, `20310`, or `30310` depending on the kind range it is emulating
- Include a `~` tag equal to the stringified event kind it is emulating.
- Include an `h` tag equal to the room ID

For example, if I wanted to post a `kind 1` to room `9S6F1A`, I would format it this way:

```yaml
{
  "kind": 310,
  "content": "Happy #birthday",
  "tags": [
    ["p", "469b89e25c0e9953f62ac0055a15fc6b3ffca67b72fb5c88fdda4949543015e9", 'wss://relay.example.com/'],
    ["t", "birthday"],
    ["h", "9S6F1A"],
    ["~", "1"]
  ]
}
```

A calendar event posted to a room would use a `kind 30310` instead of `kind 31922`:

```yaml
{
  "kind": 30310,
  "content": "Come hang out",
  "tags": [
    ["d", "98W2KJLS9"],
    ["title", "My event"],
    ["start", "2025-01-01"],
    ["end", "2025-01-01"],
    ["h", "9S6F1A"],
    ["~", "31922"]
  ],
}
```

A `kind 0` posted to a room would use a `kind 10310` because they act like regular replaceables:

```yaml
{
  "kind": 10310,
  "content": "{...profile meta}",
  "tags": [
    ["h", "9S6F1A"],
    ["~", "0"]
  ],
}
```

## Relay support

Relays MAY enhance room support by implementing read/write access controls, room metadata re-writes, and custom error messages, as long as enhancements are implemented by means of standard nostr protocol affordances (AUTH, OK, CLOSED, EVENT, REQ, etc).

Here are some ways a relay might progressively enhance room support:

- When a conflicting room metadata event is published, reject it with an OK message
- When a room metadata event is deleted, delete all events published to it
- Automatically update room membership when join/leave requests are received
- Hide hidden room metadata, membership, and content from non-members
- Refuse to serve content from private rooms to non-members
- Reject posts from non-members for closed rooms
