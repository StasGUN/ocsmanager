# 🚀 OCS Manager: beginner's guide

**English** · [Русский](QUICKSTART.ru.md) · [← back to README](../README.md)

This guide takes you from renting a server to a working VPN on your phone and
computer. No Linux experience is needed: if you can copy and paste commands,
you can do this. It takes about 20–30 minutes.

---

## Step 1. Rent a VPS

Almost any inexpensive VPS will do. When ordering, choose:

- **OS:** Ubuntu 24.04 (or Ubuntu 22.04, Debian 12);
- **resources:** 1 CPU, 1 GB RAM or more, 10 GB disk or more, which is plenty;
- **virtualization:** **KVM** is best. On an OpenVZ/LXC plan, make sure
  **TUN/TAP** is enabled in the hosting control panel.

After payment the host sends you the server's **IP address** and **root password**
(or asks you to add an SSH key).

## Step 2. Set up a domain name (recommended)

A domain lets you get a real Let's Encrypt certificate, so clients won't warn
about an "untrusted certificate".

1. Buy a domain from any registrar, or use a subdomain of one you own
   (e.g. `vpn.example.com`).
2. In the DNS settings create an **`A`** record: name `vpn`, value: your server's IP.
3. Wait 5–30 minutes for the record to propagate.

> No domain? You can skip it: the script will create a self-signed certificate.
> On the first connection the client asks whether to trust the server; answer "yes".

## Step 3. Connect to the server over SSH

- **Windows 10/11:** open "Terminal" or PowerShell.
- **macOS / Linux:** open "Terminal".

Type (use your server's IP):

```bash
ssh root@203.0.113.10
```

Answer `yes` to `Are you sure you want to continue connecting?`, then enter the
root password. The password isn't shown as you type; that's normal.

## Step 4. Run the installer

Copy and run these two commands:

```bash
wget https://github.com/StasGUN/ocsmanager/releases/latest/download/install-ocserv.sh
sudo bash install-ocserv.sh
```

The script asks a few questions. If unsure, press **Enter** to take the default.

| Prompt | What it means | What to answer |
|---|---|---|
| `Server domain name` | Domain name | `vpn.example.com` from step 2 |
| `VPN address pool` | Internal VPN network | Just **Enter** |
| `VPN user name` | User name | E.g. your first name: `ivan` |
| `Password for ...` | Password | Make one up, or press Enter to get a strong random one |
| `Enable camouflage?` | Hide the server | **Enter** (yes, recommended) |
| `Camouflage secret` | Secret for the address | **Enter**: generated automatically |
| `TLS certificate` | Certificate type | `1` (Let's Encrypt) if you set up a domain, otherwise `2` |
| `E-mail for Let's Encrypt` | Expiry notices | Your e-mail, or Enter |
| `Start the installation?` | Go? | **Enter** |

The build takes 5–15 minutes. Keep the terminal window open.

## Step 5. Save the summary

At the end you'll see a green **INSTALLATION COMPLETED SUCCESSFULLY** and a summary.
**Copy it somewhere safe** (a password manager or notes). The key parts:

```
  User            : ivan
  Password        : Xy7...           <- save it, it is not stored anywhere in clear text
  Server address to enter in the client:
      https://vpn.example.com/?a1B2c3D4e5F6g7H8
```

> ⚠️ **The client address includes the secret after `/?`.** With camouflage on,
> plain `vpn.example.com` won't work: the server pretends to be an ordinary website.

## Step 6. Connect to the VPN

### 📱 Phone (iPhone / Android)

1. Install **Cisco Secure Client** from the App Store or Google Play.
2. Open it → **Connections** → **Add VPN Connection**.
3. **Description:** any name, e.g. `My VPN`.
4. **Server Address:** the address from step 5, *including the secret*.
5. Save, flip the switch on, and enter your user name and password.

### 💻 Computer (Windows / macOS)

1. Open `https://vpn.example.com:8443/` in a browser (no secret needed).
2. Sign in with your VPN user name and password and download the client for your OS.
   *It shows up there once the administrator has uploaded installers in the panel
   (see step 7). Otherwise install Cisco Secure Client or
   [OpenConnect GUI](https://gui.openconnect-vpn.net/) yourself.*
3. In the client, enter the address from step 5 and click **Connect**.

### 🐧 Linux

```bash
sudo apt install openconnect
sudo openconnect 'https://vpn.example.com/?a1B2c3D4e5F6g7H8'
```

**Check it:** open any "what is my IP" site. It should show your server's IP.

## Step 7. Open the admin panel

1. **Connect to the VPN** first, because the panel opens only through it.
2. Open `https://vpn.example.com:8443/portal` and sign in as the first VPN user
   (from step 5).

In the panel you can:

- 👥 add and remove users, change passwords;
- 📊 see traffic and connection history (click a user name);
- 🔌 see and drop active sessions;
- 📦 upload Cisco Secure Client installers for Windows and macOS;
- 🚫 manage fail2ban: bans, whitelist;
- 🔄 update the OS, the panel and ocserv, reboot the server.

## Step 8. Updates

Check the panel now and then: the **OCS Manager** card → **Update the panel**.
Updates are verified by a digital signature, and if something fails the panel
returns to the previous version by itself.

---

## ❓ Troubleshooting

<details>
<summary><b>The client can't connect / "server not found"</b></summary>

- Make sure the address includes the secret: `https://domain/?secret`.
- Check that your hosting provider has no firewall of its own blocking port
  **443** (TCP and UDP). If it does, open the port in the hosting panel.
</details>

<details>
<summary><b>The admin panel shows 404</b></summary>

That's by design: the panel opens only with the VPN connected. Connect and try
again. If the VPN itself doesn't work, use an SSH tunnel:

```bash
ssh -L 8443:localhost:8443 root@vpn.example.com
```

then open `https://localhost:8443/portal`.
</details>

<details>
<summary><b>The browser warns about the panel's certificate</b></summary>

If you open the panel by its internal VPN address (e.g.
`https://172.16.28.1:8443/portal`), the warning is expected: the certificate is
issued for the domain, not the IP. Continue anyway, or open the panel by domain.
</details>

<details>
<summary><b>The portal on port 8443 doesn't open from an office or guest Wi-Fi</b></summary>

Some networks block non-standard ports. The VPN itself (port 443) still works.
You can move the portal: `sudo OC_PORTAL_PORT=<port> bash install-ocserv.sh`.
</details>

<details>
<summary><b>Error "/dev/net/tun is missing"</b></summary>

TUN/TAP is disabled on the server. Enable it in the hosting panel (or ask
support) and run the script again.
</details>

<details>
<summary><b>Let's Encrypt didn't issue a certificate</b></summary>

Usually the domain's `A` record doesn't point to the server yet, or port 80 is
busy. The script then falls back to a self-signed certificate. Once DNS works,
just run the script again; re-running is safe.
</details>

<details>
<summary><b>I forgot the password</b></summary>

Change it in the panel or over SSH:

```bash
sudo ocpasswd -c /etc/ocserv/ocpasswd ivan
```
</details>

Still stuck? Look at the install log `/var/log/ocserv-install-<date>.log`
and [open an issue](https://github.com/StasGUN/ocsmanager/issues).
