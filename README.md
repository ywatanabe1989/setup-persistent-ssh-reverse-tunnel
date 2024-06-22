# Setup Persistent SSH Reverse Tunnel

## Getting Started

This repository contains scripts for setting up a persistent SSH reverse tunnel using autossh. It is designed to maintain a stable connection between a target server and a bastion server.

## Connection Overview
Diagrams below illustrates the SSH connections established by this setup:

#### Reverse SSH Tunneling
| Target Server | -> (Reverse SSH Tunneling) -> | Bastion Server|

#### Client Connection After Setup
| Client | -> | Bastion Server | -> (Port Forwarding) -> | Target Server |


## Prerequisites

- SSH access to both the bastion and target server.
- sudo privileges on the target server.


## Installation (on the target server)

1. **Generate SSH Key**: Without a passphrase for automation.

   ```bash
   ssh-keygen -t rsa -b 4096 -f $SECRET_KEY_PATH -N ""
   ```

2. **Register the Target Server's Public Key**:

   ```bash
   cat "${SECRET_KEY_PATH}.pub" # Copy the contents
   # Add the copied contents to $BASTION_SERVER's $HOME/.ssh/authorized_keys
   ```

3. **Define Parameters**:

   ```bash
   PORT=4800 # As you like
   BASTION_SERVER=<USER-NAME>@<EXTERNAL IP ADDRESS>
   SECRET_KEY_PATH=$HOME/.ssh/id_rsa
   ```

4. **Clone the Repository and Install autossh**:

   ```bash
   git clone https://github.com/ywatanabe1989/setup-persistent-ssh-reverse-tunnel.git
   cd setup-persistent-ssh-reverse-tunnel
   sudo yum install -y autossh
   ```

4. **Run the setup script**:
   ```bash
   ./setup-autossh-service.sh -p $PORT -b $BASTION_SERVER -s $SECRET_KEY_PATH
   # /etc/systemd/system/autossh-tunnel-"$PORT".service is created
   ```

5. **Check if it is ready for rebooting**:
<details>
<summary>Log</summary>
============================================================

YOU

Instruction:

Is it ready to reboot now??

(CREST)~$ sudo systemctl status autossh*.service
● autossh-tunnel-8939.service - AutoSSH tunnel service
     Loaded: loaded (]8;;file://CREST/etc/systemd/system/autossh-tunnel-8939.service/etc/systemd/system/autossh-tunnel-8939.service]8;;; enabled; preset: disabled)
     Active: active (running) since Sat 2024-06-22 19:53:51 JST; 20min ago
   Main PID: 112071 (autossh)
      Tasks: 2 (limit: 403832)
     Memory: 1.6M
        CPU: 98ms
     CGroup: /system.slice/autossh-tunnel-<PORT-1>.service
             ├─112071 /usr/bin/autossh -M 0 -N -o PubkeyAuthentication=yes -o PasswordAuthentication=no -i /home/<USER-NAME>/.ssh/id_rsa -R <PORT-1>:localhost:22 <USER-NAME>@<BATSION-SEVER-IP-ADDRESS>
             └─112072 /usr/bin/ssh -N -o PubkeyAuthentication=yes -o PasswordAuthentication=no -i /home/<USER-NAME>/.ssh/id_rsa -R <PORT-1>:localhost:22 <USER-NAME>@<BATSION-SEVER-IP-ADDRESS>

Jun 22 19:53:51 CREST systemd[1]: Started AutoSSH tunnel service.
Jun 22 19:53:51 CREST autossh[112071]: port set to 0, monitoring disabled
Jun 22 19:53:51 CREST autossh[112071]: starting ssh (count 1)
Jun 22 19:53:51 CREST autossh[112071]: ssh child pid is 112072

● autossh-tunnel-<PORT-2>.service - AutoSSH tunnel service
     Loaded: loaded (]8;;file://CREST/etc/systemd/system/autossh-tunnel-<PORT-2>.service/etc/systemd/system/autossh-tunnel-<PORT-2>.service]8;;; enabled; preset: disabled)
     Active: active (running) since Sat 2024-06-22 20:10:26 JST; 3min 38s ago
   Main PID: 113415 (autossh)
      Tasks: 2 (limit: 403832)
     Memory: 1.4M
        CPU: 28ms
     CGroup: /system.slice/autossh-tunnel-<PORT-2>.service
             ├─113415 /usr/bin/autossh -M 20002 -N -o PubkeyAuthentication=yes -o PasswordAuthentication=no -i /home/<USER-NAME>/.ssh/id_rsa -R <PORT-2>:localhost:22 <USER-NAME>@<BATSION-SEVER-IP-ADDRESS>
             └─113416 /usr/bin/ssh -L 20002:127.0.0.1:20002 -R 20002:127.0.0.1:20003 -N -o PubkeyAuthentication=yes -o PasswordAuthentication=no -i /home/<USER-NAME>/.ssh/id_rsa -R <PORT-2>:localhost:22 <USER-NAME>@3>

Jun 22 20:10:26 CREST systemd[1]: Started AutoSSH tunnel service.
Jun 22 20:10:26 CREST autossh[113415]: starting ssh (count 1)
Jun 22 20:10:26 CREST autossh[113415]: ssh child pid is 113416


============================================================

CLAUDE-3-5-SONNET-20240620

Yes, based on the status output you've provided, both autossh services appear to be running correctly now:

1. autossh-tunnel-<PORT-1>.service:
   - Active and running since 19:53:51 JST
   - No error messages visible

2. autossh-tunnel-<PORT-2>.service:
   - Active and running since 20:10:26 JST
   - No error messages visible

Both services are:
- Loaded and enabled
- In an active (running) state
- Showing their main processes and child SSH processes

This indicates that the issue with the <PORT-2> port has been resolved, and both tunnels are now operating as expected. The system should now be ready for a reboot, and these services should start automatically after the reboot due to their "enabled" status.

To ensure everything works as expected after a reboot:

1. Make sure there are no conflicting manual SSH commands in startup scripts or user login scripts.
2. After rebooting, check the status of these services again to confirm they've started correctly.

If you want to proceed with the reboot now, the autossh tunnels appear to be properly set up and should recover automatically.

============================================================
</details>



## Usage

1. **SSH into the Bastion Server**:

   ```bash
   ssh $BASTION_SERVER
   ```

2. **Test the Reverse Tunnel**:

   ```bash
   ssh localhost -p $PORT
   ```

## Removing the Service

To remove the established SSH reverse tunnel service:

```bash
sudo systemctl status autossh*.service | grep autossh-tunnel-
./remove-autossh-service.sh -p "$PORT"
```

## Appendix

Register the client's public key to both the bastion and target server to skip password verification:

```bash
cat "${SECRET_KEY_PATH}.pub" # copy the contents
# Add the copied contents to $BASTION_SERVER's $HOME/.ssh/authorized_keys
# Add the copied contents to Target Server's $HOME/.ssh/authorized_keys
```

## About

This project aims to facilitate the setup of secure, persistent SSH tunnels using autossh.

## Security

Ensure your SSH keys do not have a passphrase for automation purposes but are stored securely to prevent unauthorized access.
