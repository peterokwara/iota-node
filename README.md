# IOTA Node

Configuration files for launching a iota node on a cloud server using docker. 

This is for
- A single node
- Multiple nodes connected to each other
- A private node network

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See deployment for notes on how to deploy the project on a live system.

### Prerequisites

The following software needs to be installed

- docker
- docker-compose
- git

### Installing

- Before installing, ensure the following ports are open. This is both in the firewall software running on the server and on   the firewall provided by the cloud service provider, if the node is being run in the cloud.

  ```
  14626 UDP - Autopeering port
  15600 TCP - Gossip (neighbors) port
  14265 TCP - API port (optional if you don't want to access your node's API from external)
  ```

- Instructions on how to set up docker, docker-commpose and ufw (on Ubuntu Server 18.04) can be found here
  - Docker https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04
  - Docker-compose https://www.digitalocean.com/community/tutorials/how-to-install-docker-compose-on-ubuntu-18-04
  - UFW (Uncomplicated Firewall) https://www.digitalocean.com/community/tutorials/    how-to-set-up-a-firewall-with-ufw-on-ubuntu-18-04

#### Single node

1. The first step is to clone the repository by running:

    ```
    git clone https://github.com/peterokwara/iota-node.git
    ```

2. With the repository cloned, navigate to the `iota-node/hornet/` directory and run:

    ```
    sudo docker-compose up -d
    ```

3. This allows one to run docker in a detached mode. To view the logs, run:

    ```
    sudo docker-compose logs -f
    ```

    This streams the log file of what is happening within the docker file. 

4. To shutdown the node, you can simply run:

    ```
    sudo docker-compose down
    ```

#### Multiple nodes connected to each other

1. The first step is to clone the repository by running:

    ```
    git clone https://github.com/peterokwara/iota-node.git
    ```

2. The next step is to add neighbours. Go to the `iota-node/hornet/config/peering.json` and add the url and the port number of the other nodes you need to connect to under peers. 

    ```
    {
      "acceptAnyConnection": false,
      "maxPeers": 5,
      "peers": [
        {
          "identity": "example.neighbor.com:15600",
          "alias": "Example Peer",
          "preferIPv6": false
        }
      ]
    }
    ```

    Multiple nodes can be added by just adding another json object shown below

    ```
    {
      "acceptAnyConnection": false,
      "maxPeers": 5,
      "peers": [
        {
          "identity": "example.neighbor1.com:15600",
          "alias": "Example Peer 1",
          "preferIPv6": false
        },
        {
          "identity": "example.neighbo2r.com:15600",
          "alias": "Example Peer 2",
          "preferIPv6": false
        }
      ]
    }
    ```

    Make sure the node address and port numbers are added on both sides for the nodes to successfully talk to each other.

3. Once this is done, you can now move into the `iota-node/hornet` folder and run:

    ```
    docker-compose up -d
    ```

4. This allows one to run docker in a detached mode. To view the logs, run:

    ```
    docker-compose logs -f
    ```

    This streams the log file of what is happening within the docker file. 

5. To shutdown the node, you can simply run:

    ```
    docker-compose down
    ```
### Private node

