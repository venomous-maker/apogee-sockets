# Changelog

Every release of apogee-sockets, newest first. Each is a `vX.Y.Z` tag with a GitHub
release carrying its section; the `grav` line in `grav.toml` says which
compiler versions it builds with.

## Unreleased

## 0.1.1 (2026-11-21)

### Performance

- An event is encoded once per channel instead of once per recipient: fanning
  one out to 100 subscribers takes 43% fewer instructions, and delivery went
  from about 84,000 to about 104,000 messages a second.
- Each app's channels have their own lock, so busy apps do not wait on each
  other.

## 0.1.0 (2026-11-20)

The first release. Needs grav 1.10.0 or newer.

- `SocketsServer`: the Pusher Channels protocol (7) over WebSockets: public,
  private and presence channels, subscription signatures, client events,
  pings, connection quotas, allowed origins.
- The signed HTTP API: triggering events (one, a batch, leaving out a socket)
  and asking about channels and presence users; a health check.
- Works with the official pusher-js client and the pusher server library.
