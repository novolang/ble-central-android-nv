# ble-central-android-nv

The Central role of Bluetooth LE on an Android phone, as an implementation of `GattCentral` from [ble-central-nv](../ble-central-nv).

A phone gives a program scan, connect, discover, read, write and subscribe and no radio below them.  An app built with `novo mobile build` reaches them through
the toolchain's own Java classes (a dex of 15 kB that the build adds to an app that calls this package) and a ring in the runtime that the platform's callbacks
fill; this package turns what the ring holds into the events of the trait and the trait's calls into the runtime's.  A program that codes against
`GattCentral` runs here, over the noble Host on a PC, and over a scripted peripheral in a test.

```novo ignore
use central
use session
use ble_android

fn main() [io, ffi, mutate, time]
    let phone = ble_android.central()
    match phone.start()
        Some(e) => println(e.message())
        None =>
            match session.open_link(phone, session.pump(), central.peer(address), central.connect_default(), 517)
                Err(e)   => println(e.message())
                Ok(link) => println("max write ${phone.max_write(link, true)}")
```

The app lists the permission it needs, and the build refuses one that does not:

```toml
[mobile]
permissions = ["bluetooth"]
```

## What it does for a program

- **Operations queue.**  The platform runs one GATT operation at a time and drops a second one issued early.  Each link queues, so a program may issue a read,
  a write and a subscription back to back; each completes as one event, in order.  An operation the platform never answers is reported as failed when the next one
  is issued, 20 seconds later.
- **The cache of services.**  The platform answers a discovery from what it learned of a device earlier, which a firmware update makes stale; the first
  discovery of a link throws the cache away through the framework's `refresh`.
- **A link that ends.**  A failed connection and a lost link are one state change to the platform; here the first is `Connected` with a status that is not 0, the second
  is `Disconnected`, and the platform's client is freed in both.  `connect` returns at once with the link; a program that wants a time uses `session.connect_wait`.
- **The longest write** is the MTU less 3 and at most 512, the size of an attribute.  `request_mtu(517)` asks, `MtuChanged` answers.
- **Bonding is the system's.**  `Security.Encrypted` and `Security.Authenticated` ask for a bond once the link is up; the system shows its dialog and `BondChanged` reports.
- **The adapter**: `adapter()` says on, off, permission missing or no adapter, and `AdapterChanged` follows the switch.

## The extern surface

The functions the package binds are the runtime's own, `novo_android_ble_*` in `bin/runtime/mobile/novo_rt_android_ble.c`, declared in `ble_android.nv` with `@ffi`
and the effect `[ffi]`; they exist in an Android app only, so the package is layer `app`.  Everything above them is plain novo-lang: `decode.nv` turns a raw event
into a `CentralEvent`, keeps the peer and the MTU of each link and reads the table of services, and is tested on the host.  A scan returns what a platform reports
only while the screen is on, or for what a filter names: with the screen off the platform scans at a low duty and for filters only.

## Install and test

```
novo pkg add ble-central-android-nv
novo test tests/decode_tests.nv
bash tests/coverage.sh                  # the lines of src/ the suite reaches
bash tests/mobile/test_android_ble.sh   # in the repository: on a phone, against a peripheral it finds
```

## Modules

| Module | Contents |
| --- | --- |
| `ble_android` | `central()`, the `AndroidCentral` that implements `GattCentral`, its `start`, `stats`, `request_phy`, `read_phy`, `link_info`, `last_event_us` and `now_us`, and the `@ffi` declarations of the runtime |
| `decode` | `Raw`, `event_of`, `services_of`, the table of links, the errors of the runtime's calls, and the arguments made from the trait's values |