1. The first step is to generate a seed for the Coordinator and for the Snapshot. One way of doing this is buy runnig this command twice in a linux machine:

    ```
    cat /dev/urandom |tr -dc A-Z9|head -c${1:-81}
    ```
    
    Examples on how to generate a seed for other platforms (Mac/Linux) can be found [here](https://docs.iota.org/docs/getting-started/0.1/tutorials/create-a-seed)
    
    **WARNING: Do not expose the seeds that you have created. This will leave you vunerable to an attack. Keep your seeds safe.**

2. Change `.example.env` file found in `iota-node/private_node/hornet/layers_calculator` and the `iota-node/private_node/hornet`  to a `.env` file. Fill it with the seed.

    From:

    ```
    COO_SEED=
    ```
    To
    
    ```
    COO_SEED=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    ```
    
    Where `XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX` represents the seed you need to put.
    
3. Take the second seed that you generated. Navigate to the `iota-node/private_hornet/hornet/config` folder. Create a snapshot.csv file. Add the seed as shown below
 
    ```
    XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX;2779530283277761
    ```
    
    Where `XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX` represents the seed you need to put.
    
4. We then need to change the `config.json` found in `iota-node/private_hornet/hornet/config` folder. Enable the coordinator plugin by adding "Coordinator" in "enablePlugins"

    ```
    "node":{
        "alias": "Coordinator",
        "showAliasInGetNodeInfo": false,
        "disablePlugins": [],
        "enablePlugins": ["Coordinator"]
        },
    ```
    
    In the `coordinator` section of the `config.json` file, add the following settings
    
    Some hints:
    - Set a low MWM. If you are the only one using this Tangle, you need no (high) spam protection
    - The higher the merkle tree depth, the longer it will take to compute the merkle tree. Calculate how many milestones you really need:
    Number of possible milestones: 2merkleTreeDepth
    
    ```
    "coordinator":{
        "address":"",
        "securityLevel":2,
        "merkleTreeDepth":18,
        "mwm":5,
        "stateFilePath":"coordinator.state",
        "merkleTreeFilePath":"coordinator.tree",
        "intervalSeconds":60,
        "checkpointTransactions":5
    },
    ```
    For the snapshot, ensure it looks like this:

    ```
    "snapshots": {
        "loadType": "global",
        "global": {
            "path": "snapshot.csv",
            "spentAddressesPaths": [],
            "index": 0
        }
    },
    ```
5. The next step is to generate the address of our coordinator. To do this, go to the `iota-node/private_node/layers_calculator` and run 

    ```
    sudo docker-compose up -d
    ```

6. This allows one to run docker in a detached mode. To view the logs, run:

    ```
    docker-compose logs -f
    ```

    This streams the log file of what is happening within the docker file. The result should look something like this.
    
    ```
    calculating 1024 addresses...
    calculated 1024/1024 (100.00%) addresses (took 1s).
    calculating nodes for layer 9
    calculating nodes for layer 8
    calculating nodes for layer 7
    calculating nodes for layer 6
    calculating nodes for layer 5
    calculating nodes for layer 4
    calculating nodes for layer 3
    calculating nodes for layer 2
    calculating nodes for layer 1
    calculating nodes for layer 0
    merkle tree root: BHKJSBMRSZLFMXJFYE9NHYTZCRAZQHLZTIBTKVNZLVWAXKESPOANYARWQYOYYHONDJYEAMMSOQEGGEPKB
    successfully created merkle tree (took 1s).
    ```
7. Once it has finished and the `merkle tree root` is displayed after running

    ```
    docker-compose logs -f
    ```
    
    You can then copy the `merkle tree root` to the config file in the `iota-node/private_hornet/hornet/config` directory under `coordinator address` as shown below.
    
    ```
    "coordinator":{
        "address":"BHKJSBMRSZLFMXJFYE9NHYTZCRAZQHLZTIBTKVNZLVWAXKESPOANYARWQYOYYHONDJYEAMMSOQEGGEPKB",
        "securityLevel":2,
        "merkleTreeDepth":18,
        "mwm":5,
        "stateFilePath":"coordinator.state",
        "merkleTreeFilePath":"coordinator.tree",
        "intervalSeconds":60,
        "checkpointTransactions":5
    },
    ```

4. We can now bootstrap and run the hornet coordinator by navigating to the `iota-node/private_hornet/hornet/` and running

    ```
    sudo docker-compose up -d
    ```
    
5. This allows one to run docker in a detached mode. To view the logs, run:

    ```
    docker-compose logs -f
    ```

    This streams the log file of what is happening within the docker file. To know if the bootstrap process is complete, We will start seeing checkpoints being issued as shown below.
    
    ```
    2020-06-03T20:56:48Z    INFO    Coordinator     checkpoint issued (2/5)
    req(qu/pe/proc/lat): 00000/00000/00000/0000ms, reqQMs: 0, processor: 00000, LSMI/LMI: 19/19, TPS (in/new/out): 00000/00000/00000
    req(qu/pe/proc/lat): 00000/00000/00000/0000ms, reqQMs: 0, processor: 00000, LSMI/LMI: 19/19, TPS (in/new/out): 00000/00000/00000
    ```
10. To install an additonal hornet, in your new machine,The first step is to clone the repository by running:
    
    ```
    git clone https://github.com/peterokwara/iota-node.git
    ```
    
    Copy over the config.json from `iota-node/private_hornet/hornet/config` but disable the coordinator plugin by ensuring the node section looks like this:

    ```
    "node":{
    "alias": "Coordinator",
    "showAliasInGetNodeInfo": false,
    "disablePlugins": [],
    "enablePlugins": []
    },
    ```

    Copy over the snapshot.csv file from `iota-node/private_hornet/hornet/config`, move to the `iota-node/private_hornet/hornet` directory and run: 

    ```
    sudo docker-compose up -d
    ```
    
    This allows one to run docker in a detached mode. To view the logs, run:
    
    ```
    docker-compose logs -f
    ```
    
## Built With

* [GoHornet](https://github.com/gohornet/hornet) - The IOTA fullnode software 
* [Compass](https://github.com/iotaledger/compass) - The IOTA Network coordinator

## Contributing

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details on our code of conduct, and the process for submitting pull requests to us.


## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

## References

* Installing hornet https://github.com/gohornet/hornet/wiki/Tutorials%3A-Linux%3A-Install-HORNET
* Setting up a private tangle https://github.com/gohornet/hornet/wiki/Tutorials%3A-Private-Tangle
