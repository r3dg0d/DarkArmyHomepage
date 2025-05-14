# DarkArmyHomepage

The official homepage for The Dark Army.

## Creating a Tor Hidden Service with a Custom v3 Onion Address

This guide explains how to set up a Tor hidden service with a custom v3 `.onion` address using `mkp224o` on Arch Linux. A hidden service allows you to host a website accessible only through the Tor network, and `mkp224o` generates a vanity address (e.g., starting with `darkarmy`).

### Prerequisites
- Arch Linux system with Tor installed (`sudo pacman -S tor`).
- A web server like `nginx` (`sudo pacman -S nginx`).
- Basic command-line knowledge.

### Step 1: Set Up the Hidden Service
1. **Configure Tor**:
   Edit `/etc/tor/torrc`:
   ```bash
   sudo nano /etc/tor/torrc
   ```
   Add:
   ```text
   HiddenServiceDir /var/lib/tor/hidden_service/
   HiddenServicePort 80 127.0.0.1:80
   ```
   Save and exit.

2. **Set Permissions**:
   Create and secure the hidden service directory:
   ```bash
   sudo mkdir -p /var/lib/tor/hidden_service
   sudo chown -R tor:tor /var/lib/tor/hidden_service
   sudo chmod 700 /var/lib/tor/hidden_service
   ```

3. **Restart Tor**:
   Apply changes:
   ```bash
   sudo systemctl restart tor
   ```

4. **Check Default Address**:
   View the generated `.onion` address:
   ```bash
   sudo cat /var/lib/tor/hidden_service/hostname
   ```
   This will be replaced with a custom address in the next steps.

### Step 2: Generate a Custom Onion Address with `mkp224o`
`mkp224o` creates a vanity v3 `.onion` address by brute-forcing key pairs.

1. **Install `mkp224o`**:
   Use an AUR helper like `yay`:
   ```bash
   sudo pacman -S yay
   yay -S mkp224o
   ```
   Alternatively, build from source:
   ```bash
   git clone https://github.com/cathugger/mkp224o.git
   cd mkp224o
   ./autogen.sh
   ./configure
   make
   sudo make install
   ```

2. **Generate Vanity Address**:
   Run `mkp224o` to create an address starting with your desired prefix (e.g., `darkarmy`):
   ```bash
   mkp224o darkarmy
   ```
   - This creates a directory (e.g., `darkarmy...`) with `hostname`, `hs_ed25519_public_key`, and `hs_ed25519_secret_key`.
   - For faster results, use multiple threads (e.g., 4):
     ```bash
     mkp224o darkarmy -t 4
     ```
   - Note: Longer prefixes take more time. A 4–8 character prefix is practical on a standard CPU.

3. **Install the Custom Keys**:
   Copy the generated keys to the hidden service directory:
   ```bash
   sudo cp -r darkarmy*/* /var/lib/tor/hidden_service/
   sudo chown -R tor:tor /var/lib/tor/hidden_service/
   sudo chmod 700 /var/lib/tor/hidden_service/
   ```

4. **Restart Tor**:
   Reload Tor to use the new address:
   ```bash
   sudo systemctl restart tor
   ```

5. **Verify the Custom Address**:
   Check the new `.onion` address:
   ```bash
   sudo cat /var/lib/tor/hidden_service/hostname
   ```
   It should start with `darkarmy` (e.g., `darkarmy123...xyz.onion`).

### Step 3: Test the Hidden Service
1. **Set Up a Test Page**:
   Create a simple webpage for `nginx`:
   ```bash
   echo "<h1>Dark Army Onion Site</h1>" | sudo tee /usr/share/nginx/html/index.html
   ```

2. **Access the Site**:
   Use Tor Browser to visit your custom `.onion` address. If it loads, your hidden service is live.

### Security Tips
- **Backup Keys**: Save the contents of `/var/lib/tor/hidden_service/` securely. Losing `hs_ed25519_secret_key` means losing the address.
- **Firewall**: Restrict web server access to localhost:
  ```bash
  sudo ufw allow from 127.0.0.1 to any port 80
  ```
- **Anonymity**: Avoid linking your identity to the service. Use Tor for all operations.
- **Optimize `mkp224o`**: For faster address generation, use a shorter prefix or a more powerful machine.

### Troubleshooting
- **Site not loading**: Check Tor logs (`sudo journalctl -u tor`) and web server status (`systemctl status nginx`).
- **Slow `mkp224o`**: Use more threads (`-t`) or a shorter prefix. For specific prefixes, consider cloud resources.
- **Permission issues**: Ensure the Tor user owns `/var/lib/tor/hidden_service/` with `700` permissions.

Your Dark Army homepage is now live on the Tor network with a custom `.onion` address! For advanced setups, refer to the [Tor Project documentation](https://www.torproject.org/docs/tor-onion-service.html.en) or `mkp224o`’s [GitHub page](https://github.com/cathugger/mkp224o).

*保持隐秘！(Stay hidden!)*
