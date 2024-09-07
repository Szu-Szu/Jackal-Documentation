# Making a storage provider the Coney Daddy way

<div style="background-color: red; border: 1px solid #666; padding: 10px;">WARNING:Before you get started there are a few key elements here.  You'll need 10k $jkl, a running RPC, and a personal domain. These may take prep time.  If you can acquire all of these things then proceed!</div>

<p>
</p>

## Preparing your machine for the provider

It's good practice to `sudo apt update && apt upgrade -y` before installing new software.  Also, install git and other tools with `apt install git build-essential -y`.

### Installing GO
The provider code is written in GO.  We'll need to install it on your provider.  Just copy and poaste this code block:

```
rm -rf $HOME/go
rm -rf /usr/local/go
cd ~
curl https://dl.google.com/go/go1.21.1.linux-amd64.tar.gz | sudo tar -C/usr/local -zxvf -
cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF
source $HOME/.profile
```
Check to make sure GO is installed correctly with `go version`. It should return version 1.21.

## Installing Sequoia

We need to clone the github repo on to your machine.

`git clone https://github.com/JackalLabs/sequoia.git`

Next, go into the sequoia directory and install the code.

```
cd sequoia
make install
```
This installation should only take a minute.  After it's done type `sequoia` in and if the installation went as planned you should see something like this.

```
Sequoia is a fast and light-weight Jackal Storage Provider.

Usage:
  sequoia [command]

Available Commands:
  completion     Generate the autocompletion script for the specified shell
  help           Help about any command
  init           initializes sequoias config folder
  ipfs           Details about the IPFS network
  jprovd-salvage salvage unmigrated jprovd files to sequoia. jprovd directory required as arg
  start          Starts the provider
  terminate      Permanently remove provider from network and get deposit back
  version        checks the version of sequoia
  wallet         Wallet subcommands

Flags:
  -h, --help               help for sequoia
      --home string        sets the home directory for sequoia (default "$HOME/.sequoia")
      --log-level string   log level. info|error|debug (default "info")

Use "sequoia [command] --help" for more information about a command.
```

You can initiate your provider using `sequoia init`.

## Configuring your provider

Sequoia automagically generates the .sequoia directory with two important files in it. We'll review them one at a time. Let's get into the .sequoia directory.  `cd $HOME/.sequoia`.

We'll first take a look at the provder_wallet.json file.  This file contains the wallet address that sequoia automagically generated for you.  

<div style="background-color: blue; border: 1px solid #666; padding: 10px;">TIP: You can actually change this file edit the file and import the seedphrase of the wallet you wish to use.</div>

<p>
</p>

Let's take a look at the config.yaml file. Use ~~vim~~ nano to edit this file.  `nano config.yaml`.

You should see an output like this:

```
######################
### Sequoia Config ###
######################

queue_interval: 10
proof_interval: 120
stray_manager:
    check_interval: 30
    refresh_interval: 120
    hands: 2
chain_config:
    bech32_prefix: jkl
    rpc_addr: http://localhost:26657
    grpc_addr: localhost:9090
    gas_price: 0.02ujkl
    gas_adjustment: 1.5
domain: https://example.com
total_bytes_offered: 1092616192
data_directory: $HOME/.sequoia/data
api_config:
    port: 3333
    ipfs_port: 4005
    ipfs_domain: dns4/ipfs.example.com/tcp/4001
proof_threads: 1000

######################
```

There are a few lines we need to change here.  Let's do them one by one.

First let go to the line rpc_addr: http://localhost:26657.  Change the address to the IP address of your RPC that you previously made.  Keep the port there though! (26657)

Then to the next line grpc_addr: localhost:9090.  Change localhost to your RPC's IP address.  Once again keep the port. (9090).

Next it the domain line domain: https://example.com.  Change this line to the the personal domain that you have prepared.

The next line is the hard part. *Math*.  The Jackal provider understands things in bytes.  On the total_bytes_offered line you must enter in a number of bytes.  [Here](https://www.omnicalculator.com/conversion/byte-converter) is a good website to convert bytes to anything you might need.  

<div style="background-color: blue; border: 1px solid #666; padding: 10px;"> TIP: You can check how much space you have to offer by using the df -h command.</div>
<p>
</p>

Finally on the ipfs_domain: dns4/ipfs.example.com/tcp/4001 change the example.com part to your personal domain.

Exit your editor using ~~:q~~ Ctrl+x --> y--> Enter

## Starting your provider

All that is left is to start your provider.  Just do `sequoia start` and the provider will start!  Congratulations!  You are now participating in the Jackal provider network.

<div style="background-color: blue; border: 1px solid #666; padding: 10px;"> BONUS TIP: You can make a unit file to have this process go on in the background.</div>
<p>
</p>

```
sudo tee /etc/systemd/system/sequoiad.service > /dev/null <<EOF
[Unit]
Description=sequoia Daemon
After=network-online.target
[Service]
User=$USER
ExecStart=$(which sequoia) start
Restart=always
RestartSec=3
LimitNOFILE=infinity
[Install]
WantedBy=multi-user.target
EOF
```
`systemctl enable sequoiad.service`

`systemctl start sequoiad.service`










