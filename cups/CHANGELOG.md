# Changelog

## 1.9-beta9

- Neutralise the CUPS frame-buster in the ingress proxy. Beyond the framing
  headers, every CUPS page hides itself with `html{display:none;}` and runs
  `top.location = self.location` when it detects a frame, which navigated the
  whole browser window out of the ingress iframe instead of rendering in it.

## 1.9-beta8

- Fix the blank ingress panel. The CUPS web interface now runs behind a small
  nginx reverse proxy on the ingress port. CUPS sends `X-Frame-Options: DENY`
  and `Content-Security-Policy: frame-ancestors 'none'`, and ingress renders
  add-on UIs in an iframe, so the browser refused to display anything. CUPS has
  no setting for those headers, so the proxy strips them. The proxy also
  rewrites the absolute links CUPS emits (`/cups.css`, `/admin`, ...) onto the
  ingress path prefix, which would otherwise 404.
- The panel is served on port 8099, restricted to `172.30.0.0/16` so only
  Supervisor-managed networks reach it. Ingress arrives from the Supervisor's
  own network, not the hassio bridge, so a narrower rule rejects it. Port 631
  is unchanged for direct access on the LAN.

## 1.9-beta6

- Revert `publish-addresses=no` from 1.9-beta5. Supervisor's `hassio_multicast`
  turned out to be `mdns-repeater`, a packet repeater rather than a responder,
  so there was never a competing publisher to defer to. Suppressing our address
  records only made the printer's SRV target depend on some other process
  publishing `homeassistant.local`. The "another mDNS stack" warning comes from
  the repeater holding port 5353 and cannot be avoided while it runs.
- Drop the `dbus: true` key from the add-on config. Supervisor's option is
  `host_dbus`; `dbus` was never recognised and had no effect. The add-on runs
  its own D-Bus for Avahi, and mapping the host bus would collide with it.

## 1.9-beta5

- Stop advertising SSH and SFTP over mDNS. The avahi package ships service
  files for them, so under `host_network` this print server was publishing
  `_ssh._tcp` and `_sftp-ssh._tcp` records for the host.
- Wait for the D-Bus socket before starting Avahi. s6 starts the dependency but
  does not wait for readiness, so on a cold start avahi-daemon found no bus,
  exited 255, and only came up on the supervisor's restart.
- Set `publish-addresses=no`. Supervisor's `hassio_multicast` plugin runs its
  own Avahi on this host and already publishes the host's address records;
  republishing them is what triggered avahi's "Detected another mDNS stack"
  warning. Only the printer service is advertised now, and the SRV target still
  resolves through the Supervisor's responder.

## 1.9-beta4

- Fix AirPrint failing on iOS when adding the printer. Two independent causes:
  - CUPS had no `ssl` directory, so it could not create its self-signed
    certificate ("Unable to create server credentials") and reset every TLS
    handshake -- while Avahi still advertised `_ipps._tcp` with `TLS=1.2`.
    Apple clients prefer the encrypted record and failed at that point.
  - The seeded ACL allowed a hardcoded IPv4 subnet and, for IPv6, only
    loopback. Apple clients resolve `<host>.local` to IPv6 first and were
    refused with 403 exactly when adding the printer. Now `Allow @LOCAL`.
- Repair the legacy hardcoded ACL on existing installs. Seeding only ran when
  cupsd.conf was absent, so no update could ever correct it.
- Remove cups-browsed. It rediscovered this add-on's own advertised queue and
  rebuilt it in a loop, republishing mDNS records each cycle. This is also the
  cause of the 1.9-beta3 known issue where a client reported "printing failed"
  after a successful print and the job was re-queued.
- Bind Avahi to the detected LAN interface. Under `host_network` it otherwise
  also published HA's docker bridge and veth addresses, which clients cannot
  reach.
- Actually apply `admin_username` / `admin_password`. They were declared in the
  schema but never read, so no CUPS admin account existed.
- Add the missing `build.yaml`. Current Supervisor releases no longer supply an
  implicit `BUILD_FROM`, so the build failed with "base name ($BUILD_FROM)
  should not be blank".
- Add `edge/main` to the apk repositories. `hplip` is only in `edge/community`
  and now requires `python3~3.14`, which Alpine 3.23 does not carry, so the
  build failed with "python3-3.12.14-r0: breaks: hplip-3.26.4-r0[python3~3.14]".

## 1.9-beta3

- Start cups-browsed so not only discovery but actual printing works
- Known issues: In one test setup this caused the client to receive a "printing failed" error after the document was successfully printed. The job needs to be canceled on the client or will be re-queued. 

