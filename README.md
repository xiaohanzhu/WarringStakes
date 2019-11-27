# Instructions for Participating in the Meter Test Net

# Overview
In Meter, there are several types of full nodes in the network:
1. Regular full node: They sync for each block and can support interactions with wallets
2. Delegate nodes: These nodes are candidates for the committee nodes and have opportunity to propose and sign blocks.  To become a delegate node, the top N (N is a protocol parameter) staked full nodes(including both self staking and votes from other stakers) are the delegate nodes.
3. Committee nodes: A random subset of the delegate nodes are selected for every epoch.  These nodes form a committee quorum and perform consensus.  The committee nodes take turns to propose blocks.  If a proposed block receives endorsement signatures from 2/3 of nodes in the committee, the signatures form a QC (Quorum Certificate).  Each newly proposed block carries QC for the previous block.  Once the newly proposed block obtains a QC, the previous block is considered as confirmed and finalized.

In the test net and initial launch of the main net.  The number of Delegate Nodes will be the same as the number of the Committee nodes.

It is also required to run both PoS and PoW processes on the same physical or virtual machine to ensure security.

Requirements for running a delegate/committee node:
To achieve the full performance of the Meter network, the recommended hardware configuration is more than 8 vCPU, 16GB of memory and 100GB of SSD.  The maximum block size in Meter is around 1.3MB. It is also recommended to have data center class 1Gbps internet connection.  However the Meter consensus protocol is capable of adapting to transaction load, network and node processing speed to some extent by varying the block period from 2 sec to up to 30 sec.  The minimum requirement is 2 vCPU and 4GB of memory.

# Setting up a full node
Node software is currently provided as docker images.  Please refer to [Ubuntu Docker Installation Guide](https://phoenixnap.com/kb/how-to-install-docker-on-ubuntu-18-04).

1. Download [Meter desktop wallet](https://meter.io/developers) and create an account

2. Obtain the delegates.json file from the GitHub repo to your home directory (the following instructions assume delegates.json is in /home/ubuntu).  This file contains the nodes that validate the genesis blocks and also serves as backup delegate nodes to run the committee in case there is not enough staked delegate nodes to form the committee

3. Configure network ports for the nodes.  It is recommended to have a public IP address for your full node and have the following ports open for inbound TCP connections

| Port Range           | Functions        |
|----------------------|------------------|
| 9209                 | PoW P2P          |
| 8332                 | PoW API          |
| 8669                 | Wallet REST API  |
| 8670-8671            | PoW/PoS Messages |
| 55555                | Discovery Server |
| 11235                | PoS P2P          |
| 9100                 | node explorers   |

2. Launch the Meter container

```
sudo docker run -e DISCO_SERVER="enode://e1e364d452cc65b97e219ce10bcdbfbb87fb968c2752e3085db5c66dbb9e42ea18b31a5251bbb29bd95973abf6a496f0e213512bc402af76d981779c7e0cd6d1@3.90.46.4:55555"  -e POW_LEADER="34.220.171.70" -e COMMITTEE_SIZE="8" -e DELEGATE_SIZE="8" -v /home/ubuntu/delegates.json:/pos/delegates.json --network host --name metertest -d dfinlab/meter-all-in-one:latest
```
In the above command DISCO_SERVER points to the discovery server for other peers in the network. POW_LEADER points to node that could provide peer information for the PoW chain.  The two IP addresses should be the same.  The committee and delegate size should be configured properly based on the instructions of the test net.
It will download the latest container and launch a meter full node

Several useful commands for docker:

```
sudo docker container ls

```
The output will be like the following:
```
CONTAINER ID        IMAGE                      COMMAND                  CREATED             STATUS              PORTS               NAMES
260bbd571d1a        dfinlab/meter-all-in-one   "/usr/bin/supervisord"   23 hours ago        Up 23 hours                             metertest
```
```
sudo docker container stop metertest              //stop the container
sudo docker container start metertest             //stop the container
sudo docker container rm metertest                //remove the container
sudo docker container exec -it metertest bash   // launch a bash in the container
```
The log files can be located inside the container, under /var/log/supervisor directory.  If you file any bugs, please remember to attach the logs for PoS in the bug.
