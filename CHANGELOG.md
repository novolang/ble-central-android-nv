# Changelog

Every published version, newest first.  This file is on the publish allow-list, so it travels with the package.

## Unreleased

- `AndroidCentral.request_phy`, `read_phy` and `link_info`: the PHYs of a link asked for as masks (1M, 2M, coded) and read, and what the platform told of the radio of a link, as
  [interval in microseconds, latency, supervision timeout in milliseconds, transmit PHY, receive PHY, MTU].  The trait has no call for them: they are the implementation's own, and
  advisory like the priority.  The interval comes from `onConnectionUpdated`, a hidden callback of the framework, so it can stay 0 on a phone that does not call it.
- `AndroidCentral.last_event_us` and `now_us`: the time the platform's thread posted the event `next_event` returned last, and the time now, on one clock, for timing a request.

## 0.1.0

First release.

- `ble_android`: `AndroidCentral`, the implementation of `GattCentral` for an app built with `novo mobile build`: scan with filters, connect, discovery after a refresh of
  the platform's cache, read, write with and without a response, notifications and indications, the MTU and the longest write, the connection priority, a bond as a
  request, and every link loss as an event.
- `decode`: the events and values of ble-central-nv made from what the runtime reports, and the table of links.
