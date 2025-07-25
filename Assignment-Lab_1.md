Assignment/Lab 1 [SONiC Fundamentals] – 21/7/2025
🧪 Build Environment Setup & SONiC Virtual Switch Installation

Objective
Set up a SONiC build/runtime environment using a prebuilt sonic-vs.img, and install/run it on a virtual platform using QEMU/KVM.

Prerequisites
A Debian-based Linux VM (e.g., Ubuntu 20.04+)

Internet access

sudo privileges

At least 8 GB RAM, 2 CPUs, and 10 GB+ disk space

✅ Step 1: Install QEMU, KVM, and Networking Tools

sudo apt update
sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils

📝 This installs the virtualization tools needed to run QEMU VMs with networking support.

✅ Step 2: Add Current User to Required Groups

sudo adduser $USER libvirt
sudo adduser $USER kvm

📌 After this, log out and log back in for group changes to take effect.

✅ Step 3: Download Prebuilt SONiC Virtual Switch Image

wget -O sonic-vs.img.gz 'https://artprodcus3.artifacts.visualstudio.com/.../sonic-vs.img.gz'

📝 This image is a compressed version of SONiC Virtual Switch (VS).

✅ Step 4: Extract the Image

gunzip sonic-vs.img.gz

📂 You will get a file named sonic-vs.img.

✅ Step 5: Create Startup Script

Create and edit a new file:

nano start-sonic.sh

Paste the following content:

#!/bin/bash

# VM Configuration
SONIC_IMAGE="sonic-vs.img"
MEMORY="4096" # in MB
CPUS="2"

# SSH port forwarding (Host Port -> Guest Port)
MGMT_PORT_FWD="tcp::2222-:22"

# Data plane ports (sockets for inter-VM connection)
DP_SOCKETS=(5000 5001 5002 5003)

# --- Build QEMU Command ---
QEMU_CMD="qemu-system-x86_64"
QEMU_CMD+=" -m ${MEMORY}"
QEMU_CMD+=" -smp ${CPUS}"
QEMU_CMD+=" -drive file=${SONIC_IMAGE},format=qcow2,index=0,media=disk"

# Management Interface (eth0)
QEMU_CMD+=" -device e1000,netdev=management"
QEMU_CMD+=" -netdev user,id=management,hostfwd=${MGMT_PORT_FWD}"

# Data Plane Interfaces (Ethernet0, Ethernet1, ...)
for i in "${!DP_SOCKETS[@]}"; do
    QEMU_CMD+=" -device e1000,netdev=dpu${i},mac=02:00:00:00:00:$(printf %02x ${i})"
    QEMU_CMD+=" -netdev socket,id=dpu${i},listen=:${DP_SOCKETS[$i]}"
done

# Console and Display
QEMU_CMD+=" -nographic"

# --- Run QEMU ---
echo "Starting SONiC VM... To exit, press Ctrl+A then X."
eval ${QEMU_CMD}

Then make it executable:

chmod +x start-sonic.sh

✅ Step 6: Start the SONiC Virtual Machine

./start-sonic.sh

⏳ It will boot into a SONiC console inside the terminal.

✅ Step 7: Access SONiC CLI via SSH (Optional)

If you'd prefer to access via SSH from your host:

ssh -p 2222 admin@localhost

Default username: admin

Default password: YourPaSsWoRd

🔍 Verification

show version
show platform summary
docker ps

📝 Summary & Learnings

Learned how to install and configure QEMU/KVM.
Successfully launched SONiC Virtual Switch in a VM.
Validated system functionality using CLI commands.

