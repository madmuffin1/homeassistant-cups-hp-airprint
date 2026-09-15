# Changelog

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

