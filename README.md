![GitHub Repo Size](https://img.shields.io/github/repo-size/caostheory/Unbound) ![License](https://img.shields.io/github/license/caostheory/Unbound) ![Last Commit](https://img.shields.io/github/last-commit/caostheory/Unbound)

---

## Installation

1. **Download the Unbound configuration file**  
   ```bash
   doas fetch -o /usr/local/etc/unbound/unbound.conf \
   https://raw.githubusercontent.com/caostheory/Unbound/refs/heads/main/unbound.conf
   ```

2. **Generate the root key for DNSSEC validation**  
   ```bash
   doas unbound-anchor -a /usr/local/etc/unbound/root.key
   ```

3. **Create the auth-zone directory and fetch `root.zone`**  
   ```bash
   # Create auth-zone directory
   doas mkdir -p /usr/local/etc/unbound/zones/

   # Download root.zone
   doas fetch -o /usr/local/etc/unbound/zones/root.zone https://www.internic.net/domain/root.zone
   ```

4. **Create a directory for logs**  
   ```bash
   doas mkdir -p /usr/local/etc/unbound/logs/
   ```

5. **Generate server and control keys**  
   ```bash
   # Create a folder for keys
   doas mkdir -p /usr/local/etc/unbound/keys/

   # Install keys in the right directory
   doas unbound-control-setup -d /usr/local/etc/unbound/keys/
   ```

6. **Set correct permissions**  
   ```bash
   doas chown -R unbound:unbound /usr/local/etc/unbound/
   doas chmod 600 /usr/local/etc/unbound/keys/*.key
   ```

---

## Testing Your Setup

- **Basic DNS resolution**  
  ```bash
  dig @127.0.0.1 -p 5335 www.internic.net
  ```

- **Verify DNSSEC validation**  
  ```bash
  dig @127.0.0.1 -p 5335 +dnssec www.internic.net
  ```

  **How to interpret results:**
  - If DNSSEC is supported, the output includes **RRSIG** records.
  - If validation is functioning properly, you’ll see the **`ad`** flag in the response header.
    ```bash
    ;; flags: qr rd ra ad;
    ```
    The presence of `ad` confirms that the response was authenticated via DNSSEC.

- **Check Unbound status and statistics**  
  ```bash
  doas unbound-control status
  doas unbound-control stats_noreset
  ```

---

## Integration

- **AdGuard Home**  
  Use `127.0.0.1:5335` and `[::1]:5335` as upstream DNS servers.

- **Pi-hole**  
  Set custom DNS to `127.0.0.1#5335` and `::1#5335`.

---

## Tips & Maintenance

- **You can schedule a monthly update using `cron`**:

  ```bash
  doas crontab -e

  # Monthly update of root.key
  @monthly /usr/local/sbin/unbound-anchor -a /usr/local/etc/unbound/root.key > /dev/null 2>&1
  ```

---

*Made with ❤️ by [caostheory](https://github.com/caostheory)*