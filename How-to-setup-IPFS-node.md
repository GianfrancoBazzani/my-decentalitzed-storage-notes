
# How to setup IPFS node (linux)

## download and install Kubo

Kubo is the most widely used implementation of IPFS.

Download the Linux binary from [dist.ipfs.io](https://dist.ipfs.io/#kubo):

````
wget https://dist.ipfs.io/kubo/v0.14.0/kubo_v0.14.0_linux-amd64.tar.gz
````

Unzip tar file:

````
tar -xvzf kubo_v0.14.0_linux-amd64.tar.gz
````

Move into the kubo folder and run the install script:

````
cd kubo
sudo bash install.sh
````

Check installation

````
ipfs --version
````

> ipfs version 0.14.0

## Start IPFS node daemon

In a new terminal run:

````
ipfs daemon
````

You have to wait until `Daemon is ready` message apears.

Now we can check that the daemon process is running:

````
ps aux | grep ipfs
````

> pc          4657  3.8  1.1 1280684 89736 pts/0   Sl   15:38   0:00 ipfs daemon

And that it is listening to the correct ports:

````
netstat -tunlp | grep ipfs
````

>tcp        0      0 127.0.0.1:8080          0.0.0.0:*               LISTEN      4657/ipfs           
tcp        0      0 0.0.0.0:4001            0.0.0.0:*               LISTEN      4657/ipfs           
tcp        0      0 127.0.0.1:5001          0.0.0.0:*               LISTEN      4657/ipfs           
tcp6       0      0 :::4001                 :::*                    LISTEN      4657/ipfs           
udp        0      0 0.0.0.0:4001            0.0.0.0:*                           4657/ipfs           
udp        0      0 0.0.0.0:5353            0.0.0.0:*                           4657/ipfs           
udp        0      0 0.0.0.0:5353            0.0.0.0:*                           4657/ipfs           
udp6       0      0 :::4001                 :::*                                4657/ipfs           
udp6       0      0 :::5353                 :::*                                4657/ipfs           
udp6       0      0 :::5353                 :::*                                4657/ipfs           

Ports description:

* 4001: default libp2p swarm port - should be open to public for all nodes if possible
* 5001: API port - provides write/admin access to the node, shouldn't be exposed at all
* 8080: HTTP Gateway to IPFS network + read only API subset 

In order to the other nodes of the network being able to connect to our node, we have to configure NAT on the router and forward the requests to port 4001 to our node private IP, otherwise we will not be able to receive incoming communications initiated by other nodes, and as a result we are unable to respond their requests.

Check your peers, the peers are listed using the following format:

`"Transport layer address"`/p2p/`"PeerID"`

````
ipfs swarm peers
````

> /ip4/104.131.131.82/tcp/4001/p2p/QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ
/ip4/104.236.151.122/tcp/4001/p2p/QmSoLju6m7xTh3DuokvT3886QRYqxAzb1kShaanJgW36yx
/ip4/134.121.64.93/tcp/1035/p2p/QmWHyrPWQnsz1wxHR219ooJDYTvxJPyZuDUPSDpdsAovN5
/ip4/178.62.8.190/tcp/4002/p2p/QmdXzZ25cyzSF99csCQmmPZ1NTbWTe8qtKFaZKpZQPdTFB

You can access to Web GUI in: `localhost:5001/webui`

## Add files and get files from network.

To add files to IPFS network we use add command, this command returns the CID of the file:
````
ipfs add "filename"
````

To retrieve a file from the network we use get command:

````
ipfs get "CID" > "destination filename"
````

To view a file from the network we use cat command:

````
ipfs cat "CID"
````

## Pinning files
Pinning is the mechanism that allows you to tell IPFS to always keep a given object somewhere — the default being your local node, though this can be different if you use a third-party remote pinning service.
To prevent that garbage collection, simply pin the CID you care about, objects added through `ipfs add` are pinned recursively by default.

### Pin types:
* Direct pins, which pin just a single block and no others in relation to it.
* Recursive pins, which pin a given block and all of its children.
* Indirect pins, which are the result of a given block's parent being pinned recursively.

To list the objects that you have pinned to your local storage.

````
ipfs pin ls --type=<pin type>
````

> "CID" "pin type"


To unpin that you have pinned to your local storage.

````
ipfs pin rm "CID"
````

To run the garbage collection.

````
ipfs repo gc              
````

## MFS mutable file system
MFS is accessed through the file commands in the IPFS CLI and API. Data added to the Mutable File System (MFS) is protected from garbage collection in the same way as local pinning but is somewhat easier to manage. To use it via command line:

To create a directory:

````
ipfs files mkdir "/path_in_MFS"            
````


To add a local file to MFS:

````
ipfs files write --create "/path_in_MFS" "/path_of_local_file"              
````

To check a directory/file status (returns the CID and other info):

````
ipfs files stat "/path_in_MFS"              
````

To view content of a directory:

````
ipfs files ls "/path_in_MFS"       
````

To copy a file/directory:

````
ipfs files cp "/origin_path_in_MFS" "/destination_path_in_MFS"        
````

To move a file/directory:

````
ipfs files mv "/origin_path_in_MFS" "/destination_path_in_MFS"        
````

To read the content of a file:
````
ipfs files read "/path_in_MFS"       
````

To remove a file/directory from MFS:
use -r to recusively removal

````
ipfs files rm "/path_in_MFS"   
ipfs files rm -r "/path_in_MFS"      
````

## IPNS Interplanetary name service

The InterPlanetary Name System (IPNS) allows to create an address that redirects to a configurable CID and can be updated. A name in IPNS is the hash of a public key. It is associated with a record containing information about the hash it links to that is signed by the corresponding private key. New records can be signed and published at any time.

To publish a new IPNS address that redirects to an IPFS CID:

````
ipfs name publish /ipfs/<CID to be published>
````

> Published to "IPNS address": /ipfs/"CID published" 

To check to what CID is redirecting a IPNS address:

````
ipfs name resolve <IPNS address (optional, if you don't specify it, it will check your main address)>
````
> /ipfs/"Redirecting CID"

To generate a secondary IPN address:

````
ipfs key gen <Custom key name>
````

> "New IPNS address" 

To publish a custom IPNS address that redirects to an IPFS CID:
````
ipfs name publish --key=<Custom key name> /ipfs/<CID to be published>
````
> Published to "Custom IPNS address": /ipfs/"CID published" 
