# MeshConnect

`meshconnect.exe` joins one Windows PC to your NordVPN Meshnet for remote access only.
It adds one network adapter with a Meshnet IP and routes **only** `100.64.0.0/10` through
it — no default route, no DNS change. Your internet and LAN traffic are untouched.

Built on NordVPN's open-source libtelio/tcli. x64, static CRT (no VC++ redistributable
needed), WireGuardNT driver embedded. SHA-256 of this build:
`4bd3f4aa42c58b9df954105824a7f32fde5dbcea92e58bce3af860cc5a203412`.

## Install (on the target PC)

1. Copy `meshconnect.exe` to the PC and run it as administrator (it prompts for elevation).
   SmartScreen may say "Windows protected your PC" (the exe is unsigned) → More info → Run anyway.
2. It offers to turn on Remote Desktop and to disable sleep on AC, then asks for a
   **NordVPN access token**: my.nordaccount.com → NordVPN → Manual setup → Access token →
   Generate new token. **Pick "Doesn't expire".** Input is hidden.
3. It registers the PC on your Meshnet, installs a boot-start service, and prints the PC's
   Meshnet IP. Connect to that IP with Remote Desktop from your Mac/iPhone/Pi.

Keep your current way into the PC until RDP over Meshnet works.

## Commands (elevated)

- `meshconnect.exe` — install, or show status / offer uninstall if already installed
- `meshconnect.exe status`
- `meshconnect.exe set-token` — replace an expired/revoked token (no reinstall)
- `meshconnect.exe uninstall` — removes everything and takes the PC off your Meshnet

Non-interactive: `meshconnect.exe install --token-file C:\path\token.txt --yes`
(the file is deleted after reading), or set `NORD_TOKEN` in the environment.

## What it does / does not do

- Remote Desktop is admitted **only** from the Meshnet IPs of devices you've allowed
  (NordVPN "Allow incoming connections"), bound to this PC's Meshnet IP, TCP 3389 only.
- No internet or LAN traffic is routed through NordVPN. Verify with `route print -4`:
  `0.0.0.0` stays on your normal gateway; the only new line is `100.64.0.0 255.192.0.0`.
- The token is stored under `C:\Program Files\MeshConnect\Data` (admin-only), never on the
  command line or in logs.

## Verified

End-to-end on a real Windows Server 2022 runner (55/55 checks), including: install, Meshnet
join, routes/DNS/egress unchanged, per-peer RDP firewall, inbound RDP from the author's Mac
over Meshnet, service restart + crash recovery, token replacement, and clean uninstall
leaving the machine byte-identical to baseline.

Source: `../libtelio/clis/meshconnect/`. Rebuild: see `../FIXPLAN.md` and
`~/.claude/.../memory/remote-windows-pc-mesh-rdp-design.md`.
