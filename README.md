Here’s how to set up **Docker inside a Debian 12 LXC container** on Proxmox:

---

### **1. Prepare the Debian 12 LXC Container**

1. **Create the LXC Container**:
   - Go to the Proxmox Web UI and create an LXC container using the **Debian 12 template** you downloaded earlier.
   - Allocate enough resources for the container:
     - CPU: 2 cores.
     - Memory: 4 GB.
     - Disk: 8 GB.
   - Select a network bridge and configure an IP address.

2. **Enable Nesting for the Container**:
   Docker requires specific kernel features that may be restricted by default in LXC containers. Enable nesting to ensure these features work:
   - After creating the container, edit its configuration file:
     ```bash
     nano /etc/pve/lxc/<CTID>.conf
     ```
   - Add the following line:
     ```bash
     features: keyctl=1,nesting=1
     ```
   - Save and exit.

3. **Restart the Container**:
   ```bash
   pct restart <CTID>
   ```

---

### **2. Install Docker in the LXC Container**

1. **Access the LXC Container**:
   - Open the Proxmox Web UI, select your container, and click **Console**.
   - Alternatively, SSH into the container.

2. **Update the Package List**:
   ```bash
   apt update && apt upgrade -y
   ```

3. **Install Required Packages**:
   ```bash
   apt install -y ca-certificates curl gnupg lsb-release
   ```

4. **Add Docker’s GPG Key**:
   ```bash
   curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
   ```

5. **Set Up the Docker Repository**:
   ```bash
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```

6. **Install Docker**:
   ```bash
   apt update && apt install -y docker-ce docker-ce-cli containerd.io
   ```

7. **Verify Docker Installation**:
   ```bash
   docker --version
   ```
   Example output:
   ```
   Docker version 24.x.x, build xxxxxxx
   ```

---

### **3. SSH Connection to the LXC Container**
The easiest way to copy files is with an SSH connection:

#### **Steps**:
1. **Ensure SSH is Installed and Running in the LXC Container**:
   - SSH is often not installed by default in an LXC container.
   - Inside the container, install and enable SSH:
     ```bash
     apt update && apt install -y openssh-server
     systemctl enable ssh
     systemctl start ssh
     ```
   - Find the container's IP address using:
     ```bash
     ip addr
     ```

---

If you're having trouble authenticating as `root` for your LXC container, here are the most common causes and solutions:

---

### **4. Ensure Root SSH Access is Enabled**
By default, many Debian-based systems disable root SSH access for security reasons.

#### **Fix: Enable Root Login for SSH**
1. Log in to the container using the Proxmox console.
2. Edit the SSH configuration file inside the container:
   ```bash
   nano /etc/ssh/sshd_config
   ```
3. Look for the following line:
   ```bash
   PermitRootLogin prohibit-password
   ```
   Change it to:
   ```bash
   PermitRootLogin yes
   ```
4. Restart the SSH service:
   ```bash
   systemctl restart ssh
   ```

---
