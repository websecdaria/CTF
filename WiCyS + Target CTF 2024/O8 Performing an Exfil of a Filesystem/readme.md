### O8: Performing an Exfil of a Filesystem

You have a solid foothold in The Lucky Lion's environment - now it's time to start poking around. Looking through more of the [emails from the account you compromised](https://target-httpd.chals.io/webmail/webmail-inbox.html), you see something interesting: a backup of a server was recently uploaded to a secure fileshare. You wager there could be some valuable information to sell if you can get those files, but the backup is password protected.

To get you started, we've provided you with the host & port where we've noticed a password vault service running. You can connect to it with Netcat (`nc`) as shown at the bottom.

**Note:** the hints worth 50 are about the ZIP!

**Objectives**

- Find a way to extract passwords from the vault so you can download the ZIP.
- Once you obtain the ZIP, find a way to break the encryption scheme and find the flag file within.

**Tools Required**

- Shell environment with `nc` (netcat) installed.

**Additional Resources**

- [All about ZIPs](https://en.wikipedia.org/wiki/ZIP_(file_format))
- [ValuVault documentation](https://target-httpd.chals.io/valuvault.html)

```bash
nc 0.cloud.chals.io 18529
```
---

**Approach:**

Connect to the Valuvault service via Netcat:

```bash
nc 0.cloud.chals.io 18529
```

**Valuvault Insights:**

Valuvault provides several commands: `LIST`, `GET`, `MOTD`, and `QUIT`. By experimenting with `MOTD`, which echoes back entered input, I identified that it can be exploited using template variables. Using:

```bash
MOTD {now.__init__.__globals__}
```

I accessed sensitive information, including the master password. This allowed me to list user accounts and retrieve passwords, including the `backup_admin` password needed to download the backup ZIP file.

**Retrieving the Backup:**

With the `backup_admin` password in hand, I downloaded the backup file from the provided URL. The ZIP file contained two password-protected files: `slots.txt` and `flag.txt`.

**Decrypting ZIP Files:**

Standard tools like John the Ripper, Fcrackzip, and Hashcat were ineffective due to the encryption method. The solution was `bkcrack`, which performs a plaintext attack against ZIP files. 

I used the following steps:

1. Retrieve `slots.txt` using the `slots_admin` password obtained earlier.
2. Use `bkcrack` with `backup.zip`, `slots.txt`, and a plaintext version of `slots.txt` (`slotsdeciphered.txt`) to find decryption keys.

```bash
bkcrack -C backup.zip -c slots.txt -p slotsdeciphered.txt
```

The command returned three keys: `32b5ff40`, `f29e8dfd`, and `f1b4a5da`. These keys can be used to decrypt `flag.txt`:

```bash
bkcrack -C backup.zip -c flag.txt -k 32b5ff40 f29e8dfd f1b4a5da -d decrypted.txt
```

**Flag:**

```bash
flag{xamine_your_zip_pretty_darn_quick}
```

