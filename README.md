# setup-persistent-ssh-reverse-tunnel

## On the target server

1. Generate SSH key without a passphrase:
   ```bash
   ssh-keygen -t rsa -b 4096 -f $SECRET_KEY_PATH -N ""
   ```

2. Register the target srever's public key to the bastion server
   ```bash
   cat "${SECRET_KEY_PATH}.pub" # copy the contents
   # Add the copied contents to $BASTION_SERVER's $HOME/.ssh/authorized_keys
   ```

2. Define parameters:
   ```bash
   PORT=4800 # As you like
   BASTION_SERVER=<USER-NAME>@<EXTERNAL IP ADDRESS> # ywatanabe@xx.xx.xx.xx
   SECRET_KEY_PATH=$HOME/.ssh/id_rsa
   ```

3. Clone the repository and install autossh:
   ```bash
   git clone https://github.com/ywatanabe1989/setup-persistent-ssh-reverse-tunnel.git
   cd setup-persistent-ssh-reverse-tunnel
   sudo yum install -y autossh
   ```

4. Setup the reverse tunnel:
   ```bash
   ./setup-autossh-service.sh -p $PORT -b $BASTION_SERVER -s $SECRET_KEY_PATH
   # /etc/systemd/system/autossh-tunnel-"$PORT".service is created
   ```
   
5. SSH into the bastion server:
   ```bash
   ssh $BASTION_SERVER
   ```

6. Test the reverse tunnel:
   ```bash
   PORT=4800 # The designated value
   ssh localhost -p $PORT
   ```

7. Remove the service

``` bash
sudo systemctl status autossh*.service | grep autossh-tunnel-
./remove-autossh-service.sh -p "$PORT"
```



## Appendics
1. Register the client's public key to both the bastion server and the target server to skip password verification step:
   ```bash
   cat "${SECRET_KEY_PATH}.pub" # copy the contents
   # Add the copied contents to $BASTION_SERVER's $HOME/.ssh/authorized_keys
   # Add the copied contents to Target Server's $HOME/.ssh/authorized_keys
   ```
