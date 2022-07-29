# near-stakewars-iii running validator node for shardnet on Hetzner cloud provider
1.register a wallet，register address:https://wallet.shardnet.near.org/，create a successful example
![1659081863656](https://user-images.githubusercontent.com/100293899/181713743-fb0c0015-7e58-4e7b-b4a4-b15f0237377c.png)


2.log in to <a href="https://accounts.hetzner.com/login">hetzner</a>, if you don't have an account, please complete the registration first .

3.Buy a 4-core 8G 500G server, choose Ubuntu 20.04 as the operating system for example
![1659081237390](https://user-images.githubusercontent.com/100293899/181713065-d2f16500-455c-4419-8d1c-935e22a12039.png)

4. ssh login server
![1659082056452](https://user-images.githubusercontent.com/100293899/181714223-cc660cc0-b28d-4781-9e16-49c54d208b84.png)

5.make sure the linux machine is up-to-date.
```
sudo apt update && sudo apt upgrade -y
```

6.Install developer tools, Node.js, and npm
```
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -  
sudo apt install build-essential nodejs
PATH="$PATH"
```

7.Install NEAR-CLI and select the correct network
```
sudo npm install -g near-cli
export NEAR_ENV=shardnet
echo 'export NEAR_ENV=shardnet' >> ~/.bashrc
```

8.NEAR CLI Commands Guide
```
near proposals
near validators current
near validators next
```

9.Install developer tools:
```
sudo apt install -y git binutils-dev libcurl4-openssl-dev zlib1g-dev libdw-dev libiberty-dev cmake gcc g++ python docker.io protobuf-compiler libssl-dev pkg-config clang llvm cargo
sudo apt install python3-pip
```

10.Set the configuration
```
USER_BASE_BIN=$(python3 -m site --user-base)/bin
export PATH="$USER_BASE_BIN:$PATH"
```

11.Building env&Install Rust & Cargo
```
sudo apt install clang build-essential make
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
```

12.Clone nearcore project from GitHub
```
git clone https://github.com/near/nearcore
cd nearcore
git fetch
git checkout c1b047b8187accbf6bd16539feb7bb60185bdc38
```

13.Compile nearcore binary
```
cargo build -p neard --release --features shardnet
```

14.Initialize working directory & Replace the config.json
```
./target/release/neard --home ~/.near init --chain-id shardnet --download-genesis
rm ~/.near/config.json
wget -O ~/.near/config.json https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/config.json
```

15.Install AWS Cli
```
sudo apt-get install awscli -y
```

16.Run the node
```
cd ~/nearcore
./target/release/neard --home ~/.near run
```
![1659085163059](https://user-images.githubusercontent.com/100293899/181723959-a0619ad6-4a1b-49c4-81e7-ce65449a91e4.png)
The node is now running you can see log outputs in your console. Your node should be find peers, download headers to 100%, and then download blocks.

17.Authorize Wallet Locally
```
near login
```
Copy the link in your browser

![1659085338515](https://user-images.githubusercontent.com/100293899/181724513-ca3952e9-1a9d-4342-9749-0ec557314176.png)

Grant Access to Near CLI

![1659085407557](https://user-images.githubusercontent.com/100293899/181724854-f5aaaaa5-3b83-4e41-a438-dae6ae1d0a9a.png)

After Grant, you will see a page like this, go back to console

![1659085492240](https://user-images.githubusercontent.com/100293899/181724974-37119df0-8287-4da2-9e93-cab7bb34ee05.png)

 Enter your wallet and press Enter
 
 ![1659085539454](https://user-images.githubusercontent.com/100293899/181725130-1868b552-dd3b-4200-a2c0-6c0e55f02b61.png)
 
 18.Create a validator_key.json
 ```
 near generate-key andots
 cp ~/.near-credentials/shardnet/andots.factory.shardnet.near.json ~/.near/validator_key.json
 ```
 
 19.Change private_key to secret_key(vim ~/.near/validator_key.json),File content must be in the following pattern:
  ```
  {
  "account_id": "xx.factory.shardnet.near",
  "public_key": "ed25519:HeaBJ3xLgvZacQWmEctTeUqyfSU4SDEnEwckWxd92W2G",
  "secret_key": "ed25519:****"
 }
  ```

20.Setup Systemd Command:
 ```
sudo vi /etc/systemd/system/neard.service
 ```
 Paste:
  ```
 [Unit]
Description=NEARd Daemon Service

[Service]
Type=simple
User=root
#Group=near
WorkingDirectory=/root/.near
ExecStart=/root/nearcore/target/release/neard run
Restart=on-failure
RestartSec=30
KillSignal=SIGINT
TimeoutStopSec=45
KillMode=mixed

[Install]
WantedBy=multi-user.target
 ```
 Command:
  ```
  sudo systemctl enable neard
  sudo systemctl start neard
  ```
  
 Watch logs
```
journalctl -n 100 -f -u neard
```
log after successful installation

![1659086542132](https://user-images.githubusercontent.com/100293899/181728436-9d175e46-622c-4539-8e05-71adde3376d2.png)
