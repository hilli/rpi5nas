# NAS software

I'm using the 'NAS' term lightly here... It's one drive, but it does publish a disk drive to the local network.

Evaluated:

- [CasaOS 0.4.6](https://casaos.io)
- [Open Media Vault](https://www.openmediavault.org)

CasaOS, while looking the sleekest, failed pretty badly on not having user access control on the fileshares. Otherwise, being made with Go, it looked pretty apealing.

OpenMediaVault, on the other hand, does not accept sharing disk space from the system drive. Unneccecary hold back, if you ask me, especially in this situation. Could probably run with it fine off an SD card for, well, however an SD card holds which is not for long with continued writes to the same sectors as a regular linux install would do.

So we will go with CasaOS and then configure Samba shares manually.

## CasaOS

Simply installed with

```
curl -fsSL https://get.casaos.io | sudo bash
```

When done it will show version and where to create a login:

```
 CasaOS v0.4.6 is running at:
 ─────────────────────────────────────────────────────
 - http://10.0.0.162 (eth0)
 - http://10.0.0.163 (wlan0)
 Open your browser and visit the above address.
 ─────────────────────────────────────────────────────
 ```

## Tailscale

Super nice to have to get access to the NAS from any of your other devices in the tailnet.

Install with the app installer, using a key from your tailscale account.
