# Setup Persistent SSH Reverse Tunnel
A reverse SSH tunnel allows a remote server to access services on a local machine, bypassing firewalls and NAT.

## Getting Started

This repository contains scripts for setting up a persistent SSH reverse tunnel using autossh. It is designed to maintain a stable connection between a host and a bastion (jump) server.

#### Illustration
| Host | -> (Reverse SSH Tunneling) -> | Bastion Server|
| Client | -> | Bastion Server | -> (Port Forwarding) -> | Host |


## Prerequisites

- sudo privileges on the bastion server.


## Installation

#### Host

###### ssh daemon
``` bash
sudo apt update
sudo apt install openssh-server
sudo service ssh start
sudo service ssh status
```

###### Secret key

```bash
SECRET_KEY_PATH=$HOME/.ssh/id_rsa
ssh-keygen -t rsa -b 4096 -f $SECRET_KEY_PATH -N ""
cat "${SECRET_KEY_PATH}.pub" # Copy the contents of the public key to your clipboard
```

#### Bastion Server

``` bash
echo <HOST'S_PUBLIC_KEY_CONTENTS> >> $HOME/.ssh/authorized_keys
```

#### Host
``` bash
# Installs autossh
sudo yum install -y autossh

# Sets up ssh reverse tunnel as a daemon service
cd /tmp
git clone https://github.com/ywatanabe1989/setup-persistent-ssh-reverse-tunnel.git
cd setup-persistent-ssh-reverse-tunnel

# Parameters
PORT=$(shuf -i 1024-65535 -n 1) # Generates a random port number for security
BASTION_SERVER=<USER-NAME>@<EXTERNAL IP ADDRESS> # Replace with your bastion server details
SECRET_KEY_PATH=$HOME/.ssh/id_rsa # Path to your private key

# Registers ```/etc/systemd/system/autossh-tunnel-"$PORT".service```
./setup-autossh-service.sh -p $PORT -b $BASTION_SERVER -s $SECRET_KEY_PATH
# See /etc/systemd/system/autossh-tunnel-<YOUR_PORT>.service
# Created symlink /etc/systemd/system/multi-user.target.wants/autossh-tunnel-<YOUR_PORT>.service → /etc/systemd/system/autossh-tunnel-<YOUR_PORT>.service.
# ● autossh-tunnel-<YOUR_PORT>.service - AutoSSH tunnel service
#      Loaded: loaded (]8;;file://HOST/etc/systemd/system/autossh-tunnel-<YOUR_PORT>.service/etc/systemd/system/autossh-tunnel-<YOUR_PORT>.service]8;;; enabled; preset: disabled)                                
#      Active: active (running) since Sat 2024-08-31 22:14:16 JST; 12ms ago
#    Main PID: 63681 (autossh)
#       Tasks: 2 (limit: 403847)
#      Memory: 1.6M
#         CPU: 4ms
#      CGroup: /system.slice/autossh-tunnel-<YOUR_PORT>.service
#              ├─63681 /usr/bin/autossh -M 0 -N -o PubkeyAuthentication=yes -o PasswordAuthentication=no -i /home/>
#              └─63682 /usr/bin/ssh -N -o PubkeyAuthentication=yes -o PasswordAuthentication=no -i /home/ywatanabe>
#  
# Aug 31 22:14:16 HOST systemd[1]: Started AutoSSH tunnel service.
# Aug 31 22:14:16 HOST autossh[63681]: port set to 0, monitoring disabled
# Aug 31 22:14:16 HOST autossh[63681]: starting ssh (count 1)
# Aug 31 22:14:16 HOST autossh[63681]: ssh child pid is 63682
```

## Connection Confirmation (From Client or Host)
``` bash
ssh $BASTION_SERVER # Connect to the bastion server
ssh localhost -p <YOUR_PORT> # Connect to the Host through the tunnel
# You may be prompted for a password initially
# Upon success, you should be logged into the Host without additional prompts
```


## Troubleshooting

- If the connection fails, check the service status:
  ```
  sudo systemctl status autossh-tunnel-<YOUR_PORT>.service
  ```
- Verify the SSH keys are correctly set up on both Host and Bastion Server.
- Ensure the PORT is not blocked by firewalls.

## Uninstallation
#### Host
```bash
sudo systemctl status autossh*.service | grep autossh-tunnel-
./remove-autossh-service.sh -p <YOUR_PORT>
```

## How to skip password confirmations

Register the client's public key to both the bastion and host servers to skip password verification:

```bash
cat "${SECRET_KEY_PATH}.pub" # copy the contents
# Add the copied contents to $BASTION_SERVER's $HOME/.ssh/authorized_keys
# Add the copied contents to Host's $HOME/.ssh/authorized_keys
```

## Security

- Keep your SSH keys secure and never share your private key. 
- Ensure appropriate permissions for SSH keys:
``` bash
ssh-correct-permissions ()
{ 
    chmod 700 ~/.ssh 2> /dev/null;
    chmod 600 ~/.ssh/* 2> /dev/null;
    chmod 644 ~/.ssh/*.pub 2> /dev/null;
    chmod 600 ~/.ssh/config 2> /dev/null
}
```

## Contact
Yusuke Watanabe (ywtanabe@alumni.u-tokyo.ac.jp)
