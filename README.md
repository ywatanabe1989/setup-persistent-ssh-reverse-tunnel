# Setup Persistent SSH Reverse Tunnel

## Getting Started

This repository contains scripts for setting up a persistent SSH reverse tunnel using autossh. It is designed to maintain a stable connection between a target server and a bastion server.

## Prerequisites

- SSH access to both the bastion and target server.
- sudo privileges on the target server.

## Installation

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

<!-- ## Installation
 !-- 
 !-- 1. **Generate SSH Key**: Without a passphrase for automation.
 !--    ```bash
 !--    ssh-keygen -t rsa -b 4096 -f $SECRET_KEY_PATH -N ""# setup-persistent-ssh-reverse-tunnel
 !-- 
 !-- ## On the target server
 !-- 
 !-- 1. Generate SSH key without a passphrase:
 !--    ```bash
 !--    ssh-keygen -t rsa -b 4096 -f $SECRET_KEY_PATH -N ""
 !--    ```
 !-- 
 !-- 2. Register the target srever's public key to the bastion server
 !--    ```bash
 !--    cat "${SECRET_KEY_PATH}.pub" # copy the contents
 !--    # Add the copied contents to $BASTION_SERVER's $HOME/.ssh/authorized_keys
 !--    ```
 !-- 
 !-- 2. Define parameters:
 !--    ```bash
 !--    PORT=4800 # As you like
 !--    BASTION_SERVER=<USER-NAME>@<EXTERNAL IP ADDRESS> # ywatanabe@xx.xx.xx.xx
 !--    SECRET_KEY_PATH=$HOME/.ssh/id_rsa
 !--    ```
 !-- 
 !-- 3. Clone the repository and install autossh:
 !--    ```bash
 !--    git clone https://github.com/ywatanabe1989/setup-persistent-ssh-reverse-tunnel.git
 !--    cd setup-persistent-ssh-reverse-tunnel
 !--    sudo yum install -y autossh
 !--    ```
 !-- 
 !-- 4. Setup the reverse tunnel:
 !--    ```bash
 !--    ./setup-autossh-service.sh -p $PORT -b $BASTION_SERVER -s $SECRET_KEY_PATH
 !--    # /etc/systemd/system/autossh-tunnel-"$PORT".service is created
 !--    ```
 !--    
 !-- 5. SSH into the bastion server:
 !--    ```bash
 !--    ssh $BASTION_SERVER
 !--    ```
 !-- 
 !-- 6. Test the reverse tunnel:
 !--    ```bash
 !--    PORT=4800 # The designated value
 !--    ssh localhost -p $PORT
 !--    ```
 !-- 
 !-- 7. Remove the service
 !-- 
 !-- ``` bash
 !-- sudo systemctl status autossh*.service | grep autossh-tunnel-
 !-- ./remove-autossh-service.sh -p "$PORT"
 !-- ```
 !-- 
 !-- 
 !-- 
 !-- ## Appendics
 !-- 1. Register the client's public key to both the bastion server and the target server to skip password verification step:
 !--    ```bash
 !--    cat "${SECRET_KEY_PATH}.pub" # copy the contents
 !--    # Add the copied contents to $BASTION_SERVER's $HOME/.ssh/authorized_keys
 !--    # Add the copied contents to Target Server's $HOME/.ssh/authorized_keys
 !--    ``` -->
