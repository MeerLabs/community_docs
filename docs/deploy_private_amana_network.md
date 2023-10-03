# Deploy Private Amana Network

Amana is a POA network that uses the Clique consensus protocol. The instructions below will demonstrate how to create a private, local 3-node network. In practice, an Amana network would only require one node to operate the network.

## Requirements

`qng` Source Code - [https://github.com/Qitmeer/qng](https://github.com/Qitmeer/qng)

`go` - *`1.19.x`* or above (*`1.20.x`* is recommended)

`build-essential` (we’ll assume you’re using Ubuntu/Debian)

## Clone ****qng**** repository

Amana relies on the existing `qng` node and in our private network, also requires making certain modifications to the `qng/meerevm/amana/genesis.go` file before needing to be compiled. This requires cloning the repository from GitHub.

```bash
git clone https://github.com/Qitmeer/qng.git
```

## Create Directories

We will separate each of the nodes by creating three separate directories: `node1`, `node2`, and `node3`.

```bash
mkdir node1 node2 node3
```

## Generate Private Keys + Address

Amana POA requires each node to hold its own 256-bit private key for that node to become a sealer on the network. This private key will be used to derive an Ethereum address that must be unlocked to seal blocks. We will use Python and the `secrets` module included in the Python Standard Library (although you can use alternative means to randomly derive these keys). To generate a private key using `secrets`:

```bash
Python 3.10.6 (main, May 29 2023, 11:10:38) [GCC 11.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import secrets
>>> priv_key_1 = "0x" + secrets.token_hex(32)
>>> priv_key_1
'0x8c31ac305c9455ee7fdf025bb69a58bc08ee72e78a6fef9ca53cb17975811959'
>>> priv_key_2 = "0x" + secrets.token_hex(32)
>>> priv_key_2
'0x14abc2f5c4d55398985a9d44a7d0e0ad6e5d8743a3c3e25a79c930f69e12144c'
>>> priv_key_3 = "0x" + secrets.token_hex(32)
>>> priv_key_3
'0x8cb9f7d90138c0825a76715e00b2e073fd65da71c43ae51c339fd0c1a7b12d51'
```

To retrieve the public address from each of the private keys, we can use the Python package `eth-account`

```bash
Python 3.10.6 (main, May 29 2023, 11:10:38) [GCC 11.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from eth_account import Account
>>> address_1 = Account.from_key('0x8c31ac305c9455ee7fdf025bb69a58bc08ee72e78a6fef9ca53cb17975811959').address
>>> address_1
'0x160d02d70A278De6dfB45a8854319a30D62E9f8E'
>>> address_2 = Account.from_key('0x14abc2f5c4d55398985a9d44a7d0e0ad6e5d8743a3c3e25a79c930f69e12144c').address
>>> address_2
'0xE79544b2A7C0946b42f8e7Cc0bdD6371fa9AD285'
>>> address_3 = Account.from_key('0x8cb9f7d90138c0825a76715e00b2e073fd65da71c43ae51c339fd0c1a7b12d51').address
>>> address_3
'0xedC4c57154bF81c28b4ba6263758DA2729897cE5'
```

## Generating custom *Extra Data* and ******alloc****** Field

When initialising the genesis block, a custom `extraData` field is needed in order to list the authorised signers on the network. The formula for the `extraData` section in the genesis block is as follows:

`extraData` = Vanity (32 Byte) + Addresses of Authorised Signers + Signature (64 Byte)

In most cases, both vanity and signature are going to be set to all zeroes. 

A short Python script has been created to automate this process further:

```python
from eth_account import Account

vanity = "0" * 64   # 32 bytes
sealer_addr_1 = Account.from_key(PRIV_KEY_1).address[2:]    # Truncate "0x"
sealer_addr_2 = Account.from_key(PRIV_KEY_2).address[2:]
sealer_addr_3 = Account.from_key(PRIV_KEY_3).address[2:]

sig = "0" * 130     # 65 bytes
extraData = "0x" + vanity + sealer_addr_1 + sealer_addr_2 + sealer_addr_3 + sig
```

Once the custom `extraData` field has been generated, you will need to modify the `qng/meerevm/amana/genesis.go` file:

```go
package amana

import (
	mparams "github.com/Qitmeer/qng/meerevm/params"
	qparams "github.com/Qitmeer/qng/params"
	qcommon "github.com/ethereum/go-ethereum/common"
	"github.com/ethereum/go-ethereum/common/hexutil"
	"github.com/ethereum/go-ethereum/core"
	"github.com/ethereum/go-ethereum/params"
	"math/big"
)

func AmanaGenesis() *core.Genesis {
	return &core.Genesis{
		Config:     mparams.AmanaChainConfig,
		Nonce:      0,
		Number:     0,
		ExtraData:  hexutil.MustDecode("0x000000000000000000000000000000000000000000000000000000000000000071bc4403af41634cda7c32600a8024d54e7f64990000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"),
		GasLimit:   0x47b760,
		Difficulty: big.NewInt(1),
		Alloc:      decodePrealloc(),
		Timestamp:  uint64(qparams.MainNetParam.GenesisBlock.Block().Header.Timestamp.Unix()),
	}
}

func AmanaTestnetGenesis() *core.Genesis {
	return &core.Genesis{
		Config:     mparams.AmanaTestnetChainConfig,
		Nonce:      1,
		Number:     0,
		ExtraData:  hexutil.MustDecode("0x000000000000000000000000000000000000000000000000000000000000000071bc4403af41634cda7c32600a8024d54e7f64990000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"),
		GasLimit:   0x47b760,
		Difficulty: big.NewInt(1),
		Alloc:      decodePrealloc(),
		Timestamp:  uint64(qparams.TestNetParam.GenesisBlock.Block().Header.Timestamp.Unix()),
	}
}

func AmanaMixnetGenesis() *core.Genesis {
	return &core.Genesis{
		Config:     mparams.AmanaMixnetChainConfig,
		Nonce:      0,
		Number:     0,
		ExtraData:  hexutil.MustDecode("0x000000000000000000000000000000000000000000000000000000000000000071bc4403af41634cda7c32600a8024d54e7f64990000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"),
		GasLimit:   0x47b760,
		Difficulty: big.NewInt(1),
		Alloc:      decodePrealloc(),
		Timestamp:  uint64(qparams.MixNetParam.GenesisBlock.Block().Header.Timestamp.Unix()),
	}
}

func AmanaPrivnetGenesis() *core.Genesis {
	return &core.Genesis{
		Config:     mparams.AmanaPrivnetChainConfig,
		Nonce:      0,
		Number:     0,
		// This section needs to be modified
		ExtraData:  hexutil.MustDecode("0x0000000000000000000000000000000000000000000000000000000000000000160d02d70A278De6dfB45a8854319a30D62E9f8EE79544b2A7C0946b42f8e7Cc0bdD6371fa9AD285edC4c57154bF81c28b4ba6263758DA2729897cE50000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"),
		GasLimit:   0x47b760,
		Difficulty: big.NewInt(1),
		Alloc:      decodePrealloc(),
		Timestamp:  uint64(qparams.PrivNetParam.GenesisBlock.Block().Header.Timestamp.Unix()),
	}
}

func decodePrealloc() core.GenesisAlloc {
	ga := core.GenesisAlloc{}
	// ...and this too. The address and the amount allocated can be changed.
	ga[qcommon.HexToAddress("0x160d02d70A278De6dfB45a8854319a30D62E9f8E")] = core.GenesisAccount{Balance: big.NewInt(params.Ether).Mul(big.NewInt(params.Ether), big.NewInt(10000000000))}
	ga[qcommon.HexToAddress("0xE79544b2A7C0946b42f8e7Cc0bdD6371fa9AD285")] = core.GenesisAccount{Balance: big.NewInt(params.Ether).Mul(big.NewInt(params.Ether), big.NewInt(10000000000))}
	ga[qcommon.HexToAddress("0xedC4c57154bF81c28b4ba6263758DA2729897cE5")] = core.GenesisAccount{Balance: big.NewInt(params.Ether).Mul(big.NewInt(params.Ether), big.NewInt(10000000000))}
	return ga
}
```

In our example each of our sealers has `10000000000` Ether but this value and who recieves it can also be changed.

## Compile ***qng*** binary

Once `genesis.go` has been modified, you will need to compile the source code:

```bash
# Enter the qng root directory
cd ~/qng
make
```

If successful you should find a `qng` binary in the `qng/build/bin` directory. Copy this binary into the each of the node directories.

```bash
cp ~/qng/build/bin/qng node1
cp ~/qng/build/bin/qng node2
cp ~/qng/build/bin/qng node3
```

## Generate Node Files

To generate the neccesary files for your Amana network, run the following command `./qng -A ./ --privnet --amana --cleanup`for each Amana node.

```bash
./qng -A ./ --privnet --amana --cleanup
2023-09-30|06:17:18.417 [INFO ] System info                         QNG Version=1.0.24+dev-4ba0c35-dirty Go version=go1.20.5
2023-09-30|06:17:18.417 [INFO ] System info                         Home dir=/home/xboxuser/test-amana/node1
2023-09-30|06:17:18.417 [INFO ] Loading block database              dbPath=/home/xboxuser/test-amana/node1/data/privnet/blocks_ffldb
2023-09-30|06:17:18.448 [INFO ] Block database loaded 
2023-09-30|06:17:18.448 [INFO ] Removing block database from '/home/xboxuser/test-amana/node1/data/privnet/blocks_ffldb' 
2023-09-30|06:17:18.449 [INFO ] Finished cleanup:/home/xboxuser/test-amana/node1/data/privnet/meereth module=MEER
2023-09-30|06:17:18.449 [INFO ] Finished cleanup:/home/xboxuser/test-amana/node1/data/privnet/amana module=Amana
2023-09-30|06:17:18.449 [INFO ] Finished cleanup 
2023-09-30|06:17:18.449 [INFO ] Gracefully shutting down the database... 
2023-09-30|06:17:18.449 [INFO ] Shutdown complete
```

You should see two further directories created: `data` and `logs`. 

## Create Keystore

This private key must be encrypted (with a password as the encryption key) and stored in each node directory as a JSON keystore directory. We can use  `eth-account` to encrypt the private key using password “amana1” with the output saved onto a JSON file. 

```bash
# First create the keystore directories for each node
mkdir $NODE_1_DIRECTORY/data/privnet/keystore
mkdir $NODE_2_DIRECTORY/data/privnet/keystore
mkdir $NODE_3_DIRECTORY/data/privnet/keystore
```

We will use a Python script to automate the process of creating the keystore file, and generating the `password.txt` for each of the nodes.

```python
from eth_account import Account
import subprocess

with open(NODE_1_DIRECTORY + "/data/privnet/keystore/keystore_1.json", "w") as node1_file:
		# Encrypt our private key using the passphrase "amana1"
	  node_1_json = json.dumps(Account.encrypt(PRIV_KEY_1, "amana1"), indent=4)
	  node1_file.write(node_1_json)

with open(NODE_2_DIRECTORY + "/data/privnet/keystore/keystore_2.json", "w") as node2_file:
		node_2_json = json.dumps(Account.encrypt(PRIV_KEY_2, "amana1"), indent=4)
    node2_file.write(node_2_json)
   
with open(NODE_3_DIRECTORY + "/data/privnet/keystore/keystore_3.json", "w") as node3_file:
    node_3_json = json.dumps(Account.encrypt(PRIV_KEY_3, "amana1"), indent=4)
    node3_file.write(node_3_json)
    
# Password files to unlock accounts
subprocess.Popen("cp password.txt " + NODE_1_DIRECTORY, shell=True).wait()
subprocess.Popen("cp password.txt " + NODE_2_DIRECTORY, shell=True).wait()
subprocess.Popen("cp password.txt " + NODE_3_DIRECTORY, shell=True).wait()
```

## Generate Config file

Each node would need an configuration file with some changes needing to be made (usually port numbers if nodes are ran locally). Config files are in the TOML format.

Example config file:

```toml
privnet=true
amana=true
amanaenv="--unlock 0x160d02d70A278De6dfB45a8854319a30D62E9f8E --port 37000 --password password.txt --http --http.api=eth,net,web3,amana --http.corsdomain=* --http.port 8545 --allow-insecure-unlock miner.etherbase 0x160d02d70A278De6dfB45a8854319a30D62E9f8E --mine"
```

## Running Amana Node

To start the Amana node, run the command: `./qng -A ./ -C config.toml`

(`config.toml` is what we have named our configuration file).

```bash
./qng -A ./ -C config.toml
2023-09-30|06:40:14.734 [INFO ] System info                         QNG Version=1.0.24+dev-4ba0c35-dirty Go version=go1.20.5
2023-09-30|06:40:14.737 [INFO ] System info                         Home dir=./
2023-09-30|06:40:14.737 [INFO ] Loading block database              dbPath=/home/xboxuser/qng_privnet/node2/data/privnet/blocks_ffldb
2023-09-30|06:40:14.810 [INFO ] Block database loaded 
2023-09-30|06:40:14.816 [INFO ] transaction index is enabled        module=INDEX
2023-09-30|06:40:14.827 [INFO ] anticone size:4                     module=DAG
2023-09-30|06:40:14.830 [INFO ] System info                         module=EVM   ETH VM Version=meervm-v0.0.2 Go version=go1.20.5
2023-09-30|06:40:15.065 [INFO ] New local node record               seq=1696052415063 id=8642152b221a63ca ip=127.0.0.1 udp=8538 tcp=0
2023-09-30|06:40:15.107 [INFO ] Prepare meereth on NetWork(8133)... 
2023-09-30|06:40:15.113 [INFO ] Maximum peer count                  ETH=0 LES=0 total=0
2023-09-30|06:40:15.120 [INFO ] Smartcard socket not found, disabling err="stat /run/pcscd/pcscd.comm: no such file or directory"
2023-09-30|06:40:15.141 [WARN ] Sanitizing cache to Go's GC limits  provided=4096 updated=1303
2023-09-30|06:40:15.145 [INFO ] Set global gas cap                  cap=50000000
2023-09-30|06:40:15.150 [INFO ] Allocated trie memory caches        clean=195.00MiB dirty=325.00MiB
2023-09-30|06:40:15.154 [INFO ] Using leveldb as the backing database 
2023-09-30|06:40:15.155 [INFO ] Allocated cache and file handles    database=/home/xboxuser/qng_privnet/node2/data/privnet/meereth/chaindata cache=649.00MiB handles=524288
2023-09-30|06:40:15.232 [INFO ] Using LevelDB as the backing database 
2023-09-30|06:40:15.284 [INFO ] Opened ancient database             database=/home/xboxuser/qng_privnet/node2/data/privnet/meereth/chaindata/ancient/chain readonly=false
2023-09-30|06:40:15.340 [INFO ] Disk storage enabled for MeerEngine caches dir=/home/xboxuser/qng_privnet/node2/data/privnet/meereth/ethash count=3
2023-09-30|06:40:15.341 [INFO ] Disk storage enabled for MeerEngine DAGs dir=/home/xboxuser/qng_privnet/node2/data/privnet/meereth/ethash/dataset count=2
2023-09-30|06:40:15.345 [INFO ] Initialising Ethereum protocol      network=8133 dbversion=8
2023-09-30|06:40:16.144 [INFO ]  
2023-09-30|06:40:16.155 [INFO ] --------------------------------------------------------------------------------------------------------------------------------------------------------- 
2023-09-30|06:40:16.155 [INFO ] Chain ID:  8133 (qng-priv)
...

.__  __                                                                    
    _____|__|/  |_  _____   ____   ___________    QNG 1.0.24+dev-4ba0c35-dirty
   / ____/  \   __\/     \_/ __ \_/ __ \_  __ \   Port: 38130
  < <_|  |  ||  | |  Y Y  \  ___/\  ___/|  | \/   PID : 266551
   \__   |__||__| |__|_|  /\___  >\___  >__|      Network : privnet                      
      |__|              \/     \/     \/          https://github.com/Qitmeer/qng
```

## Running Proxy

To run the proxy enter the command: `python3 [proxy.py](http://proxy.py)`

```bash
python3 proxy.py
* Serving Flask app 'proxy'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on http://localhost:5000
Press CTRL+C to quit
127.0.0.1 - - [30/Sep/2023 06:49:54] "OPTIONS / HTTP/1.1" 200 -
127.0.0.1 - - [30/Sep/2023 06:49:54] "OPTIONS / HTTP/1.1" 200 -
{"method":"eth_getBlockByNumber","params":["latest",false],"id":42,"jsonrpc":"2.0"}
{"method":"eth_chainId","params":[],"id":43,"jsonrpc":"2.0"}
<Response [200]>
127.0.0.1 - - [30/Sep/2023 06:49:54] "POST / HTTP/1.1" 200 -
<Response [200]>
127.0.0.1 - - [30/Sep/2023 06:49:54] "POST / HTTP/1.1" 200 -
127.0.0.1 - - [30/Sep/2023 06:49:54] "OPTIONS / HTTP/1.1" 200 -
{"method":"net_version","params":[],"id":44,"jsonrpc":"2.0"}
<Response [200]>
127.0.0.1 - - [30/Sep/2023 06:49:54] "POST / HTTP/1.1" 200 -
127.0.0.1 - - [30/Sep/2023 06:49:54] "OPTIONS / HTTP/1.1" 200 -
{"method":"net_version","params":[],"id":45,"jsonrpc":"2.0"}
<Response [200]>
127.0.0.1 - - [30/Sep/2023 06:49:55] "POST / HTTP/1.1" 200 -
127.0.0.1 - - [30/Sep/2023 06:49:55] "OPTIONS / HTTP/1.1" 200 -
{"method":"eth_accounts","params":[],"id":46,"jsonrpc":"2.0"}
<Response [200]>
127.0.0.1 - - [30/Sep/2023 06:49:55] "POST / HTTP/1.1" 200 -
127.0.0.1 - - [30/Sep/2023 06:49:55] "OPTIONS / HTTP/1.1" 200 -
{"method":"eth_getBalance","params":["0xf39fd6e51aad88f6f4ce6ab8827279cfffb92266","latest"],"id":47,"jsonrpc":"2.0"}
<Response [200]>
127.0.0.1 - - [30/Sep/2023 06:49:55] "POST / HTTP/1.1" 200 -
127.0.0.1 - - [30/Sep/2023 06:49:55] "OPTIONS / HTTP/1.1" 200 -
{"method":"eth_getBalance","params":["0x70997970c51812dc3a010c7d01b50e0d17dc79c8","latest"],"id":48,"jsonrpc":"2.0"}
<Response [200]>
```

The verbose output is the result of connecting the proxy to Remix IDE which sends multiple JSON-RPC requests very frequently. 

## Deploy ACL Smart Contract to Amana

Because Amana is EVM compatible, the steps needed to deploy a smart contract is very similar to other EVM compatible networks. We can use `Hardhat` and the `ethers.js` library to create a JS script to automate the deployment. The script would also insert an address(in this case `0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266`) to be put into the whitelist.

```jsx
// deploy.js 
const fs = require('fs');

async function main() {
    const [deployer] = await ethers.getSigners();
  
    console.log("Deploying contracts with the account:", deployer.address);
    const permission = await ethers.deployContract("Permission");

    const address = await permission.getAddress();
    console.log("Contract Address:", address);

    const content = "ADDRESS="+ address;

    try {
        fs.writeFileSync(".env", content);
        // file written successfully
    } catch (err) {
        console.error(err);
    }

    // Insert test data
   let tx = await permission.addToWhitelist("0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266");
    await tx.wait()

  }
  
  main()
    .then(() => process.exit(0))
    .catch((error) => {
      console.error(error);
      process.exit(1);
    });
```