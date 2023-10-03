# AWS - Deploy Amana Privnet

A public AMI (Amazon Machine Image) has been created to allow the quick deployment of EC2 instances with Amana nodes and all its dependencies already preconfigured. 

To launch an EC2 instance with the Amana Privnet AMI, on the EC2 Management console, click on **Launch instances**. In the section **Application and OS Images (Amazon Machine Image)**, click on **Browse more AMIs**. 

In the search bar, type in “**Meer Labs - Amana Privnet**” or the AMI ID, “**ami-07aca94d110e01229**” and click on the **Community AMIs** tab. You should see a single AMI entry labelled **"Meer Labs - Amana Privnet”**

![aws_ami.PNG](https://raw.githubusercontent.com/MeerLabs/community_docs_assets/main/img/aws_ami.PNG)

Click select and enter the remaining configuration details for your EC2 instance.


> ⚠️ The size of the EBS volumes (storage) must be set to at least 20 GiB. This is due to the first instance which created this AMI being configured with a storage space of 20 GiB.


> ⚠️ The minimum instance type that may be able to run all three nodes inside one instance is the `t3.micro`. Although it is recommended to pick an instance type larger than this such as `t3.small` or `t3.medium`.


## Running Amana Nodes

When first running your EC2 instance, you should find a directory called `qng_privnet` where the nodes for the Amana network reside. Inside, will contain the directories `node1`, `node2` and `node3` each preconfigured with their own configuration files. There should also be a JSON file called `private_keys.json` which contains all the addresses and private keys for each of the nodes.

To run a node, enter the directory for the node you wish to run and enter the command: `./qng -A ./ -C <config-file>.toml`. Each node will have its own uniquely named `config.toml` file. 

```bash
cd ~/qng_privnet
cd node1
./qng -A ./ -C config_1.toml

2023-10-01|05:11:50.672 [INFO ] System info                         QNG Version=1.1.0+dev-bde9231-dirty Go version=go1.20.8
2023-10-01|05:11:50.711 [INFO ] System info                         Home dir=./
2023-10-01|05:11:50.712 [INFO ] Loading block database              module=LCDB dbPath=/home/ubuntu/qng_privnet/node1/data/privnet/blocks_ffldb
2023-10-01|05:11:50.730 [INFO ] Block database loaded               module=LCDB
2023-10-01|05:11:50.731 [INFO ] transaction index is enabled        module=INDEX
2023-10-01|05:11:50.732 [INFO ] anticone size:4                     module=DAG
2023-10-01|05:11:50.735 [INFO ] Meer chain                          module=MEER  version=meervm-v0.0.2
2023-10-01|05:11:50.939 [INFO ] New local node record               seq=1696137110938 id=b708dc642e50c371 ip=127.0.0.1 udp=8538 tcp=0
2023-10-01|05:11:50.949 [INFO ] Prepare meereth on NetWork(8133)... 
2023-10-01|05:11:50.950 [INFO ] Maximum peer count                  ETH=0 LES=0 total=0
2023-10-01|05:11:50.952 [INFO ] Smartcard socket not found, disabling err="stat /run/pcscd/pcscd.comm: no such file or directory"
2023-10-01|05:11:50.956 [WARN ] Sanitizing cache to Go's GC limits  provided=4096 updated=313
2023-10-01|05:11:50.957 [INFO ] Set global gas cap                  cap=50000000
2023-10-01|05:11:50.957 [INFO ] Initializing the KZG library        backend=gokzg
2023-10-01|05:11:51.151 [INFO ] Allocated trie memory caches        clean=46.00MiB dirty=78.00MiB
2023-10-01|05:11:51.151 [INFO ] Defaulting to pebble as the backing database 
2023-10-01|05:11:51.152 [INFO ] Allocated cache and file handles    database=/home/ubuntu/qng_privnet/node1/data/privnet/meereth/chaindata cache=156.00MiB handles=524288
2023-10-01|05:11:51.199 [INFO ] Opened ancient database             database=/home/ubuntu/qng_privnet/node1/data/privnet/meereth/chaindata/ancient/chain readonly=false
2023-10-01|05:11:51.201 [INFO ] Initialising Ethereum protocol      network=8133 dbversion=<nil>
2023-10-01|05:11:51.201 [INFO ] Writing custom genesis block 
2023-10-01|05:11:52.368 [INFO ] Persisted trie from memory database nodes=36705 size=3.41MiB time=156.475635ms gcnodes=0 gcsize=0.00B gctime=0s livenodes=0 livesize=0.00B
```

If the nodes are running successfully, you should see an output like this:

```bash
2023-10-01|05:13:53.007 [INFO ] Imported new chain segment          number=9 hash=bbf56d..954f8c                                                     blocks=1 txs=0 mgas=0.000 elapsed=357.829µs mgasps=0.000 dirty=0.00B
2023-10-01|05:13:53.008 [INFO ] Commit new sealing work             number=10 sealhash=688fd9..dc6e55 uncles=0 txs=0 gas=0 fees=0 elapsed=239.493µs
2023-10-01|05:13:53.009 [INFO ] Commit new sealing work             number=10 sealhash=688fd9..dc6e55 uncles=0 txs=0 gas=0 fees=0 elapsed=821.172µs
2023-10-01|05:13:56.001 [INFO ] Successfully sealed new block       number=10 sealhash=688fd9..dc6e55 hash=c526d9..7ada52                                                     elapsed=2.992s
2023-10-01|05:13:56.001 [INFO ] 🔨 mined potential block             number=10 hash=c526d9..7ada52
2023-10-01|05:13:56.002 [INFO ] Commit new sealing work             number=11 sealhash=82e681..6cc56c uncles=0 txs=0 gas=0 fees=0 elapsed=655.949µs
2023-10-01|05:13:56.003 [WARN ] Block sealing failed                err="signed recently, must wait for others"
2023-10-01|05:13:56.003 [INFO ] Commit new sealing work             number=11 sealhash=82e681..6cc56c uncles=0 txs=0 gas=0 fees=0 elapsed=2.259ms
2023-10-01|05:13:59.002 [INFO ] Imported new chain segment          number=11 hash=64c417..9a54b6                                                     blocks=1 txs=0 mgas=0.000 elapsed=285.388µs mgasps=0.000 dirty=0.00B
2023-10-01|05:13:59.003 [INFO ] Commit new sealing work             number=12 sealhash=ae4064..8a92e9 uncles=0 txs=0 gas=0 fees=0 elapsed=202.539µs
2023-10-01|05:13:59.004 [INFO ] Commit new sealing work             number=12 sealhash=ae4064..8a92e9 uncles=0 txs=0 gas=0 fees=0 elapsed=1.019ms
2023-10-01|05:13:59.298 [INFO ] Looking for peers                   peercount=2 tried=0 static=0
2023-10-01|05:14:02.002 [INFO ] Imported new chain segment          number=12 hash=ea2ec4..78ffc8
```


> ⚠️ The instructions here are only applicable for running multiple nodes locally on one EC2 instance. If you wish to run the nodes on separate instances, than you must make certain changes to the networking configuration. This may include changes to the config file as well as changes to your EC2 *Security Groups*.

