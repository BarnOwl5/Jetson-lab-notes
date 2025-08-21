## Jetson Firefox Setup


### 1. Remove Snap-Based Firefox
Remove the pre-installed Snap version of Firefox completely, including its dependencies.
```bash
sudo snap remove firefox
sudo apt purge firefox
sudo apt autoremove
```

### 2. Add the Mozillateam PPA
Enable the official Mozillateam PPA, which provides regularly updated `.deb` packages of Firefox.
```bash
sudo add-apt-repository ppa:mozillateam/ppa
sudo apt update
```

### 3. Replace snap with PPA Version
Ensure that apt installs Firefox from the Mozillateam PPA instead of falling back to Snap.
```bash
sudo tee /etc/apt/preferences.d/mozilla-firefox > /dev/null <<EOF
Package: firefox*
Pin: release o=LP-PPA-mozillateam
Pin-Priority: 1001
EOF
```

### 4. Install Firefox (Deb Version)
Install Firefox from the PPA. This Version will receive updates via apt upgrade.
```bash
sudo apt update
sudo apt install firefox -y
```

### 5. Verify Binary Path
Confirm that the binary points to `/usr/bin/firefox` (PPA Version), not `/snap/bin/firefox`.
```bash
which firefox
>> /usr/bin/firefox

hash -r
firefox --version
>> Mozilla Firefox 142.0
```

### 6. (Optional) Add Alias for Convenience
Add an alias so that typing Firefox always executes the PPA Version.
```bash
echo "alias firefox='usr/bin/firefox'" >> ~/.bashrc
source ~/.bashrc
```

## 📌Note
💡 This setup ensures that Firefox is installed and updated through the Mozillateam PPA(apt upgrade).

💡 For unattended security updates, you can enable unattended-upgrades during setup.

💡 If Snap is reinstalled, confirm your apt preferences file to keep Firefox pinned to the PPA.

## License
This work is licensed under [CC BY-NC-ND 4.0](./LICENSE) 

- Non-commercial use only
- Attribution required
- No modifications
