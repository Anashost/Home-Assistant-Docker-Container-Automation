<!-- anashost_support_badges_start -->
[![Revolut.Me][revolut_me_shield]][revolut_me]
[![PayPal.Me][paypal_me_shield]][paypal_me]
[![ko_fi][ko_fi_shield]][ko_fi_me]
[![buymecoffee][buy_me_coffee_shield]][buy_me_coffee_me]
[![patreon][patreon_shield]][patreon_me]
<!-- anashost_support_badges_end -->
<!-- 
```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
-->

# Control and Automate Docker Containers using Home Assistant

With this method, you can seamlessly interact with Docker containers running on a different machine (e.g., a virtual machine or physical server) using Home Assistant. Whether you need to start, stop, restart, or kill containers, this setup provides a simple way to control your containers using Home Assistant’s powerful automation system.

- start container
- stop container
- kill container
- restart container

---

### **Step 1: Generate SSH Keys in Home Assistant**
1. **Install the SSH Add-on**:
   - In Home Assistant, go to **Settings → Add-ons → Add-on Store**.
   - Search for **Terminal & SSH**, install it, and start it. Enable `Start on boot` and `Watchdog`.

2. **Generate SSH Keys**:
   - Open the **Terminal & SSH** add-on terminal.
   - Run these commands:
     ```bash
     mkdir -p /config/.ssh
     ssh-keygen -t ed25519 -f /config/.ssh/id_rsa -N ""
     ```
     - This creates a key pair **without a passphrase** (to avoid prompts).  
     - **Important**: Use `/config/.ssh`, **not** `/data/.ssh`. This ensures the keys persist after updates.

---

### **Step 2: Copy the Public Key to Your VM**
1. **Find Your Public Key**:
   - In the SSH terminal, run:
     ```bash
     cat /config/.ssh/id_rsa.pub
     ```
   - Copy the output (it starts with `ssh-ed25519 ...`).

2. **Add the Key to the VM (the VM where docker is installed)**:
   - SSH into your VM (from your local PC or HA terminal):
   - Note: You can use [PuTTY](https://www.putty.org/) to ssh to Home Assistant and the VM where Docker is installed.

     ```bash
     ssh user@vm-ip
     ```
   - On the VM, run:
     ```bash
     mkdir -p ~/.ssh
     echo "PASTE_YOUR_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys
     chmod 700 ~/.ssh
     chmod 600 ~/.ssh/authorized_keys
     ```

---

### **Step 3: Fix SSH Permissions**
#### **On Home Assistant**:
- Ensure the private key has strict permissions:
  ```bash
  chmod 600 /config/.ssh/id_rsa
  ```

#### **On the VM**:
- Edit the SSH config file (`/etc/ssh/sshd_config`):
  ```bash
  sudo nano /etc/ssh/sshd_config
  ```
- Add/update these lines:
  ```ini
  PermitRootLogin no          # Disable root login (Optional)
  PasswordAuthentication no   # Disable password login
  PubkeyAuthentication yes    # Enable SSH keys
  ```
- Restart SSH:
  ```bash
  sudo systemctl restart sshd
  ```

---

### **Step 4: Test SSH Connection**
- From Home Assistant’s SSH terminal, test:
  ```bash
  ssh -i /config/.ssh/id_rsa user@vm-ip
  ```
  - Replace `user` and `vm-ip` with your VM’s username and IP.
  - If it works, you’ll log in without a password. Type `exit` to quit.

---

### **Step 5: Fix "Host Key Verification Failed"**
This error occurs because Home Assistant doesn’t "trust" the VM’s SSH fingerprint. To fix it:

1. **Manually Accept the Fingerprint**:
   - Run this command **once** in the HA SSH terminal:
     ```bash
     ssh -o StrictHostKeyChecking=no -i /config/.ssh/id_rsa user@vm-ip "exit"
     ```
   - This adds the VM’s fingerprint to `/root/.ssh/known_hosts` in HA.

---

### **Step 6: Update Home Assistant Configuration**
1. **Add `shell_command`**:
   - In `configuration.yaml`, add:
     ```yaml
     shell_command:
       docker_start_container_name: 'ssh -o StrictHostKeyChecking=no -i /config/.ssh/id_rsa user@vm-ip "docker start container_name"'
       docker_stop_container_name: 'ssh -o StrictHostKeyChecking=no -i /config/.ssh/id_rsa user@vm-ip "docker stop container_name"'
       docker_kill_container_name: 'ssh -o StrictHostKeyChecking=no -i /config/.ssh/id_rsa user@vm-ip "docker kill container_name"'
       docker_restart_container_name: 'ssh -o StrictHostKeyChecking=no -i /config/.ssh/id_rsa user@vm-ip "docker restart container_name"'
     
     ```
   - Replace `user`, `vm-ip`, and `container_name`.
   - This command disables host key checks and uses the correct key path. Once confirmed working, you COULD remove `-o StrictHostKeyChecking=no`.

3. **Restart Home Assistant**:
   - Go to **Settings → System → Restart**.

---

### **Step 7: Create Automations**
1. **Create a Test Automation**:
   - Go to **Settings → Automations → Create Automation**.
   - Use a simple trigger (e.g., time), and set the action to `Call Service` → `shell_command.docker_start_container_name`.
   -  Replace `container_name`.
   - Test it to see if the container starts.

---
## Enjoy..

---

[paypal_me_shield]: https://img.shields.io/badge/PayPal-00457C?style=for-the-badge&logo=paypal&logoColor=white

[paypal_me]: https://paypal.me/anasboxsupport

[revolut_me_shield]:
https://img.shields.io/badge/revolut-FFFFFF?style=for-the-badge&logo=revolut&logoColor=black

[revolut_me]: https://revolut.me/anas4e

[ko_fi_shield]: https://img.shields.io/badge/Ko--fi-F16061?style=for-the-badge&logo=ko-fi&logoColor=white

[ko_fi_me]: https://ko-fi.com/anasbox

[buy_me_coffee_shield]: 
https://img.shields.io/badge/Buy%20Me%20Coffee-ffdd00?style=for-the-badge&logo=buy-me-a-coffee&logoColor=black

[buy_me_coffee_me]: https://www.buymeacoffee.com/anasbox

[patreon_shield]: 
https://img.shields.io/badge/patreon-404040?style=for-the-badge&logo=patreon&logoColor=white

[patreon_me]:  https://patreon.com/AnasBox
