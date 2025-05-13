## 🧼 Full Wazuh Cleanup & Fresh Install

Use this command to fully uninstall and reinstall Wazuh, including the manager, indexer, dashboard, and related components. Useful during lab resets or corrupted installs.

```bash
sudo bash wazuh-install.sh -a -o
````

> ⚠️ This will **wipe all data and configs** — use only in controlled environments (e.g., test labs).

---

## ⚙️ Enable Vulnerability Detector & Other Features

Edit the Wazuh configuration file to manually enable features like the **vulnerability detector**, **syscollector**, **file integrity monitoring**, etc.

```bash
sudo nano /var/ossec/etc/ossec.conf
```

> 📝 Tip: Look for `<vulnerability-detector>`, `<syscollector>`, and other modules in the XML structure.

---

## 🔁 Restart the Wazuh Manager Service

Apply changes made to configuration files by restarting the main Wazuh manager daemon.

```bash
sudo systemctl restart wazuh-manager
```

> 🔄 Needed after config edits or upgrades.

---

## 🔐 Reset Wazuh Dashboard User Passwords

Reset all Wazuh dashboard user passwords — including `admin`. The script is interactive and will prompt for new passwords.

```bash
sudo bash /usr/share/wazuh-indexer/plugins/opensearch-security/tools/wazuh-passwords-tool.sh -a
```

> 🛑 This resets **all dashboard users**. For **just `admin`**, use:

```bash
sudo bash /usr/share/wazuh-indexer/plugins/opensearch-security/tools/wazuh-passwords-tool.sh -u admin
```

---

## 📌 Notes

| Command / File                    | Purpose                                      |
| --------------------------------- | -------------------------------------------- |
| `wazuh-install.sh -a -o`          | Full uninstall & reinstall                   |
| `ossec.conf`                      | Core config for Wazuh agent/manager features |
| `systemctl restart wazuh-manager` | Reload config & restart services             |
| `wazuh-passwords-tool.sh`         | Resets OpenSearch / Dashboard user passwords |



---