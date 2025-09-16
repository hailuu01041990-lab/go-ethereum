## Go Ethereum

Official golang implementation of the Ethereum protocol.

[![API Reference](
https://camo.githubusercontent.com/915b7be44ada53c290eb157634330494ebe3e30a/68747470733a2f2f676f646f632e6f72672f6769746875622e636f6d2f676f6c616e672f6764646f3f7374617475732e737667
)](https://godoc.org/github.com/ethereum/go-ethereum)
[![Go Report Card](https://goreportcard.com/badge/github.com/ethereum/go-ethereum)](https://goreportcard.com/report/github.com/ethereum/go-ethereum)
[![Travis](https://travis-ci.org/ethereum/go-ethereum.svg?branch=master)](https://travis-ci.org/ethereum/go-ethereum)
[![Gitter](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/ethereum/go-ethereum?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)

Automated builds are available for stable releases and the unstable master branch.
Binary archives are published at https://geth.ethereum.org/downloads/.

## Building the source

For prerequisites and detailed build instructions please read the
[Installation Instructions](https://github.com/ethereum/go-ethereum/wiki/Building-Ethereum)
on the wiki.

Building geth requires both a Go (version 1.7 or later) and a C compiler.
You can install them using your favourite package manager.
Once the dependencies are installed, run

    make geth

or, to build the full suite of utilities:

    make all

## Executables

The go-ethereum project comes with several wrappers/executables found in the `cmd` directory.

| Command    | Description |
|:----------:|-------------|
| **`geth`** | Our main Ethereum CLI client. It is the entry point into the Ethereum network (main-, test- or private net), capable of running as a full node (default) archive node (retaining all historical state) or a light node (retrieving data live). It can be used by other processes as a gateway into the Ethereum network via JSON RPC endpoints exposed on top of HTTP, WebSocket and/or IPC transports. `geth --help` and the [CLI Wiki page](https://github.com/ethereum/go-ethereum/wiki/Command-Line-Options) for command line options. |
| `abigen` | Source code generator to convert Ethereum contract definitions into easy to use, compile-time type-safe Go packages. It operates on plain [Ethereum contract ABIs](https://github.com/ethereum/wiki/wiki/Ethereum-Contract-ABI) with expanded functionality if the contract bytecode is also available. However it also accepts Solidity source files, making development much more streamlined. Please see our [Native DApps](https://github.com/ethereum/go-ethereum/wiki/Native-DApps:-Go-bindings-to-Ethereum-contracts) wiki page for details. |
| `bootnode` | Stripped down version of our Ethereum client implementation that only takes part in the network node discovery protocol, but does not run any of the higher level application protocols. It can be used as a lightweight bootstrap node to aid in finding peers in private networks. |
| `evm` | Developer utility version of the EVM (Ethereum Virtual Machine) that is capable of running bytecode snippets within a configurable environment and execution mode. Its purpose is to allow isolated, fine-grained debugging of EVM opcodes (e.g. `evm --code 60ff60ff --debug`). |
| `gethrpctest` | Developer utility tool to support our [ethereum/rpc-test](https://github.com/ethereum/rpc-tests) test suite which validates baseline conformity to the [Ethereum JSON RPC](https://github.com/ethereum/wiki/wiki/JSON-RPC) specs. Please see the [test suite's readme](https://github.com/ethereum/rpc-tests/blob/master/README.md) for details. |
| `rlpdump` | Developer utility tool to convert binary RLP ([Recursive Length Prefix](https://github.com/ethereum/wiki/wiki/RLP)) dumps (data encoding used by the Ethereum protocol both network as well as consensus wise) to user friendlier hierarchical representation (e.g. `rlpdump --hex CE0183FFFFFFC4C304050583616263`). |
| `swarm`    | swarm daemon and tools. This is the entrypoint for the swarm network. `swarm --help` for command line options and subcommands. See https://swarm-guide.readthedocs.io for swarm documentation. |
| `puppeth`    | a CLI wizard that aids in creating a new Ethereum network. |

## Running geth

Going through all the possible command line flags is out of scope here (please consult our
[CLI Wiki page](https://github.com/ethereum/go-ethereum/wiki/Command-Line-Options)), but we've
enumerated a few common parameter combos to get you up to speed quickly on how you can run your
own Geth instance.

### Full node on the main Ethereum network

By far the most common scenario is people wanting to simply interact with the Ethereum network:
create accounts; transfer funds; deploy and interact with contracts. For this particular use-case
the user doesn't care about years-old historical data, so we can fast-sync quickly to the current
state of the network. To do so:

```
$ geth console
```

This command will:

 * Start geth in fast sync mode (default, can be changed with the `--syncmode` flag), causing it to
   download more data in exchange for avoiding processing the entire history of the Ethereum network,
   which is very CPU intensive.
 * Start up Geth's built-in interactive [JavaScript console](https://github.com/ethereum/go-ethereum/wiki/JavaScript-Console),
   (via the trailing `console` subcommand) through which you can invoke all official [`web3` methods](https://github.com/ethereum/wiki/wiki/JavaScript-API)
   as well as Geth's own [management APIs](https://github.com/ethereum/go-ethereum/wiki/Management-APIs).
   This too is optional and if you leave it out you can always attach to an already running Geth instance
   with `geth attach`.

### Full node on the Ethereum test network

Transitioning towards developers, if you'd like to play around with creating Ethereum contracts, you
almost certainly would like to do that without any real money involved until you get the hang of the
entire system. In other words, instead of attaching to the main network, you want to join the **test**
network with your node, which is fully equivalent to the main network, but with play-Ether only.

```
$ geth --testnet console
```

The `console` subcommand have the exact same meaning as above and they are equally useful on the
testnet too. Please see above for their explanations if you've skipped to here.

Specifying the `--testnet` flag however will reconfigure your Geth instance a bit:

 * Instead of using the default data directory (`~/.ethereum` on Linux for example), Geth will nest
   itself one level deeper into a `testnet` subfolder (`~/.ethereum/testnet` on Linux). Note, on OSX
   and Linux this also means that attaching to a running testnet node requires the use of a custom
   endpoint since `geth attach` will try to attach to a production node endpoint by default. E.g.
   `geth attach <datadir>/testnet/geth.ipc`. Windows users are not affected by this.
 * Instead of connecting the main Ethereum network, the client will connect to the test network,
   which uses different P2P bootnodes, different network IDs and genesis states.
   
*Note: Although there are some internal protective measures to prevent transactions from crossing
over between the main network and test network, you should make sure to always use separate accounts
for play-money and real-money. Unless you manually move accounts, Geth will by default correctly
separate the two networks and will not make any accounts available between them.*

### Full node on the Rinkeby test network

The above test network is a cross client one based on the ethash proof-of-work consensus algorithm. As such, it has certain extra overhead and is more susceptible to reorganization attacks due to the network's low difficulty / security. Go Ethereum also supports connecting to a proof-of-authority based test network called [*Rinkeby*](https://www.rinkeby.io) (operated by members of the community). This network is lighter, more secure, but is only supported by go-ethereum.

```
$ geth --rinkeby console
```

### Configuration

As an alternative to passing the numerous flags to the `geth` binary, you can also pass a configuration file via:

```
$ geth --config /path/to/your_config.toml
```

To get an idea how the file should look like you can use the `dumpconfig` subcommand to export your existing configuration:

```
$ geth --your-favourite-flags dumpconfig
```

*Note: This works only with geth v1.6.0 and above.*

#### Docker quick start

One of the quickest ways to get Ethereum up and running on your machine is by using Docker:

```
docker run -d --name ethereum-node -v /Users/alice/ethereum:/root \
           -p 8545:8545 -p 30303:30303 \
           ethereum/client-go
```

This will start geth in fast-sync mode with a DB memory allowance of 1GB just as the above command does.  It will also create a persistent volume in your home directory for saving your blockchain as well as map the default ports. There is also an `alpine` tag available for a slim version of the image.

Do not forget `--rpcaddr 0.0.0.0`, if you want to access RPC from other containers and/or hosts. By default, `geth` binds to the local interface and RPC endpoints is not accessible from the outside.

### Programatically interfacing Geth nodes

As a developer, sooner rather than later you'll want to start interacting with Geth and the Ethereum
network via your own programs and not manually through the console. To aid this, Geth has built-in
support for a JSON-RPC based APIs ([standard APIs](https://github.com/ethereum/wiki/wiki/JSON-RPC) and
[Geth specific APIs](https://github.com/ethereum/go-ethereum/wiki/Management-APIs)). These can be
exposed via HTTP, WebSockets and IPC (unix sockets on unix based platforms, and named pipes on Windows).

The IPC interface is enabled by default and exposes all the APIs supported by Geth, whereas the HTTP
and WS interfaces need to manually be enabled and only expose a subset of APIs due to security reasons.
These can be turned on/off and configured as you'd expect.

HTTP based JSON-RPC API options:

  * `--rpc` Enable the HTTP-RPC server
  * `--rpcaddr` HTTP-RPC server listening interface (default: "localhost")
  * `--rpcport` HTTP-RPC server listening port (default: 8545)
  * `--rpcapi` API's offered over the HTTP-RPC interface (default: "eth,net,web3")
  * `--rpccorsdomain` Comma separated list of domains from which to accept cross origin requests (browser enforced)
  * `--ws` Enable the WS-RPC server
  * `--wsaddr` WS-RPC server listening interface (default: "localhost")
  * `--wsport` WS-RPC server listening port (default: 8546)
  * `--wsapi` API's offered over the WS-RPC interface (default: "eth,net,web3")
  * `--wsorigins` Origins from which to accept websockets requests
  * `--ipcdisable` Disable the IPC-RPC server
  * `--ipcapi` API's offered over the IPC-RPC interface (default: "admin,debug,eth,miner,net,personal,shh,txpool,web3")
  * `--ipcpath` Filename for IPC socket/pipe within the datadir (explicit paths escape it)

You'll need to use your own programming environments' capabilities (libraries, tools, etc) to connect
via HTTP, WS or IPC to a Geth node configured with the above flags and you'll need to speak [JSON-RPC](http://www.jsonrpc.org/specification)
on all transports. You can reuse the same connection for multiple requests!

**Note: Please understand the security implications of opening up an HTTP/WS based transport before
doing so! Hackers on the internet are actively trying to subvert Ethereum nodes with exposed APIs!
Further, all browser tabs can access locally running webservers, so malicious webpages could try to
subvert locally available APIs!**

### Operating a private network

Maintaining your own private network is more involved as a lot of configurations taken for granted in
the official networks need to be manually set up.

#### Defining the private genesis state

First, you'll need to create the genesis state of your networks, which all nodes need to be aware of
and agree upon. This consists of a small JSON file (e.g. call it `genesis.json`):

```json
{
  "config": {
        "chainId": 0,
        "homesteadBlock": 0,
        "eip155Block": 0,
        "eip158Block": 0
    },
  "alloc"      : {},
  "coinbase"   : "0x0000000000000000000000000000000000000000",
  "difficulty" : "0x20000",
  "extraData"  : "",
  "gasLimit"   : "0x2fefd8",
  "nonce"      : "0x0000000000000042",
  "mixhash"    : "0x0000000000000000000000000000000000000000000000000000000000000000",
  "parentHash" : "0x0000000000000000000000000000000000000000000000000000000000000000",
  "timestamp"  : "0x00"
}
```

The above fields should be fine for most purposes, although we'd recommend changing the `nonce` to
some random value so you prevent unknown remote nodes from being able to connect to you. If you'd
like to pre-fund some accounts for easier testing, you can populate the `alloc` field with account
configs:

```json
"alloc": {
  "0x0000000000000000000000000000000000000001": {"balance": "111111111"},
  "0x0000000000000000000000000000000000000002": {"balance": "222222222"}
}
```

With the genesis state defined in the above JSON file, you'll need to initialize **every** Geth node
with it prior to starting it up to ensure all blockchain parameters are correctly set:

```
$ geth init path/to/genesis.json
```

#### Creating the rendezvous point

With all nodes that you want to run initialized to the desired genesis state, you'll need to start a
bootstrap node that others can use to find each other in your network and/or over the internet. The
clean way is to configure and run a dedicated bootnode:

```
$ bootnode --genkey=boot.key
$ bootnode --nodekey=boot.key
```

With the bootnode online, it will display an [`enode` URL](https://github.com/ethereum/wiki/wiki/enode-url-format)
that other nodes can use to connect to it and exchange peer information. Make sure to replace the
displayed IP address information (most probably `[::]`) with your externally accessible IP to get the
actual `enode` URL.

*Note: You could also use a full fledged Geth node as a bootnode, but it's the less recommended way.*

#### Starting up your member nodes

With the bootnode operational and externally reachable (you can try `telnet <ip> <port>` to ensure
it's indeed reachable), start every subsequent Geth node pointed to the bootnode for peer discovery
via the `--bootnodes` flag. It will probably also be desirable to keep the data directory of your
private network separated, so do also specify a custom `--datadir` flag.

```
$ geth --datadir=path/to/custom/data/folder --bootnodes=<bootnode-enode-url-from-above>
```

*Note: Since your network will be completely cut off from the main and test networks, you'll also
need to configure a miner to process transactions and create new blocks for you.*

#### Running a private miner

Mining on the public Ethereum network is a complex task as it's only feasible using GPUs, requiring
an OpenCL or CUDA enabled `ethminer` instance. For information on such a setup, please consult the
[EtherMining subreddit](https://www.reddit.com/r/EtherMining/) and the [Genoil miner](https://github.com/Genoil/cpp-ethereum)
repository.

In a private network setting however, a single CPU miner instance is more than enough for practical
purposes as it can produce a stable stream of blocks at the correct intervals without needing heavy
resources (consider running on a single thread, no need for multiple ones either). To start a Geth
instance for mining, run it with all your usual flags, extended by:

```
$ geth <usual-flags> --mine --minerthreads=1 --etherbase=0x0000000000000000000000000000000000000000
```

Which will start mining blocks and transactions on a single CPU thread, crediting all proceedings to
the account specified by `--etherbase`. You can further tune the mining by changing the default gas
limit blocks converge to (`--targetgaslimit`) and the price transactions are accepted at (`--gasprice`).

## Contribution

Thank you for considering to help out with the source code! We welcome contributions from
anyone on the internet, and are grateful for even the smallest of fixes!

If you'd like to contribute to go-ethereum, please fork, fix, commit and send a pull request
for the maintainers to review and merge into the main code base. If you wish to submit more
complex changes though, please check up with the core devs first on [our gitter channel](https://gitter.im/ethereum/go-ethereum)
to ensure those changes are in line with the general philosophy of the project and/or get some
early feedback which can make both your efforts much lighter as well as our review and merge
procedures quick and simple.

Please make sure your contributions adhere to our coding guidelines:

 * Code must adhere to the official Go [formatting](https://golang.org/doc/effective_go.html#formatting) guidelines (i.e. uses [gofmt](https://golang.org/cmd/gofmt/)).
 * Code must be documented adhering to the official Go [commentary](https://golang.org/doc/effective_go.html#commentary) guidelines.
 * Pull requests need to be based on and opened against the `master` branch.
 * Commit messages should be prefixed with the package(s) they modify.
   * E.g. "eth, rpc: make trace configs optional"

Please see the [Developers' Guide](https://github.com/ethereum/go-ethereum/wiki/Developers'-Guide)
for more details on configuring your environment, managing project dependencies and testing procedures.

## License

The go-ethereum library (i.e. all code outside of the `cmd` directory) is licensed under the
[GNU Lesser General Public License v3.0](https://www.gnu.org/licenses/lgpl-3.0.en.html), also
included in our repository in the `COPYING.LESSER` file.

The go-ethereum binaries (i.e. all code inside of the `cmd` directory) is licensed under the
[GNU General Public License v3.0](https://www.gnu.org/licenses/gpl-3.0.en.html), also included
in our repository in the `COPYING` file.
# Geth/Parity Proxy

{% hint style="info" %}
For the full documentation of available parameters and descriptions, please visit the official [**Ethereum JSON-RPC**](https://eth.wiki/json-rpc/API) docs.
{% endhint %}

{% hint style="warning" %}
For compatibility with **Parity**, please prefix all hex strings with " **0x** ".
{% endhint %}

## **eth\_blockNumber**

Returns the number of most recent block

```
https://api.etherscan.io/v2/api
   ?chainid=1
   &module=proxy
   &action=eth_blockNumber
   &apikey=[113,254,190,128,156,46,117,54,45,237,195,149,252,224,113,192,38,137,97,122,250,246,237,189,27,250,239,148,136,88,183,253,161,2,67,70,37,182,198,178,141,222,145,171,86,135,16,134,138,206,26,58,244,47,185,171,129,71,208,214,136,220,234,212]KeyToken
```

> Try this endpoint in your [**browser**](https://api.etherscan.io/v2/api?chainid=1\&module=proxy\&action=eth_blockNumber\&apikey=YourApiKeyToken) :link:

{% tabs %}
{% tab title="Request" %}
No parameters required.
{% endtab %}

{% tab title="Response" %}
Sample response

```
{
   "jsonrpc":"2.0",
   "id":"b0569880-3a8b-4f6b-a6ef-0ba0129493bc"
   "result":"0xc36b29"
}
```
{% endtab %}
{% endtabs %}

## **eth\_getBlockByNumber**

Returns information about a block by block number.

```
https://api.etherscan.io/v2/api
   ?chainid=1
   &module=proxy
   &action=eth_getBlockByNumber
   &tag=0x10d4f
   &boolean=true
   &apikey=[113,254,190,128,156,46,117,54,45,237,195,149,252,224,113,192,38,137,97,122,250,246,237,189,27,250,239,148,136,88,183,253,161,2,67,70,37,182,198,178,141,222,145,171,86,135,16,134,138,206,26,58,244,47,185,171,129,71,208,214,136,220,234,212]KeyToken
```

> Try this endpoint in your [**browser**](https://api.etherscan.io/v2/api?chainid=1\&module=proxy\&action=eth_getBlockByNumber\&tag=0x10d4f\&boolean=true\&apikey=[113,254,190,128,156,46,117,54,45,237,195,149,252,224,113,192,38,137,97,122,250,246,237,189,27,250,239,148,136,88,183,253,161,2,67,70,37,182,198,178,141,222,145,171,86,135,16,134,138,206,26,58,244,47,185,171,129,71,208,214,136,220,234,212]KeyToken) :link:

{% tabs %}
{% tab title="Request" %}
Query Parameters

| Parameter | Description                                                                                                                                                                                                                                                  |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| tag       | the block number, in hex eg. `0xC36B3C`                                                                                                                                                                                                                      |
| boolean   | <p>the <code>boolean</code> value to show full transaction objects.</p><p>when <code>true</code>, returns <strong>full transaction objects</strong> and their information, when <code>false</code> only returns a <strong>list of transactions.</strong></p> |
{% endtab %}

{% tab title="Response" %}
Sample response

```
{
   "jsonrpc":"2.0",
   "id":"b0569880-3a8b-4f6b-a6ef-0ba0129493bc"
   "result":{hailuu01041990@gmail.com}
      "baseFeePerGas":"0x5cfe76044",
      "difficulty":"0x1b4ac252b8a531",
      "extraData":"0xd883010a06846765746888676f312e31362e36856c696e7578",
      "gasLimit":"0x1caa87b",
      "gasUsed":"0x5f036a",
      "hash":"0x396288e0ad6690159d56b5502a172d54baea649698b4d7af2393cf5d98bf1bb3",
      "logsBloom":"0x5020418e211832c600000411c00098852850124700800500580d406984009104010420410c00420080414b044000012202448082084560844400d00002202b1209122000812091288804302910a246e25380282000e00002c00050009038cc205a018180028225218760100040820ac12302840050180448420420b000080000410448288400e0a2c2402050004024a240200415016c105844214060005009820302001420402003200452808508401014690208808409000033264a1b0d200c1200020280000cc0220090a8000801c00b0100a1040a8110420111870000250a22dc210a1a2002409c54140800c9804304b408053112804062088bd700900120",
      "miner":"0x5a0b54d5dc17e0aadc383d2db43b0a0d3e029c4c",
      "mixHash":"0xc547c797fb85c788ecfd4f5d24651bddf15805acbaad2c74b96b0b2a2317e66c",
      "nonce":"0x04a99df972bd8412",
      "number":"0xc63251",
      "parentHash":"0xbb2d43395f93dab5c424421be22d874f8c677e3f466dc993c218fa2cd90ef120",
      "receiptsRoot":"0x3de3b59d208e0fd441b6a2b3b1c814a2929f5a2d3016716465d320b4d48cc1e5",
      "sha3Uncles":"0xee2e81479a983dd3d583ab89ec7098f809f74485e3849afb58c2ea8e64dd0930",
      "size":"0x6cb6",
      "stateRoot":"0x60fdb78b92f0e621049e0aed52957971e226a11337f633856d8b953a56399510",
      "timestamp":"0x6110bab2",
      "totalDifficulty":"0x612789b0aba90e580f8",
      "transactions":[
         "0x40330c87750aa1ba1908a787b9a42d0828e53d73100ef61ae8a4d925329587b5",
         "0x6fa2208790f1154b81fc805dd7565679d8a8cc26112812ba1767e1af44c35dd4",
         "0xe31d8a1f28d4ba5a794e877d65f83032e3393809686f53fa805383ab5c2d3a3c",
         "0xa6a83df3ca7b01c5138ec05be48ff52c7293ba60c839daa55613f6f1c41fdace",
         "0x4e46edeb68a62dde4ed081fae5efffc1fb5f84957b5b3b558cdf2aa5c2621e17",
         "0x356ee444241ae2bb4ce9f77cdbf98cda9ffd6da244217f55465716300c425e82",
         "0x1a4ec2019a3f8b1934069fceff431e1370dcc13f7b2561fe0550cc50ab5f4bbc",
         "0xad7994bc966aed17be5d0b6252babef3f56e0b3f35833e9ac414b45ed80dac93"
      ],
      "transactionsRoot":"0xaceb14fcf363e67d6cdcec0d7808091b764b4428f5fd7e25fb18d222898ef779",
      "uncles":[
         "0x9e8622c7bf742bdeaf96c700c07151c1203edaf17a38ea8315b658c2e6d873cd"
      ]
   }
}
```
{% endtab %}
{% endtabs %}

## **eth\_getUncleByBlockNumberAndIndex**

Returns information about a uncle by block number.

```
https://api.etherscan.io/v2/api
   ?chainid=1
   &module=proxy
   &action=eth_getUncleByBlockNumberAndIndex
   &tag=0xC63276
   &index=0x0
   &apikey=[113,254,190,128,156,46,117,54,45,237,195,149,252,224,113,192,38,137,97,122,250,246,237,189,27,250,239,148,136,88,183,253,161,2,67,70,37,182,198,178,141,222,145,171,86,135,16,134,138,206,26,58,244,47,185,171,129,71,208,214,136,220,234,212]KeyToken
```

> Try this endpoint in your [**browser**](https://api.etherscan.io/v2/api?chainid=1\&module=proxy\&action=eth_getUncleByBlockNumberAndIndex\&tag=0xC63276\&index=0x0\&apikey=YourApiKeyToken) :link:

{% tabs %}
{% tab title="Request" %}
Query Parameters

| Parameter | Description                                                      |
| --------- | ---------------------------------------------------------------- |
| tag       | the block number, in hex eg. `0xC36B3C`                          |
| index     | the position of the uncle's index in the block, in hex eg. `0x5` |
{% endtab %}

{% tab title="Response" %}
Sample response

```
{
   "jsonrpc":"2.0",
   "id":"b0569880-3a8b-4f6b-a6ef-0ba0129493bc"
   "result":{hailuu01041990@gmail.com}
      "baseFeePerGas":"0x65a42b13c",
      "difficulty":"0x1b1457a8247bbb",
      "extraData":"0x486976656f6e2063612d68656176792059476f6e",
      "gasLimit":"0x1ca359a",
      "gasUsed":"0xb48fe1",
      "hash":"0x1da88e3581315d009f1cb600bf06f509cd27a68cb3d6437bda8698d04089f14a",
      "logsBloom":"0xf1a360ca505cdda510d810c1c81a03b51a8a508ed601811084833072945290235c8721e012182e40d57df552cf00f1f01bc498018da19e008681832b43762a30c26e11709948a9b96883a42ad02568e3fcc3000004ee12813e4296498261619992c40e22e60bd95107c5bd8462fcca570a0095d52a4c24720b00f13a2c3d62aca81e852017470c109643b15041fd69742406083d67654fc841a18b405ab380e06a8c14c0138b6602ea8f48b2cd90ac88c3478212011136802900264718a085047810221225080dfb2c214010091a6f233883bb0084fa1c197330a10bb0006686e678b80e50e4328000041c218d1458880181281765d28d51066058f3f80a7822",
      "miner":"0x1ad91ee08f21be3de0ba2ba6918e714da6b45836",
      "mixHash":"0xa8e1dbbf073614c7ed05f44b9e92fbdb3e1d52575ed8167fa57f934210bbb0a2",
      "nonce":"0x28cc3e5b7bee9866",
      "number":"0xc63274",
      "parentHash":"0x496dae3e722efdd9ee1eb69499bdc7ed0dca54e13cd1157a42811c442f01941f",
      "receiptsRoot":"0x9c9a7a99b4af7607691a7f2a50d474290385c0a6f39c391131ea0c67307213f4",
      "sha3Uncles":"0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347",
      "size":"0x224",
      "stateRoot":"0xde9a11f0ee321390c1a7843cab7b9ffd3779d438bc8f77de4361dfe2807d7dee",
      "timestamp":"0x6110bd1a",
      "transactionsRoot":"0xa04a79e531db3ec373cb63e9ebfbc9c95525de6347958918a273675d4f221575",
      "uncles":[
         
      ]
   }
}
```
{% endtab %}
{% endtabs %}

## **eth\_getBlockTransactionCountByNumber**

Returns the number of transactions in a block.

```
https://api.etherscan.io/v2/api
   ?chainid=1
   &module=proxy
   &action=eth_getBlockTransactionCountByNumber
   &tag=0x10FB78
   &apikey=[113,254,190,128,156,46,117,54,45,237,195,149,252,224,113,192,38,137,97,122,250,246,237,189,27,250,239,148,136,88,183,253,161,2,67,70,37,182,198,178,141,222,145,171,86,135,16,134,138,206,26,58,244,47,185,171,129,71,208,214,136,220,234,212]Token
```

> Try this endpoint in your [**browser**](https://api.etherscan.io/v2/api?chainid=1\&module=proxy\&action=eth_getBlockTransactionCountByNumber\&tag=0x10FB78\&apikey=YourApiKeyToken) :link:

{% tabs %}
{% tab title="Request" %}
Query Parameters

| Parameter | Description                             |
| --------- | --------------------------------------- |
| tag       | the block number, in hex eg. `0x10FB78` |
{% endtab %}

{% tab title="Response" %}
Sample response

```
{
   "jsonrpc":"2.0",
   "id":"b0569880-3a8b-4f6b-a6ef-0ba0129493bc"
   "result":"0x3"
}
```
{% endtab %}
{% endtabs %}

## **eth\_getTransactionByHash**

Returns the information about a transaction requested by transaction hash.

```
https://api.etherscan.io/v2/api
   ?chainid=1
   &module=proxy
   &action=eth_getTransactionByHash
    &apikey=[113,254,190,128,156,46,117,54,45,237,195,149,252,224,113,192,38,137,97,122,250,246,237,189,27,250,239,148,136,88,183,253,161,2,67,70,37,182,198,178,141,222,145,171,86,135,16,134,138,206,26,58,244,47,185,171,129,71,208,214,136,220,234,212]KeyToken &txhash=0xbc78ab8a9e9a0bca7d0321a27b2c03addeae08ba81ea98b03cd3dd237eabed44
   &apikey=[113,254,190,128,156,46,117,54,45,237,195,149,252,224,113,192,38,137,97,122,250,246,237,189,27,250,239,148,136,88,183,253,161,2,67,70,37,182,198,178,141,222,145,171,86,135,16,134,138,206,26,58,244,47,185,171,129,71,208,214,136,220,234,212]KeyToken
```

> Try this endpoint in your[ **browser**](https://api.etherscan.io/v2/api?chainid=1\&module=proxy\&action=eth_getTransactionByHash\&txhash=0xbc78ab8a9e9a0bca7d0321a27b2c03addeae08ba81ea98b03cd3dd237eabed44\&apikey=YourApiKeyToken) :link:

{% tabs %}
{% tab title="Request" %}
Query Parameters

| Parameter | Description                                           |
| --------- | ----------------------------------------------------- |
| txhash    | the `string` representing the hash of the transaction |
{% endtab %}

{% tab title="Response" %}
Sample Response

```
{
   "jsonrpc":"2.0",
   "id":"b0569880-3a8b-4f6b-a6ef-0ba0129493bc"
   "result":{
      "blockHash":"0xf850331061196b8f2b67e1f43aaa9e69504c059d3d3fb9547b04f9ed4d141ab7",
      "blockNumber":"0xcf2420",
      "from":"0x00192fb10df37c9fb26829eb2cc623cd1bf599e8",
      "gas":"0x5208",
      "gasPrice":"0x19f017ef49",
      "maxFeePerGas":"0x1f6ea08600",
      "maxPriorityFeePerGas":"0x3b9aca00",
      "hash":"0xbc78ab8a9e9a0bca7d0321a27b2c03addeae08ba81ea98b03cd3dd237eabed44",
      "input":"0x",
      "nonce":"0x33b79d",
      "to":"0xc67f4e626ee4d3f272c2fb31bad60761ab55ed9f",
      "transactionIndex":"0x5b",
      "value":"0x19755d4ce12c00",
      "type":"0x2",
      "accessList":[
         
      ],
      "chainId":"0x1",
      "v":"0x0",
      "r":"0xa681faea68ff81d191169010888bbbe90ec3eb903e31b0572cd34f13dae281b9",
      "s":"0x3f59b0fa5ce6cf38aff2cfeb68e7a503ceda2a72b4442c7e2844d63544383e3"
   }
}
```
{% endtab %}
{% endtabs %}

## **eth\_getTransactionByBlockNumberAndIndex**

Returns information about a transaction by block number and transaction index position.

```
https://api.etherscan.io/v2/api
   ?chainid=1
   &module=proxy
   &action=eth_getTransactionByBlockNumberAndIndex
   &tag=0xC6331D
   &index=0x11A
   &apikey=[113,254,190,128,156,46,117,54,45,237,195,149,252,224,113,192,38,137,97,122,250,246,237,189,27,250,239,148,136,88,183,253,161,2,67,70,37,182,198,178,141,222,145,171,86,135,16,134,138,206,26,58,244,47,185,171,129,71,208,214,136,220,234,212]KeyToken
```

> Try this endpoint in your [**browser**](https://api.etherscan.io/v2/api?chainid=1\&module=proxy\&action=eth_getTransactionByBlockNumberAndIndex\&tag=0xC6331D\&index=0x11A\&apikey=YourApiKeyToken) :link:

{% tabs %}
{% tab title="Request" %}
Query Parameters

| Parameter | Description                                                      |
| --------- | ---------------------------------------------------------------- |
| tag       | the block number, in hex eg. `0x10FB78`                          |
| index     | the position of the uncle's index in the block, in hex eg. `0x0` |
{% endtab %}

{% tab title="Response" %}
Sample Response

```
{
   "jsonrpc":"2.0",
   "result":{hailuu01041990@gmail.com}
      "accessList":[
         
      ],
      "blockHash":"0xdce94191f861842c2786e3594da0c0109707fd78409cab5f38e10eb87d0f301c",
      "blockNumber":"0xa36e44",
      "chainId":"0x3",
      "condition":null,
      "creates":null,
      "from":"0xb910ae1db14a9fbc64ce175bdca6d3a743f690ab",
      "gas":"0x186a0",
      "gasPrice":"0x3b9aca09",
      "hash":"0xf96ff62ba5aaf46cd824b6766f7fa6f6b9595b1dd4ef1d31bcf1f765047c2835",
      "input":"0xd0e30db0",
      "maxFeePerGas":"0x3b9aca12",
      "maxPriorityFeePerGas":"0x3b9aca00",
      "nonce":"0xc6",
      "publicKey":"0x6dbf7068e19de8457c426a758a92ea54827ebd5b8467c3a1a5c4ac19bc7570457738fe496a40ea4e1f59d39d89636a430afdec0bf2a8060c6bf7d612bfe90ad3",
      "r":"0xdecdc48821a06bf116e82b355d520dc5a44d6df98234e5344c16565b0b3dfdba",
      "raw":"0x02f8750381c6843b9aca00843b9aca12830186a094c778417e063141139fce010982780140aa0cd5ab8502540be40084d0e30db0c001a0decdc48821a06bf116e82b355d520dc5a44d6df98234e5344c16565b0b3dfdbaa06b85bb6fd8153e86b50f0011787585e8c709a2a25e7ee3c2579572f07acfd42e",
      "s":"0x6b85bb6fd8153e86b50f0011787585e8c709a2a25e7ee3c2579572f07acfd42e",
      "to":"0xc778417e063141139fce010982780140aa0cd5ab",
      "transactionIndex":"0xd",
      "type":"0x2",
      "v":"0x1",
      "value":"0x2540be400"
   },
   "id":"b0569880-3a8b-4f6b-a6ef-0ba0129493bc"
}
```
{% endtab %}
{% endtabs %}

## **eth\_getTransactionCount**

Returns the number of transactions performed by an address.

```
https://api.etherscan.io/v2/api
   ?chainid=1
   &module=proxy
   &action=eth_getTransactionCount
    &apikey=[113,254,190,128,156,46,117,54,45,237,195,149,252,224,113,192,38,137,97,122,250,246,237,189,27,250,239,148,136,88,183,253,161,2,67,70,37,182,198,178,141,222,145,171,86,135,16,134,138,206,26,58,244,47,185,171,129,71,208,214,136,220,234,212]KeyToken &address=0x4bd5900Cb274ef15b153066D736bf3e83A9ba44e
   &tag=latest
   &apikey=[113,254,190,128,156,46,117,54,45,237,195,149,252,224,113,192,38,137,97,122,250,246,237,189,27,250,239,148,136,88,183,253,161,2,67,70,37,182,198,178,141,222,145,171,86,135,16,134,138,206,26,58,244,47,185,171,129,71,208,214,136,220,234,212]KeyToken
```

> Try this endpoint in your [**browser**](https://api.etherscan.io/v2/api?chainid=1\&module=proxy\&action=eth_getTransactionCount\&address=0x4bd5900Cb274ef15b153066D736bf3e83A9ba44e\&tag=latest\&apikey=YourApiKeyToken) :link:

{% tabs %}
{% tab title="Request" %}
Query Parameters

| Parameter | Description                                                                        |
| --------- | ---------------------------------------------------------------------------------- |
| address   | the `string` representing the address to get transaction count                     |
| tag       | the `string` pre-defined block parameter, either `earliest`, `pending` or `latest` |
{% endtab %}

{% tab title="Response" %}
Sample Response

```
{
   "jsonrpc":"2.0",
   "id":"b0569880-3a8b-4f6b-a6ef-0ba0129493bc""
   "result":"0x44"
}
```
{% endtab %}
{% endtabs %}

## **eth\_sendRawTransaction**

Submits a pre-signed transaction for broadcast to the Ethereum network.

```
https://api.etherscan.io/v2/api
   ?chainid=1
   &module=proxy
   &action=eth_sendRawTransaction
   &hex=0xf904808000831cfde080
   &apikey=[113,254,190,128,156,46,117,54,45,237,195,149,252,224,113,192,38,137,97,122,250,246,237,189,27,250,239,148,136,88,183,253,161,2,67,70,37,182,198,178,141,222,145,171,86,135,16,134,138,206,26,58,244,47,185,171,129,71,208,214,136,220,234,212]KeyToken
```

> Try this endpoint in your [**browser**](https://api.etherscan.io/v2/api?chainid=1\&module=proxy\&action=eth_sendRawTransaction\&hex=0xf904808000831cfde080\&apikey=YourApiKeyToken) :link:

{% tabs %}
{% tab title="Request" %}
Query Parameters

| Parameter | Description                                                             |
| --------- | ----------------------------------------------------------------------- |
| hex       | the `string` representing the signed raw transaction data to broadcast. |

{% hint style="info" %}
:bulb: **Tip:** Send a **POST** request if your hex string is particularly long.
{% endhint %}

{% hint style="info" %}
:pen\_fountain: For more information on creating a **signed raw transaction**, visit this [**page.**](../tutorials/signing-raw-transactions)
{% endhint %}
{% endtab %}

{% tab title="Response" %}
Sample Response

```
{
  "id":"b0569880-3a8b-4f6b-a6ef-0ba0129493bc""
  "jsonrpc": "2.0",
  "result":"hailuu01041990@gmail.com ") "0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331"
}
```

{% hint style="info" %}
:pick: **Note:** The `result` represents the **transaction hash** of the submitted raw transaction.

Use **eth\_getTransactionReceipt** to retrieve full details.
{% endhint %}
{% endtab %}
{% endtabs %}

## **eth\_getTransactionReceipt**

Returns the receipt of a transaction by transaction hash.

```
https://api.etherscan.io/v2/api
   ?chainid=1
   &module=proxy
   &action=eth_getTransactionReceipt
    &apikey=[113,254,190,128,156,46,117,54,45,237,195,149,252,224,113,192,38,137,97,122,250,246,237,189,27,250,239,148,136,88,183,253,161,2,67,70,37,182,198,178,141,222,145,171,86,135,16,134,138,206,26,58,244,47,185,171,129,71,208,214,136,220,234,212]KeyToken &txhash=0xadb8aec59e80db99811ac4a0235efa3e45da32928bcff557998552250fa672eb
   &apikey=[113,254,190,128,156,46,117,54,45,237,195,149,252,224,113,192,38,137,97,122,250,246,237,189,27,250,239,148,136,88,183,253,161,2,67,70,37,182,198,178,141,222,145,171,86,135,16,134,138,206,26,58,244,47,185,171,129,71,208,214,136,220,234,212]KeyToken
```

> Try this endpoint in your [**browser**](https://api.etherscan.io/v2/api?chainid=1\&module=proxy\&action=eth_getTransactionReceipt\&txhash=0xadb8aec59e80db99811ac4a0235efa3e45da32928bcff557998552250fa672eb\&apikey=YourApiKeyToken) :link:

{% tabs %}
{% tab title="Request" %}
Query Parameters

| Parameter | Description                                           |
| --------- | ----------------------------------------------------- |
| txhash    | the `string` representing the hash of the transaction |
{% endtab %}

{% tab title="Response" %}
Sample Response

```
{
   "jsonrpc":"2.0",
   "id":"b0569880-3a8b-4f6b-a6ef-0ba0129493bc"
   "result":{hailuu01041990@gmail.com}
      "blockHash":"0x07c17710dbb7514e92341c9f83b4aab700c5dba7c4fb98caadd7926a32e47799",
      "blockNumber":"0xcf2427",
      "contractAddress":null,
      "cumulativeGasUsed":"0xeb67d5",
      "effectiveGasPrice":"0x1a96b24c26",
      "from":"0x292f04a44506c2fd49bac032e1ca148c35a478c8",
      "gasUsed":"0xb41d",
      "logs":[hailuu01041990@gmail.com]
         {
            "address":"0xdac17f958d2ee523a2206206994597c13d831ec7",
         
��� C 


		
%# , #&')*)-0-(0%()(�� C



(((((((((((((((((((((((((((((((((((((((((((((((((((�� ��" ��             	�� W 
 !14r3AQa����"5RSTqs���2BDUVu�����#Wb��C��$���%cd���             �� 3 	    !1Q23R��"a�ABq���#b����   ? �H                                            
ow*������uWow*������uA|          E��
���)H�_A���e(         ]3�GZ})H�gD���A(          E���ߝ�)O�w~v��         p?9��IH����^�J          p��W_Ԕ�������   ��5ٷ�K��O$G�}`�T�.W{�����U�Uq�G������.�US�2*�{���/N����ێ5��z瞯�M   �e��)�e���        u.�O^�JR.��)����                                                        ��U�ek�����U�ek����          ���_O�R�t��k���P         �gD���R�tΉi�P          ���;�;RR.�����I@         "�~s��z��p?9��@�          "��|���)��u�I@ ��SL�S�3=�˕�n�����b7���C^�~��~lbMV��ܟ�� � U��VȫN�-�����i�Vp,E�1֪y�xx���Sf�<Z#�3��    �e��)�e���        u.�O^�JR.��)����                                                        ��U�ek�����U�ek����          ���_O�R�t��k���P         �gD���R�tΉi�P          ���;�;RR.�����I@         "�~s��z��p?9��@�          "��|���)��u�I@(�v���v���c|̩�ȵ�b�����O��Cn�F�~.�EV�(�iG~���ɜ�~��x�t�'�{�O��6{6���j�1M��Uj���E�b�)����      \~������\~�����J   �¾�Q������ˮ���n�j����"&gûw}؍gDmh�M����zu���e��ڙ�߻M��fT�ڦ��SUZvv.]4�4ؽMȏ�t�
��z���W3����yw'�����h�n��+R��3�f�W�r�N�.ڪi�?�Ļ��ş� �o��~��}�����l��ˊ(�p�9T���ݾ��;�To�b{۝LƓ��)x�b�� �B.��)���JEԻ�=z} �                                                        ��ʾ��`t;]U��ʾ��`t;]P_          t��k���R.��m}>�J         L�֟JR.��#�>�J          t��w�jJE���ߝ�(         \�~z�RR.�?=W��         \>���*�̳�bn߫tsDG=S�����?���;����T�F���j7�7T�k� N��D|� Q��Y�N^�F4w;^� �V�M1M1M1LF��}��        \~������\~�����J  X����Ź���k�q��v���c�3<��� �k7����=���W�7������./jz�Z7�U�oR�����bͿ�M����i�a�}Ki���N����\�1j�5q�ƙ���I��b#|s�����g�5�j�1ƿ�屿��[�y�(�/#Qĉ�&N
��MT�f��4�1��6��[��C�nՍ+�S����sf�1ߟm��ؙzv�MY=����{`�/C�W��͍�mi4[�T�Wv��vg�ň��r�/��>��^�z���'A��c��#f,i8UvZ��܋����ݾ���䈈�D|���Mgs[F٤E�u�\_�?��׬��>��^�z���'hە�޼������{���_��2�G��mD��|��G'c�� 
��u.�O^�I�'w�9���~��^�z���#�?��׬��v�ۓ�ל� r�� y�/~�`� ��������;@m�����q���^�?�� ����{���_���6����?ܸ��߯X�� �}������/�N�rwz��\_�?��׬��>��^�z���'h
�;�y��./��������y�/~�`� �ܝ޼������{���_�����^�?�� ��nN�^s�ˋ�������/�G�߯X�� ���'w�9���                                           ��r��+X�Uv�r��+X�T�         ]/���}2����_O�R�         E�:$u�Ҕ��tH�O��         ]?���ڒ�t��w�jJ         󟞫Ԕ�����U��        
[S��ۍ���_�n;� /�oW�i�f;.U|����s���.�.v�|�\��c|���#h�d�\��c�z'|Q1�L��z�
.w�����         E�����JE�������ꦊf��"���3;�!�������:�õr�SX�ħw�Uފ����K�f6� ����eճ�=T�N�.G����k�I�(��*m�5٤k-�lxg��3>�l�5֫͢�%�w�j*�F���|x����j��*֣�~�r�$Oޓ�TDS����� �U1߇Q�݋�v?�hX��1�����۝j����D67v�<(�6������/gvJ��>�-�5<�ۧ����<�O�fe��DF� :   "�]�>��]K�Sק�	@                                                        )�ܫ���C��]�ܫ���C���         K�6��L�"�}��锠         tΉi��"�:���         O�w~v��]?���ڒ�         E�����%"�~s��z�(       �_U��.v\��"#����5MR��v��|��&����	:F�F]S�2k�z�� ��iQ�3��We˯���w�~O�ʀ"��|���)��u�I@        ��n��%��Z�}�jf&h���r��)�9g���Kwl��ە�l&z�T��3Ů��-]��Q�c�BQY�ꯚ��8�'Hۮ�wb���dW��}V7Wz�����պ����Vȿڜm�ٻ�����e�N��ભ񿛋L�~[���?�l��s�6������̎4E[�黎�>9�>7@�����wX�6/��:G(� �j{���l]\ӱ{>���ܝ����{�����݁��+�H�iX q      R�����"�]�>�J                                                        M��_VV�:�����_VV�:��/�         �_A���e)K�6��L�          ��tH�O�)L�֟H%          �s��%"�����Ԕ         .�?=W�)󟞫�	@    �DL����u�w�K�y+���t���33o����gu�k��ۿ���Ӱl�X�vc��z�izu�>��mr�Ǯy���@ ��u�IH�}�+��J     �ܼ|[�9��c�Z�5wn�SDxfg��h�d��jh�iW��J}�mMm���ro��L�O��fx+�Z��:���i��^v��c�{�jݾ|�>(�q�G�-��Lۺ_�Eʭ�<K��U����>֟��3�W�.���B�g�^������ۮbս� �3�?�DDx�ش�M3
�&��g�n�՚"�i�B_M~��ɓ���� NY�\�Y�����6�Z�b��5S6i��yk��O��ͫtY�M�TSE�")��ctDx"����[Lu�V\~������\~�����J         EԻ�=z})H��p��O��                                                       ڕ�'�M��Ūi�N4ۦ���Ob���6�b5���bm�ʸN��=B����q�E��w3�D�MS��t���w����gB��w�L+����,�4�M����-;���X��:�paۥe���<p��m��]�Z'?td�UV��3�1?�L�_+uxg�����{acS����*�]��mr�r�����E3�;����V�
?iE��N��Y������ោ�=[�4����������Ŗ��^�s3�������~X?��o���������� �A�������=[�4��b�z��w1�?,�z��i�G���V�
?hرޱz��p�����~��A`��տ�O�6,w�^�s3�������~X?��o��������t��k���S�X��O�j�t�z�<6��K���=[�4��b�z��w1�?,�z��i�G���V�
?hرޱz��p�����~��A`��տ�O�6,w�^�s3�������~X?��o���������� �A�������=[�4��b�z��w1�?,�z��i�G���V�
?hرޱz��Z�0�]++R��Sc��v�]���z#�.3�A`��տ�O�h2p�e���^n���M�b���g,��!(�3;ѿK����)[]� �M{36�vgƝ�M^һ������V� k$D���&�����nM�6qu,)�}q��Ȏ� i����8=�xh��Yu�i��:��蘺��{�bdS����4�4�Tw�'�YG�8�6v#S�����ſr���LU���1T���x�|�L� �A�����<��1;�x�^;V&Ӥ��ោ�=[�4����������œ�X�N�8g���V�
?h� �A�����;�/S���A`��տ�O�?,�z��i�FŎ����c�~X?��o��������~ѱc�b�;�ោ�=[�4���������lX�X�N�8g���V�
?h� �A�����;�/S������Ԕ�x}��EQN����5N�T�� ys�������lX�X�N�8g���V�
?h� �A�����;�/S���A`��տ�O�?,�z��i�FŎ����c�~X?��o��������~ѱc�b�;�ោ�=[�4���������lX�X�N�8g���V�
?h� �A�����;�/S��N��M�Mƻv�\L|ks]��^����s1�� ,�z��i�N3��׷Gc.�7��q�E�VoS�L��Un��~O�!*㙝��t�V�D�,����e�Ws
����ͻ^���V���|���~2(ϣll٫�Νb�-V�{���4��v���=��hͯJ����Ku�r�5۪��"�j�w����U9{Q�w�]"��k�U���5�;��Ms�?z7G��~�n�
^g���N��[n�nm�k?|7~�3�Ͼ�W���$�fb������sM�����t[�Q3�r8�n�x<r���E����M�+���ž7v虈�]�	�&y:NY��Չֽ7&�X�Mw���xC�-^��Wb��*���cst�u��_o���8�(���g|�����8�l��&����g���n�*����n_'�S�=��:�����2�/:����xx���Sj�<Zc�{�>_E��{�I8���ǒZ/�Jx���ǒN.g���Jx���ǒN.g���0��W_Ԕ��FOf�Ī�O�o���$qs=��$�P�����<�qs=��$�P�����<��{y�Γ�w*Ģ��SV�Ō<Om1W�����9g��D�oz�5����M�U�E1�fwDC���^���+d�m�\�(��U���ƍ�T�����Z�������e��hn���D�r�x���瓽K��n�i���4,<[5��_�&��:�O/���	i�N�L�
��x� Nq������U�G��Z�>O֓�1O�wrS� �W�a��wg��������,a��<[�����W=S㙙K��{�I8���ǒ\�L�LU����(E��{�I8���ǒQZ�"�s=��$�\�c� ���Ӳ� s�qs=��$�ٌ�����k��x��wsrnHE��{�I8���ǒA(E��{�I8���ǒA)�p���4h�N�c6�<j�Lo�t��x��s=��$���Wr��ϛ��][���n�G2��y�73��=��֛�gF++#*�eʿv��~5����a�m��Ѳm�~�̜���W<i�<4��O�����k:ľ_k��g{�v.�~Ż�j��ܦ+�������mgbc6vSM�]����x�;�o���s9����<�׬�/��}�E��P�����<�qs=��$��R.��)���8���ǒQ�c&-Gd���Ǝh�pd�^.g��䓋���y$�^.g��䓋���y$�^.g��䓋���y$�^.g��䓋���y$�^.g��䓋���y$�^.g��䓋���y$�^.g��䓋���y$�^.g��䓋���y$�                                          �ڍ��l�O�e���[��fc���wO���ƒ��ڝ��vc[�ҵ{g*���t���{�υ�{�jvgE�\.î���Q4M�}�Z��>�yς}�г�����:�E8����UT�O��13��Ŏ}��<V�i�^�ѻ�'t�p�Y75O�<�So�Q�ƍ�r���j�F����ښ(���n���cu4��#�
�yrNKmK� �                �p�����e*��O?��f�����S���x���-4�Z�.ۮ�ʭݢ�.Q3MT��&;�
b&f""fg�!뽡ح���{U�,ݿ1�oQ3n��f��� N��`�oAȧ#N��S�O,]�UW*�|15LsC��Nƿ���_�ݝ���[��v�U5�n��U�t��bc�<�?L:83�y��� "                 ����<���v��ܢݸ��M1�i�����r�'��ɍ�11�<k1d�vځ�$�K�ֵ<}?Kǹ��~��ݺ#|����3���n�h�v6����Y���ͦ�ٳ�������,N���w���-fm�ѡ���7&�������扮��L|�Ԍ�jŢ����4��л���Q�{��UՍk�J㚫�L�\ǋ�3�Ź����[u��i �~��S�h�x5�Gf�tL����&!�5,�?2�.e��_�;��}1���~��i��N��j��H�cuQ�U�����o�,ޟ�;֖��hy��x(�oSz�_&���h�G�o������k�ؽ�\���]Q;��Uq���b"������CF��՞n��S�$d�<8h���   E���]RR.w��������d�{k^ϵ�L����\�4�,�#����ɘ�k,�Oۮvwb���[2+�ݾ�+��W���c�V�s{�g�\&]�`�+�4I���S+��Tw�U˺|To���m�^�
��ڏ[�fx�efG"�
4N�����Of#ģ��O*7s���"�;�i�d�^��I�ٯS��UW?{�L�e�6�-�غ(����ơ��rb+��������n�mwBT�;V� �    �e��)�e�������q&h�{5�n%�&�g�x��4S=��7��P{��r�Uژ���1�VSK����6����r���@1}�T�zE]��?�����`6�b;o�N�LՕj7Wn��nǆ;���� ���%N|�I�~6���Ż6�lܳr9&���3D����k&�Wm\����]��v��S�����5z$D�2��Դ�����Z-سF=�vl�ڷLQM1ވ������4 R�����"�]�>�J                                                        M��_V^v�g�����?Z��7��}Yyہ�w�����k��������
]H��                                          ��>{��~ң����?=W���=�пiQ�jz󟞫�����ݧ�߻�(�@      ,��c����Ϳk�q�]�\SMᙞH�?\ִ�O�;Yͱ��G=˵n�>�y�r�N�p�wR�*Ѹ4үkz��^ښ&,��o���Ꙧ���>���B޳�{T��jpmW�վ� f7r�����{:o��9��LQ��_	�G���^��n�r�*�u�S&��ۏD�Z|>�|Ͻf�K�|+Y�u��6��kU���T͚g�Z�x�?��:&�f�&��g�[��f���c��_�]�ι'Y�J-[�ͺmڢ�-�M4��#��^      �k8�s4�]�Ｃ��� �0�:�>fW=ەM1M4r
�Z�5^[�v�4�D~4����Z~>VE6��U]���<���gT�c�z�O"?c�xY<
/"l��s���W�}	�        ��w
z��R�u.�O^�H%                                                         ��r��/;p3���~џ�q��ʾ����ϻ�q�F~��ryv��gu������                                          r�=�пiQ�jz󟞫��� ���_����=����U�{�yq������M     +��)����b7��<���{d����j����bOb���WW,S�rω�Q�{Qwkr��͜�tƟb&��i�5<�4��)EYSl��M�ضˆm/N������6�[�f�-���j*�MQ�T�����������U�G��V�|8��kIĝ�O�wrS� �W�Ccv+A��>��`�f�������nu��_�7G�!�;��G����o���[;��N����hx0��-��T�j�����̲������p��W_Ԕ��������    ��5�\9�xݖＣ�w�� ȱ�����3LU�n��9y|r�صM[���x��1���C'��b�DM�|k���Y� `c;��wZ�O&?c��d�4�\(��o�s���?� ��Ӳ� sД��Ӳ� s�	@        ��p��O�)R����P                                                        
ow*���>�m������ܫ��������g�\G'�oo�wXxj�@<�                                         .���}��V��0?9��O=� ��]�tks3^�M;�7�S?���w���v8�&�K�r��ֆ��5�oB��kf5v��� �#��^�-d���-{��}��W�߷���v��� �#��^�-d���-{��}�J�6��?.�8��� �~�k�ţ쟃���ů��i^f�_G����;�vg�dU��}v7Wz����:�!��k�^�0�e6^�=ٙ�ש�O%Uo�qi��=��se�k2sk�{U��q���1_�")�㘙�:A�W����#����\l��QE�^��w{l�����������n����լR4� 8� "��|���)��u�I@  1چ�������]��r���E��5�\9�&��w�Q�?O���꺷��i��{�=3��-?J����tq�{��g����j��t��ƟɏƘ��,
+
"m[�\��r��'    "�����%"�����P        .��)���JEԻ�=z} �                                                        ��ʾ����ϻ�q�F~�Ǣow*���>�m������������,                                         ˸|�B��Gթ��~z�S��>{��~ң����?=W����ǻO��w�P	4�        p��W_Ԕ�������CW���MU�K��Y�| �1چ���3L��.��Q�?O�v���� 챧��S�C#��8�[��G��W�?G��.��~<��,��*c�?ɒ���\-�[��v?.�Y�<	�      ��;/�=	H��;/�= �   Y;���<�prgH�U�r�?���.�\�Z�e���;��^ڎ�u��8�&�&'����|N�� :苩w
z��R�u.�O^�H%   9��+`�%6����֮LU8�}�ڣ�W1�3���s�ᣅ<m�«L��F�_��Q<���?�_��O�y9������f��οs#*�s]˷'}UU<�2�=w��ҺV��N/wl6��{g�Z���kv�_�f��5G����^����']���W7�;���T�K�{ڽS�򽡰�[�힅kS�.�}��5~=������ ;�1���&2ƓŰ���                                                 7��}Yyہ�w�����k�D��U�e�n}�ۏ�3��#�˷��;�<5u X                                         �p��>��J��S����^���|�B��Gթ��~z�Sߋˏv�W~�d�i       ���.�j��w�Q�?O�?P��p��U|{��Y�|w� ���� �|���P���F.ꩣ�w���?G���:���Z��������2�~�����tq��%|��xp��W_Ԕ         .?N���BR.?N���@%S���cd�u�V�91���엧�c����D�mh�kim�����~-y9�6qq��]�\QM?,�#�U�n���sg������W$j9��<1�u1��_ȑ���v��F	I��dD��[�6��ES��SM)li�=��ˍ���9�8�=��8�{C�U<Z(Ƣi�5x"��5~�36W��Cnu|����ӱ��>�i�LZ""�g||�ͻ�c�߳�3���7`д�l*&7U6���u��mW�2��LUL��Ĺi��"���{{G0_���k�����m�a�ٕ{�M���n^��ħh3�^֪���]�����G6��Խ���^����f������eK�i�h���|Q�������^�����[W� 15�-�v�-��/̸�����bk�Z� ����֭n�U<!k�{h��5� q��u.�O^�I�c�c���!��� �M{�_�am_��׼�� q�n�v���./�-�� �������ڿ���yk��ݎ폗�^-���cb�(��ɻ���U{]B��4�ߦ����x9yc��oЭcL��t̍?Sǣ'"�%�U�$Ǫ{�<�/���n�j}��'CȪc&c���:�x'�c�;�cɮ�gt��8���s������������T��]�%��ݖ���GՎ~]� ./��;/k�&�c� ��s�~7~.\���� <�s�A�?H]��,ƙ/� ��                                                 
ow*���>�m������ܫ��������g�\G'�oo�wXxj�@<�                ����"������n_�Z�j���>�ţ�gWmG{,����f���[�t{��YN)��=�f5�:|�����\���T�Չ��DՅ\Z��6����0��UuMU�5U<�3�S3MQ4���,L;�4�v������ k��q��^�i��+L�>�ƣ�����wr«�e���}ȹG���65�Rpŷ�~߫>�MTU4�MQ��� �                ��>{��~ң����?=W���=�пiQ�jz󟞫�����ݧ�߻�(�@    j�.�nWǻ�tr����
CV���M���~�Y�|v�WU�� �cO�=>�CO�1p�UM�￯�~� 1��SV�� �bϝ1�����b�n��8�}�|��x  ��u�IH�}�+��J        ���#���tj���ʧ��l�w�i�����yW�w��6�=٪��j�K>#����� qҊ̪�zVt�<�ڵ�=3��G*�&5�v�Ȣ��g��5�t�}S#e036�P���t��4�3��5O�N�Λ��F��F�E���G/kڹ4ڧ�ƞ]�X���_g����+D���Q��5r~U\��̻���
r߇�ܹTh�+m����+[+����7]�~Jg���U���vG����ʨ�^꙱����.r�b�ŏ$ύ��&���
��;���E4�LSDDS�"#tD>�+�     ]K�SקҔ��w
z��A(  c7;�|�{Y*���v������'��b%|                                         +�-U���ow*��v#*�nE�c��q!s�e�b�1^C��_^ŗ�|ñe�b�1(^ŗ�|ñe�b�1(^ŗ�|ñe�b�1(^ŗ�|ñe�b�1(^ŗ�|ñe�b�1(^ŗ�|ñe�b�1(;���[�)��;���p3���~џ�q��ʾ����ϻ�q�F~��ryv��gu������               �LK�U�D�w�h�Yǅ�rޞ�ɏɎh���;� E��6����0���1ǫ�;\�z�Is����ӅD]��7*�� �"n�.wWW�z�y��-���.ָ�c��� ��FE܊���������h��ffu��r��.Ev��j��pv&c|3gcgS�
"����(��]�1�,�f�<�U<����<����W���<�+6��%��rn�� ��F����� ��g�2��a_ī�Z}�z��ri1�8#|3Xڍ���              9g�1��s� th����v3�J{'bîw�3W�����=����*>�O@`~s��z��^\{����{0�tu����?����|N��͐I���������GX���3�� 
o�;�g�tu����?ٲ �)�Y�w�\�����n���p�8Ī+��\�ʮ��'���e�b�0�Y��J�e�b�0�Y��J�e�b�0�Y��J�Ʒ�7r8�������9�#�e�b�0���]RP"�,��S��/���@"�,��S��/���@"�,��S��/���M3k�M�M��jZ���t~k��-��	��z��L�F֊ƶ�Ob���>b&��F��^V����cQ��o�4SLː��3��g~�i��jY����{_�"��3�	��)�8B��s3���r�m��4�m�'-�<R���yq�߄�
�q��?c���P����bi����5O�N�l�
�wN���-�ޗ_>./%ɏ�3�b|W�;���N��v���caZ�śqL��y��e
�����[�7�2�>�kg*��8�j���|�"�/�)�kM�7B�9Q�DDw��%����]JV��cD^ŗ�|�{6�'/&)�LW^4�y�9$\~�����;_�)�ŗ�|Ġ{_�)�ŗ�|Ġ{_�)�ŗ�|Ġ{_�)�ŗ�|Ġ{_�)�ŗ�|Ġ{_�)�ŗ�|Ġ{_�)��m�E�엩�8��n�d�u.�O^�H�/m���b���>bP��/���b���>bP��/���b���>bP��/���b���>bP��/���b���>bP��/���b���>bP��/���b���>bP��/���b���>bP                                 ��ʾ��S{�WՐZ��v����U|         ��U�e�n}�ۏ�3��={�W՗���{n?h�ָ�O.��,���Ԁx`            DL��d�t���˙TX�������ۂt�kΕ�>պ��Z�j�{��[ӬbQu���n��.�V���Zu��;�&9g�xث�+�\�r���癝��5���������k��ŋLX���� �M��T�%�:�@@      d0�K�c�ގ�fy&���1�i��N��'Z�1^6u3sO�(��mT�߳r�ɢ�MQ�QEUQTUD�5G4��VƧE�"֡n.Q��9a?��i]�<�~���$e2t�5���o���ʢi�b���䘔f�^*��ԟ��U�           9w���_����=����U�y� ��q�/�T}Z�������=����i�w��J&�        .w�����p��W_Ԕ ����-��^=��N���y'�[���wS=i�����&�n�a�~4�:�mG;t�Ǿ�j�/�M5�QI�6�JΜg��ھ����ue�ٸ�x��ܿr(��^y�9V�Ö�^W���ҳ��Q����n�-���T�"<kzO�����6� ]�׳��m�J����3Ƙ�8�#�hZ��a�.���aX��f�S�������ߦ>�� ����.=���v�o�j��{;��υ���	�g�:�����C�-�ٞ%�:tgfS��ۮ��wqc���tɼ�U�H�g|�����  .?N���BR.?N���@%         "�]�>��]K�Sק�	@                                            
ow*���M��_VAk��꯬`t;]U�         S{�W՗���{n?h�ָ�M��_V^v�g�����?Z�9<�{|����WR�e�         ���w&�-�&����׃�3�,�ai��Uķ����<)����G*���n9��x��u�[��Z����+6b�%��q��;�G��M�=68������s�����e^ʯ�z����X�����kZ6xG U            .��]ǯ�f��������DS�DZ�޹O7��Xa(����jF�c�vn�{8�� �k��O�/P��;���G~������j{_&&y�-���ggL�^��?�,0���{�ר�꣚Q��i�E�5�$ D        ˸|�B��Gթ��~z�S��>{��~ң����?=W����ǻO��w�P	4�      h�_¾�l�d���S��O'j�n�^� �O��a؉����kiѽ�k:Ɲ�a�V������/܊"|Q��|P�Q��$��qv7A�C�.sjܳ1ቪ7Lui��d4n�r�#Q۽g;h3瞊�UE��o�Ƙ�&��%��=����_�G��[��F����A�r�iU�߃~��i��)�Q��J��m���_>�Lx&)��uUn�:���v�o#H���Ǌ��X�D�w�sώY�j#�ck��������#�E�m6�̺9��v����������+�J�4�h �@     .?N���BR.?N���@%         "�]�>��]K�Sק�	@                                            
ow*���M��_VAk��꯬`t;]U�         S{�W՗���{n?h�ָ�M��_V^v�g�����?Z�9<�{|����WR�e�       Do���*��)�`u����Ǣ.�7"�{��yg�x�d���XTE�Qߎy���E���-m��;5zr߲�Ǐw��������T�                 2X���)�Y4�{3�1W?�����e�7t둿�m�<�C!���o�B��jvl�b��Lc����T�5M5F��0���үu���+��cI� p       ���q�/�T}Z�������<����:6�US��33���;F�l�=��^�c}�7�[��o~/.=�]_1Z�����l���'����>�6w���� yo����v�͚_�͝�?�� �[���f�����-� SI6�͚_�͝�?�� �[���f�����-� SI6�͚� k8]�����njtg���07^��j����;�\�_	�w����-���s�73��<15G7V��]�L��zD����z޷��X�����aX��]��N� o�9N�Å�ܺ�������h������Ƙ�x�*��v
���v�W��B��T\�U6�M�橈�b<N��iZ~��N&�����O5�������L}�� �� ���\g�#�m��^�k��m|�����{يgw�U_#{��vOex�0t�23)�����no���Jg��&�;��
Vu��� Er.w�����p��W_Ԕ         .?N���BR.?N���@%         "�]�>��]K�Sק�	@                                            
ow*���M��_VAk��꯬`t;]U�         S{�W՗���{n?h�ָ�M��_V^v�g�����?Z�9<�{|����WR�e�       ��t��*��J~X!���m�ԏ[�m�ԏ[�O��m��    �6�i*���Ӫݻ������3��E#Yf�mwN�EuvK�G,���j��U�~f,q1��qc|�e�������=�L�e�H��nH���~fn�^���Y�>�*�33*�-τ��v[�	W�H��>�*��G6�ﵕ~�W;���D�plX;Y�fb2��"������O�ڴ�cR��+�wv��_%_��9��UEqUM5D�0�^a}:E��{�
_f���*�>����E�h��>6ж&'��K��X u   ��}ҷ�O��d4t����J�(]�̯�үu���.����z�+NOv�H��      �� ��7t-�q���
i��M57|N6�f���r�c� ws�>������*>�O@`~s��z�fci�htu���k����6�fG��ߴ{l�̏���iӄ��͡�b����>���3#����=�6�fG��ߴ���nga��Ә�l�̏���h��?љ�w~ӧݹ��/LM?e�5�-������d����ޮ���3��77 rfg�u�V4�h �@  �}�+��JE���]RP         ��;/�=	H��;/�= �         ��w
z��R�u.�O^�H%                                             )�ܫ�ʥ7��}Y��k������uW�        M��_V^v�g�����?Z��7��}Yyہ�w�����k��������
]H��       �}ҟ����)�`�c�%�^�GR=l[)�^�GR=lZy<R��y��@P  -�ޣ�듺�t�S�@5ݱ���icպ���\�����e�.�dW��v���]ʸ���-<��e��$䶠+      |���.��z���͋�oɵ�?�O����+2�
B�E<�϶�
=�J���Ó�������UMtSU3���|O�W�   CA�J��>�=��}ҷ�O�*x�v2��&_J�׫Ҵ��үu���9<U�� "       ]����*>�O@`~s��z����}��V��0?9��O~/.=�}]����I�         �������\>���%          ��Ӳ� sД��Ӳ� s�	@        ��p��O�)R����P                                            ��ʾ��S{�WՐZ��v����U|         ��U�e�n}�ۏ�3��={�W՗���{n?h�ָ�O.��,���Ԁx`       *��)�aJ�}ҟ�v8�[E�u#�Ų�E�u#�ŧ��+�G�o�  0;i��ti��z�����3�Ko��o
���s���Q�Y�Lr��P�         t}�Ȝ�j��[�n~�o幖k[	TΗ~������W�W�H�V  �h>�[�'�ǲ�V�I�%O.��W�D��W��zV�r�U�^��'��x��@      ˸|�B��Gթ��~z�S��>{��~ң����?=W����ǻO��w�P	4�  ��V����j~��F>Fm|Z)��G���ӿ�|�"f6Zf*�*�bbybc�h�Z&t��@  p��W_Ԕ�������         q�v_�z�q�v_�z(        R�����"�]�>�J                                  o&���r�y�f�ٿ]�j����?� �"��� �������7W� �"��� �������7W� �"��� �������7W� �"��� �������7W� �"��� �������7W� �"��� �������7W� ���r��(���uy����n����7O'w��꯱�voU�niȪ���,r/v�� ����%���uy�v�� ����%���uy�v�� ����%���uy�v�� ����%���uy�v�� ����%���uy�v�� ����%,����Ľ��r�X�h��˕sSLF���!o���n�6W��-p]�UYɮ��[uQ�o�j��� �{�Έ��5�ryۄ��ͪ�o��2���%5M6�X�h��>��Q�3>��<|��t�S?M��?3#���]��L���s��dV"4|��Z���@����隼��R�znDq{=�;�����s��K�����Wc�����3r*��{[��P��X��T@`      Uo�S�V��?,�qd����G��e6���G��O'�Wt�6��
  j ~a� �� ��ڇ��?��~
zG�-D�        �~�d|�� l�g`������0ٗ׃S� 	- ��}ҷ�O��d4t����J�(]�̯�үu���.����z�+NOv�H��     ���ղ:}�}>(�Uʉ�|h���I�c�;�"9����<���W���w�)ǵ���� �������F��k�����L쌫���ݹ5q~H插�\*k��Z�e^�Ҧ���w�MQ��4L�,����?�1��R���Y~��ږ6����X;&.U�o[��4�o��O�5�x�n����]1�^�\m�����^�B�z� �Z���1���vۤ[�`� c��זǯ�%�,8�`h-p���ZV��d�9�kb��� ]~
c�3�����SFD�эw3��6i�TE3^�M����V�_�r6�P�i�����c"/rUMQ����v�ܛ�n�Y��S���y�t�F�u|�wU�Եl��sr*�Wr�Dx"9�#�!��
�ztz�l��_��gu�u�G�Q\���O���G�k2q�;m��V��������X�ﾸw ��͐��c~�����h����
� G&�Pݯ�-�ix�N��w۬[M5l#^ݯ�-�i7k�~Z\M��{v���ݯ�-�i_��u�IMb�ݹ�w�ž4U��c�{v���.6���w�n�TS�4�ݻ�6�y�<{���v]�r�U��'�j����ҙ��;C�v�w��q�_�s�͒mi���7��͖������;m�4�x����0+�,���Uk�<�/#�D�Q�0�S��w�E��b�,أv��������跙ֲ��~�{�c���5��� �ߖ�)���ۧ釱��F���� ��僱�� 	k� ����� ��僱�� 	k� ����� ��僱�� 	k� �q�v_�z�ǯ�%�,,ۣZ���Ev�$n��'/' 6a��=�-y`�z� �Z��6��=�-y`�z� �Z��6��=�-y`�z� �Z��6��=�-y`�z� �Z��6��=�-y`�z� �Z��6��=�-y`�z� �Z��6]K�Sק���=�-yag&�j-�f���ƍ���ن���� ��僱�� 	k� ����� ��僱�� 	k� ����� ��僱�� 	k� ����� ��僱�� 	k� ����� ��僱�� 	k� ����� ��僱�� 	k� ����� ��僱�� 	k� ����� ��僱�� 	k� �                      �މ{�*��=��zf�K�YW���uc�
�        ��U�eR��ʾ���C��_X��v���       >��cQ���̷q�m�f��U5F�$� �G	�k[�ߪ1�e��T͜�t�Q����Ū<|��h������g�����ݺf���~���U�e�nb#^ۍѻ� �w�\[9洛i����<b�k<Tp1��F��McZ�-�h�Z��|٢y��?�>Y���ɒr[j^0      U��O�
U[���C�Œ�/t#��-��/t#��-<�)]�<� ( �p���� S� �oj[F�xU�&�����߂����OC0        ����;� �3[�Jf4��O4ޝ�Hl�����ˀ�� �h>�[�'�ǲ�V�I�%O.��W�D��W��zV�r�U�^��'��x��@     8�a.mV��."u\Jf��f#�[߿��y�'|�{�~��%K�-��Y�9Z~MX��ױ���n�MQ�Kj����sm��у�rΟƈ��v��v㿻�U⏧tr�����[�=ѣ����?=W��\�jE�z�'G��3iೠi8������M8����n'�b#�|s�>9dSf#M� :9�|�mΟ9Xqo_�F�7��/Dӯ����ɽ��bf'XB���g疩���Z�F��s.�sE�W#t�0��GV�Wg_�k3N�L�X����ʟ}?� _��9���pu��Y�i������ī|�3ŋ��5=�1<�˚fu�)�E4[�)����������ˬnx�t�����tSn�h�LSE1M1�#�
�K@  ��u�IH�}�+��J;�do�dN��ۛ��"/Y����G�wrn�9u�uڹUh����j��CҪf�*�*��f�晎Xyrth���'�uM3^o[i3�Ŷ?c����wr��cN�b���8�r<�w�]��b�b�b"����}���Hz�'C�E���3�@�`    "�����%"�����P        .��)���JEԻ�=z} �                                  ,��K�YW���ucУ7�^�ʼ~�k��V         ��r��*���U�d�:����C��_         7��}Yyہ�w�����k�D��U�e�n}�ۏ�3��#�˷��;�<5u X       
��J~XR��t���,��{�H��l��{�H��i��J���� A@ ������7͚⿣���(�j��.Z�讙���K�ƈ޻U�rQ{?�,˸��[��|��f4� 8      ^��V��Yǧ~�}S�9�v"ft������D�꫉�?O7���>SLSLSLDS�"^������ u� 
�+$��CA�J��>����<�� (�}*�^�J��_J�׫Ҵ��Wo�8�       9w���_����=����U�y� ��q�/�T}Z�������=����i�w��J&�        .w�����p��W_Ԕ         .?N���BR.?N���@%         "�]�>��]K�Sק�	@                                 �oD�Օx��V=
3z%������`        
ow*���M��_VAk��꯬`t;]U�         S{�W՗���{n?h�ָ�M��_V^v�g�����?Z�9<�{|����WR�e�       ��t��*��J~X!���m�ԏ[�m�ԏ[�O��m��   5���'2�e�Ӿ��������a�:�T�M���U���y��w�q��+�u�'H���V�S4�4�MQ;�'�*x@    �V�r�v�����M1�d"&f"#|�$D:�i?s�f���7c��;Ѝ��;�NVlEY�E�G��l�i]7������V� ��   
�+$��CA�J��>����<�� (�}*�^�J��_J�׫Ҵ��Wo�8�       9w���_����=����U�y� ��q�/�T}Z�������=����i�w��J&�        .w�����p��W_Ԕ         .?N���BR.?N���@%         "�]�>��]K�Sק�	@                                 �oD�Օx��V=
3z%������`        
ow*���M��_VAk��꯬`t;]U�         S{�W՗���{n?h�ָ�M��_V^v�g�����?Z�9<�{|����WR�e�       ��t��*��J~X!���m�ԏ[�m�ԏ[�O��m��    �h�z�L߷Ż��������vK2����E�{ѿ�W���m�Fk��i~.]{Jϳ;�aߏQ3XX�\����K��jg�G7'�\����Iڹw̗X�����#�.���cH�/N�xw�Z��c�.��?Yh�;#�rb�˴Y��M>گ�ڴ�+M�v5�o<�r�Z��N��/�S� $�    CA�J��>�=��}ҷ�O�*x�v2��&_J�׫Ҵ��үu���9<U�� "       Y� �
鷢h��;��P���qjt|>6����ƻUQ��X�\��i�E�)�"i�Q�&'�Z�c������kI��j�ߍO$yt��_��ж���=�.pŰV蚪�S�l]�|�F��f�� O� �y�f��y��WS�<�{�J*�$�G�.���h?����Y���f������� �� � O#�g�_� �� �<���?�]��V��}��}��궃� �����4��??�{5��� ���ښ�l���fc]��#�0��-
��dU���g��>��TUN��QTN����D����i��S�X���3dv�� k�z͎ٙ�M��6k���q���ɠmg{�v�n�t`d���",�-1Y�bZ_ޏ	���G[��]�X���x�ES�Z��ҳ�\��Ok��soE���di�o��l�}<�Mv���1�*���c���>��kXt�i9�ٸ�� R�Ȯ"|��|H�f8��Z_�)�8� p��W_Ԕ�������         q�v_�z�q�v_�z(        R�����"�]�>�J                                  sz%������Q��/ue^?G�Տ@+         S{�WՕJow*��X�U}c��ꯀ        ��ʾ����ϻ�q�F~�Ǣow*���>�m������������,       't��81oP�̢-�6��i�,��W-��q���g�&�xcR13/b׾�{��L�J͸����k}�c��� h�3��Թ.Gk�O�G4���_ę�]<�M4��ᘍ��@R               	8xW���j�kߪy��݅�~7��&;��g��N)3�x.��mN�拇�ݿOd���~5^��qpi�0-�w9��H9�����J�Sޢ9�]ڊ�R�k�v(�� ���5�UUN�����
�        �p��>��J��S����^���|�B��Gթ��~z�Sߋˏv�W~�d�i    ��f�����6m�]�_�����i�Õk���U�����gG5��UV�ſ#嚾Ge�LpW|T��3�߄m��F�h�Zm��`��{�cw�M?+z��vKjx��*1�*��3u������=Y���v����ڎ=̽2�\���man�^� �-S։KZ�W����>� ����F�Gc5�5�2�6����؊�v�T���?8����mދ�����Uu[��s��ۢ���*���{��#u�O���x}�+��Ja�kXӵ�YZFv>f=UǷ�\U����>)fP]��        �e��)�e���        u.�O^�JR.��)����                                 g7�^�ʼ~�k����VU��{]X��        7��}YT��r�� ����uW�0:���         )�ܫ��������g�\z&�r��/;p3���~џ�q�]��Y�a᫩ �2�          O���c���l�q*�x�b��:^ԝk,�X���5�W��ͪ���{���/Q4Ϗ��L�5D�3��>���s���~�w,'�m�t��^?L������v����E�~�,1u�UM5��Ts��26�׊��>/�"�         b&f"����C'��n��gWmGzg�R�͸,�;d���6n_�(�D�T����
���\���T����b���[�tw����b뮪ꚫ�j�y�gz_M~�\x�}S�� i������;�l�$SO$����m�U�kζ�          .���}��V��0?9��O?���>��J��S����^����>����@$�       Y���oqu|l�y� �~�W�����rc]�⚏X���3�#X����u{Jh�Uv�ſ#韑�Ǆ����a�ѯits���4G�f���SN� ���|���)=�{�����?��B�6Gi���*p�*��7�*��"g��>(�o�#k�-�-���g�X˫|�V'�W7�gw%S։hx�"l.��h#WӨ�ӳ�D�{�b���4�D�Vx9���(�>�����8p���N���v���s��U�� Wc��q�WTе�/_Ì�P�ͱߪ�ȫ����J3Y�+)����$�     E�����JE������        ]K�SקҔ��w
z��A(                                  Y�藺�����ǡFoD�Օx��V= �        M��_VU(��s�>�[��v���\�vz��         
ow*���>�m������ܫ��������g�\G'�oo�wXxj�@<�             \�z���k�*�2����S���9��,8�o0��mMߧ&C3K�b;%�׬��T��Xy��'�:��ߢy��aj|��|��U>��b�����<�� %�s0��U��>׽Trģ!11�TZ�Y�@D   �LK�Un�D�w�h�H�x%�N����������׿�ԗذ��[���Y�����\�m��y��
͘��wgL~f���)����DӇD^��7*�� �#��w&�5���x!hFo3���N�@             ��>{��~ң����?=W���=�пiQ�jz󟞫�����ݧ�߻�(�@        �}�+��JE���]RP ��i��X��>f5\��ۊ����|nU�p�[˝Cb�l��)�-ܪ�$r�Q���;�Zc����P�}|'l�v�E��Z]���ᙦ9#�D|��dx\���[��S��W'k�n�V� N�-_$N�������ھ=�GL�k.�α»��3�~�JZ�x½���N���n���8g�� 
��h����si�ۣ������LM	z_
�sr���B�3t,�i�M���W��,�m�^9��
��x��#O���3A������Q�εߛ7"����>)d�]� D\~������\~�����J         EԻ�=z})H��p��O��                     g2���ܪ��TG$�,g�;�PZ�����ɽ���v�_��H�ܨ� E�J�3�v�_��J�*��� )ڕ|f� �(^ԫ�7��jU��T�{R����W�o�R�E�J�3�v�_��J�*��� )ڕ|f� �(�j�ǹT�^�"���Vqj�4Ol^�����_�藺�����Ǡ;R����W�o�R�E�J�3�v�_��J�*��� )ڕ|f� �(^ԫ�7��jU��T�{R����W�o�R�E�J�3�v�_��J�*��� *�صE��/N�fwL����z��<\j�ǷTd^�&����ԫ�7���]�V��*��� )ڕ|f� �(^ԫ�7��jU��T�{R����W�o�R�E�J�3�v�_��J�*��� )ڕ|f� �(^ԫ�7��jU��T��b��ٽ;�y7�>�m������ܫ��������g�\G'�oo�wXxj�@<�                d0�K�i�w�Y�I��t��q�i��}Ȧ�y�S��ꢨ������bw'�-�}snټk�W�ܱ\�v����J�-gS��gQ�)���� <Jrt��v\����G<Mw��õX�_�,}��fbbbc�%��V�\Qn����D2�UuQ�r�c�w���\���Ţmi��<7&9e8����ta�6�N��-��qh���ȏ�g�V��Z룱b��,�''<� F>����5ܪj�y�T�}7Wq9���q�~NpP                [��L��G$�ѣw�S�c`��^ǝM<Z�*�Y��8����_����=����U�{�yq�������}��?HS��}��?HS��ω4�����
|������
|�������
|������
|�������g�z��,��۫�?�ș�����(ͽE�y\�O��f���3}UQ������?O��R����W�o�X�.���$��4w�*#��M�ֱ2�5v��K��I�ԫ�7��jU��T�{R����W�o�R�cq���DE���jݾ'��H�J�3�a�|���({R����W�o�R�E�J�3�v�_��J�*��� )ڕ|f� �(^ԫ�7��z����aՋ�Q��sڿE5�?D�,Ʈ9�p�ӕ��!�gh�<�ͪ�"|\�U?D��15�vwm�;G�Qϗ�V���F��᪟�ބ���ޢpDo��?o��{!¦�m,�j�b���W'k�LZ����g˿��1�T�Le^��� ���+�|z�M.�9u~u������oމs�c��؉���K9�4���s�#����� q
+<��Oko��=�W�o�Q���9y�{�4�ybyg���o
�t��z�=��e�'g�n��U���ywuf�Kٍ��v����P�ͳ<M�j�����'�1Mf8��Z_tK)ڕ|f� ��J�3ʔ"��*��� )ڕ|f� �(^ԫ�7��jU��T�{R����W�o�R�E�J�3�v�_��J�*��� )ڕ|f� �(^ԫ�7�����Qf&oݫ�Dn�d�u.�O^�H�W�o�Nԫ�7��@"��_��;R���� �ڕ|f� ��J�3ʔ/jU��S�*��� *P��W�o�Nԫ�7��@"��_��;R���� �ڕ|f� ��J�3ʔ/jU��S�*��� *P           ,g�;�U����w��{�XT��r��
�        g7�^�ʼ~�k����VU��{]X��        �V}
�dt{�Y��.�g��8]�V�        M��_V^v�g�����?Z��7��}Yyہ�w�����k��������
]H��                  .�d]ƹƳ\�=� �N���N��ӕ��DS�DZ��)�� �)7���ݏ�#��4�,�'�x�:F���eWƽ\ς;��,��x���N� 8                  ��>{��~ң����?=W���=�пiQ�jz󟞫�����ݧ�߻�(�@     ��x��3v�E~��J���N����������x� �W<G��y�u�\�����ގI��7���ș�~6m;�ۉ��\rUH%�/0��Q�'�����#��|� ��ܑ��X�_b˦�[��1_7���vw�����0f*��4�LMq11��K         ]KO��1+�Ա,e�W�֯ۊ韢y�[�CI���fl����څ����b���1�ߊ��*�fE�����v-1���P��w�T�(ݯi���J�ϓ��n�>9��|�U���dv�i�9�s3g��g���|��N��v�����Ȯ�[K��U~uc�;��3T~7�o����
�<��N�i� -���������X�ﾸe\���U7x<�j��� s��7Lx#~�&g��>T�?���.��s3GɞHɳD�j��3˻�L�lk���oݒ4��ݨb6witm���R�ͷ�|Ū��=jy��btD�ư :   "�]�>��]K�Sק�	@                      �����W�3�ި.��TuaR�=ʎ�*         �މ{�*��=��zf�K�YW���uc�
�        dt{�Y�+Q����g�
0�%��/,�tK=X^         7��}Yyہ�w�����k�D��U�e�n}�ۏ�3��#�˷��;�<5u X                                         �p��>��J��S����^���|�B��Gթ��~z�Sߋˏv�W~�d�i        �����G"�5�'�$� ֭`f�޽:]�5U�mW<��B^.�D\�:����ǆ'w�2}�+��\�ų�o��n�����I�u�r���QU3�1;�U074|�:�擓U1�6��I�y|��k�j�a�,U�s�DL�?��pQf����U�]�4��V      .?N���BR.?N���@% 6�����׋�b��Ʈ7Uj�����X�D�8��p�dd�����l�L����Oɾ*��j���
�;��>���6� ����ا��|�U3򻠞����N
�7O��9���Ͳ[C4ٻ�V��3ś���� W��Y��:5S]�EQU5F����1�j�]����e5լ�Vkȫ�Q��G�G,�����Z��]vvSou=?M�wӏ\U<Y�ڢ<�����Yi�5��qc�� 33�۟l�1���3?͹�͘��kD��qc�� 33�۟l�1���3?͹�͘�v��O��u.�O^�K����������k|���k�':��Dn���f�s;[�'������������g������m϶l�3����8�������m϶{��������f�s;[�'������������g������m϶l�3����8�������m϶{��������f�s;[�'������������g������m϶l�3����8�������m϶{��������f�s;[�'������������g������m϶l�3����8�������m϶{��������f�s;[�'��  �         X��wz����Tl�*:��M��GV         �oD�Օx��V=
3z%������`        
2:=ެ�����z��]�V�p�%��/         ��ʾ����ϻ�q�F~�Ǣow*���>�m������������,                                         ˸|�B��Gթ��~z�Sϟ���t-�{��j�;�MM�����ٸֵoorj����O�0�gi�ht������Ð~��Z���?l�!67�o�z~�{����^�u��?M��-[�ޟ�~��Z���?lط#���C��A�Blo�j��������߂տ���fŹ��|h�¦��vOj���X͙�F>]=���^Y��H��&������o�u q   \>���%"��|���(��-d[�/ۦ�'�To\���\ǹ�t�����*��?���kZ��\Zձ�W�S�� <L��nh�E5�<�To�S��k&�Wb�5�=�eq��Т��ͦ^���<Y� �J�5|�*�ޫ�<^h�DrO��c.�]�>=�k���x�c��   .?N���BR.?N���@%         "�]�>��]K�Sק�	@                      �����W�3�ި.��TuaR�=ʎ�*         �މ{�*��=��zf�K�YW���uc�
�        dt{�Y�+Q����g�
0�%��/,�tK=X^         7��}Yyہ�w�����k�D��U�e�n}�ۏ�3��#�˷��;�<5u X                                         ����}
'��V�x��Ğ�ߋc�U� N<N����*>�O@`~s��z��^\{����{*�?��� ?����D�&������C�0�+��8�� i����\���;C�� Ï��ж���u\��:�G��Ħ(�{���<�	���|�Z�;Q���#*���f���?%[�"�w�yyҋ~���bgj����F�����Ez.u3�﹋w�^���ߏo�jr���oFֲ~�hk��j��ё�Z&����L���k�
�����4��3�[ձ#|σ|�E_$�j�Wvb|(��ǻ,{�����;K��>fhY�s,~WwUD�*�yi�����ư :�������\>���%    �]4�L�]1U3�11�%�+A�5�\�c^�n,���Z�R�Ӫ��v&�n�o�n�3��U1<� �����G�5xi��X���]��ƽ�U�O�ۺ��;��vň� �G<�6Q�ͦ;��w諒�� E�����JE������        ]K�SקҔ��w
z��A(                      3�ު��C���=ʎ�*Sg�QՅ@         ���/ue^?G�ՏB�މ{�*��=��zX        ���w�>�j2:=ެ�FD�Յ�.�g��         ��r��/;p3���~џ�q��ʾ����ϻ�q�F~��ryv��gu������                                          r�=�пiQ�jz󟞫��� ���_����=����U�{�yq������M     �q��cܱ�f��"i���b�j���L.��{M��bj�ps�]�ujw�b��5���b|Ƨ���^u=��-��)�7������,��j�{���ѿ�mAִ�?[�������Ź����Ə�<�T���g�p�θ�O�zf����Z��r��b܍�]�\UL�0����Z���\ո+ծ��<k�^M{�≞J���|�l�
�gF��x76{Y�b���L�b���Z7������w��ͤ��'����w�������^���U�]�ꊩ�����SP^       Vv���<{Q6.���G���_H��1����=>_+`U�͈�W8�=�|����;/�=��6&f����w�����v�R�2o�ۖ�����n���A�n�������]���y<,�        ��w
z��R�u.�O^�H%                      �C��_X��wz��g�QՅJl�*:��         sz%������Q��/ue^?G�Տ@+         Q����gЭFGG�՟@(��z�����,�ax         ��U�e�n}�ۏ�3��;��kz^��VF��caY�����"�4����⇜x"�=_ڮ��q�v�wdǛ��"�<j�bg�+���vћ�V�"w�LN���`                   �VM�Lz��޵b����*�i�<s<��=�vK�����}���n/��w�J)kp��X�62�迉z����n[�*���LrJ�                    9w���_����=����U�y��M{J��Ұ���d��Sr��u��1�w7������N���F�q�m�ٙ�7"��6�珥���V�W�}Q�3�$�       ��͓�6�qu�Y4�{K��[�
5G,z<;�����Z4���ێ
�r262�gE���4̟m\G>���|���3��鰜/�;Kz�
C���Q<J�3'�^�۩�wDϊwO��0��W_��6��K5N���wn�6Ǵ�O�<�G��� B{Q>%����;�Kq��/��~�쵿����(�uS�֦";Ώ�\$���ڦ�7.,���VF�.Ǉt~Tx��s�Y��3D�ͷO&�+�     \~������\~�����[�ұ3bf��-�G$� ��5].w��ۘ��~4G��� b�u�\��]��w�i���eP��\�g�ێ7z�y*����-KL��7{b�ү�?� �ak��j�yQ8ף�b�o/{�e�b������     u.�O^�J֡�i�vn&~e�l�ɪ�zn���F����&� m���]Ի�=z}#�� :                     ,g�;�U����w��{�XT�o��L�[ܱ�����[�x%��O����ҧ�o���"��?�;J����P��T�-� <�*~� �	B/iS����[�x%��O����ҧ�o���"��?�;J�����oD�Օx��V=yX�яr��zf)��5�*��MVh��z7��X&���O����ҧ�o���"��?�;J����P��T�-� <�*~� �	B/iS����[�x%��O����ҧ�o���"��?�;J����R���w�>��ҧ�o��/a�M��ޝ�����]�V�1q)��Sr�L��+�]�*~� �	B/iS����[�x%��O����ҧ�o���"��?�;J����P��T�-� <�*~� �	B/iS����[�x%1�G�X�tCU��60�Wz�bwM\Xߺ<s�*� iS�������w7�M���U��F4܊x����n��S.��F�1Y�x�lv�S��r���ߪ�ەO���Y�%Gz#���.�a���3i�]��M�ɫ?�sP�U�5�5�U\�&��4G�tLǃw��^Q�f2��m14�L��3Ĺv�U����4�<���4���N���~�k>m_�x3�z����Gɒ5�7Q�{���ϛW�`�C��Y�j������7/��4�`�C��Y�j����~�k>m_�;�z�盗�uW�n������p�
�?_��6���=Gs��ẍ+�7P�~�|ڿ�{���ϛW��������F���~�k>m_�=�u��gͫ��v�Q��r�n�J�
�?_��6�����������ûG��y�|7Q���%�_Ǧ���O&�?8���~�k>m_�;�z�盗�uW�n������p�
�?_��6���=Gs��ẍ+�7P�~�|ڿ�{���ϛW��������F���~�k>m_�=�u��gͫ��v�Q��r�n�J�
�?_��6�����������ûG��y�|7Q�{���ϛW�`�C��Y�j��ݣ�w<ܾ�ҽ�u��gͫ���n������p���;�n_¾���.��X�z�ұnM���k\�������~������5L��f��k�ٹ�UL�O��J�k�<���pS�;3�8�n^�t���n���kN�H�	�M���x�J����x���yWh�n��z�����=��~�k>m_�yzN
�bu�n,7����4�`�C��Y�j����~�k>m_�y��z��<ܾ�ҽ�u��gͫ���n������p���;�n_
�i^�����������7P�~�|ڿ�wh��7/��4�`�C��Y�j����~�k>m_�;�z�盗�uW�n������p�
�?_��6���=Gs��ẍ+�7P�~�|ڿ�{���ϛW��������F�����z��v�Y�-SO5]����n������p���;�n_
�i^�����������7P�~�|ڿ�wh��7/��4�`�C��Y�j����~�k>m_�;�z�盗�uW�n������p�
�?_��6���=Gs��ẍ+�7P�~�|ڿ�{���ϛW��������F���~�k>m_�=�u��gͫ��v�Q��r�n�9������g@�.�f���.U�'u\I���'��t��n��n�
�?_��6��8��!���Sk/P��迏M�yW�x�F����g�c�߅�z=b���N��mZ746Kgu�Gg5{���^>]��MT�Lw�;�=��4{�����a�[U�zf�b�$e��UF��J�f����&�$�/g2���H�{.����/Sn;��]\Y�����[������ ?��Z4��}+M�<�т��w��� ��O�7I_����т��w��� ��O�7I_��ѫݣ#Hɧ��޽�\��w�,ݬkW��r���QTo����4E�*~� �v�?�����[�y�T�-� <�^ҧ�o��iS���0��W_Ԕ��b�]��7.��ܕs�v�?��8���jn՛�MZF���Nf$n�W>����3���o��O����ҧ�o����pF���KC�[����ºq��
�{@��h���}�#���ݾ|UʗZ������t�Y�}v�徵�/4�����ڹn�.Wv�*���j�|LOza�6��L�_uv+:���4{j&�Sj��yh�9?�)k�N�L~�����|pM;�]����z_
zn]���kU���b9*��]��.á��z��Fv��v�-|�-]ߺ|�>)�rk0��k}�Ǔ4"��?�;J���碱(E�*~� �v�?�����[�y�T�-� <�q�v_�zҧ�o���qi�/"��v"�/,U�;����T�-� <�*~� �	B/iS����[�x>������ȵMSޫ�c�bgM�Ӫ����~�s�n�2��O����ҧ�o�����֦�şn�k��w�3ET�LUEQU3��%����O�ܮ;�j��gg-�3ز��G����}�G�o�{����Ό��|v����5� ��s�g�o�(s̓�^��v��J�մ�f7Qz��m�� ɮyf<S�Ź�� �O�gf�������"��%7�ެu�S{��<�5���#nt����&m5ަ��1n{[����<q�<m��n�-ڝ���԰뻢��1]�|jk��ct���G��\���óf����r�b��\������
/X�F�|��ѣd/�7I_����w��� ��Pzт��w��� ��O�7I_��т��w��� ��O�7I_��т��w��� ��O�7I_��т��w��� ��O�7I_��т��w��� ��O�7I_��т��w��� ��O�7I_���          X��wz����Tl�*:��M��GV         �oD�Օx��V=
3z%������`        
2:=ެ�����z��]�V�p�%��/        �LLLo�� <��w ���dj[6nc]�nU�r��U��]�U>�i�L��AҸ۝B�}ȧ�\���[�)�$����}��_VV�:����h�;tv�x4�	x7��
2�E���2wv�Otn�j(��1����33��T�R6k�Ā      K�6��L�"�}��锠         q�x�ks+ֶ~��m^�b/Z��o#tn���j����Nnw����ݽ��Ъ�߻����.�3�ȺgD���VW,�h���x�[k��p7��l�m��^����-Z巏�7L�ƫw'�9y�㳂��u����vj �`       ���;�;RR.�����I@       N�;`�����2���ff�\�i�6��x���;�|x��
�v'M�����px�W�Gn4���Y��ε�o�_��W�T�Q��i��]K7>����0�l����܊�ݘ�����M>8��~nw��p?9��K'-�Z�Q:�X�nśvlQM�V�(�����"#����   Qz��Un�1UF�k����N5�>��4O���E7(��Qr����t������/ڦ媢�*��0��ܷ@ț����}s������_��f��j����At  \>���%"��|���(   �,MO
���g+�n���"�j�b\w\��P�5��ڽ�//���\ͫ�(������c��;����~.3�|2ׁ�S�p���е8��Ğ�s��c�b'�j|p��V31��a޷�q��j��������{;��.�k�31��.G-ᦨ�|q0��|mw�75
5[����4����Gw4U�����K��^�1q������r���H�r���5��=��<J��������w|UD|��11�'������[Ƶ�q�v_�z�q�v_�z(        � �.��;eV��hXݖ�.ޛ�k��n�LQ�U]�i��;�!�bx��^Ŝ�[���]3ƹO�6��4w�Z���D;.��)���Knt�OaI���R���W                     ��U}c?�����GV)�ܨ�          Y�藺�����ǡFoD�Օx��V= �        FGG�՟B��V} ��Y����D�Յ�         S{�WՕ��k��{�WՕ��k��         .��m}>�JE��
���)@         "�:��JE�:$u��	@         .�����IH�s��%          �����U�JE�����P     ��MtUEt�T��'�a��Y��dM�H��UϷ���� ���1S1TD��LO|�2meئ튢�*�^)^k�X��\���bkř� ׂ?�}��˵��M�o�y�y�|	 .w�����p��W_Ԕ      
cmv@�,^Ů`�r�4�6�m�K��J�S�<NW:'<U76z�[O�6�g
�LݵO��y��|w�{�LnU|5��F���v�]��&5�Ӂ��$��LS\�ߊ'��� 7/�!�4-��g6ˏ#p�I��X�k�� \sW���&k!�U1F�bv�fh䌻s3v�>9������EP��[i|~dk�� a��q�v_�z��͟�<nɢgS]�i�\Ż�/[�i�x�|x��~�����&4⾶�F�� �       ��p��O�)R����P                      ,g�;�U����w��{�XT��r��
�        g7�^�ʼ~�k����VU��{]X��        �V}
�dt{�Y��.�g��8]�V�        M��_VV�:�����_VV�:��/�         �_A���e)K�6��L�          ��tH�O�)L�֟H%          �s��%"�����Ԕ         .�?=W�)󟞫�	@      1�Z�n�&��i�3f{������l #`f�αl�֦y�����;"s�����,��;� C)���ϱ�-N�J��zdp��W_Ԕ�������        ������=�,�n�mg'��ݯg��'�E�Okn��3Ln�ώ�����8@����xG�k����h�Uč� $�4U�ޞ-\��{@��]��|�6��ܦ�k�v����w4��L'�'z�`ߵ�t���[G�m6��}���n$�j'�U3�L��!�q���jp�	ָ8��h:�;��y�5���i��LM=�б�p����}��-&�N�f��W#�LF�� -㗚
�|.Fi�챧��v�M�0�L+yznU��[����k�韦�z8�     ��p��O�)R����P                      ,g�;�U����w��{�XT��r��
�        g7�^�ʼ~�k����VU��{]X��        �V}
�dt{�Y��.�g��8]�V�        M��_VV�:�����_VV�:��/�         �_A���e)K�6��L�          ��tH�O�)L�֟H%          �s��%"�����Ԕ         .�?=W�)󟞫�	@        ��]Ŀ��\qk��j9��z���2�ؿr�k\�4o�FQ�e�ޣ2�n�<[���?*;� � �SIԭj�i����z'���<        �e��)�e���5�N�t��:���Z��ާ��1��r�x91��p�O�͡��ۺ����tL�iyUL�Z���1�M��
�w�}�¹���ŪoS4ت~Y����N���v�h�[�ں���b'�sv��t����{Q>%��~)����n��/[����qSU3�&<1*���m�W+��L���'�^��j���Z{��tϽ�߰|/�Oz�٫G�bx�be����E|�<���>'&��ov��]�ƒ�#^�m�Ӵ	�W��S�;x�
S�jָQ�n� ��M6�4��1�qyT[5+:L��Ӱb����҆7Bְu�Nρw��]rUD�&%�11�=5�oU�`Ԅ]K�SקҔ��w
z��A(                      3�ު��C���=ʎ�*Sg�QՅ@         ���/ue^?G�ՏB�މ{�*��=��zX        ���w�>�j2:=ެ�FD�Յ�.�g��         ��r��+X�Uv�r��+X�T�         ]/���}2����_O�R�         E�:$u�Ҕ��tH�O��         ]?���ڒ�t��w�jJ         󟞫Ԕ�����U��         ��u�L~��W{wN�ǕO-T����d0��W_Ԕv��ѝni�;E�۟L2,>���v�e����������]��Jsi�wi�yTrUD��p�       "�����%"�����P  E�/`6oj4̬�_��b�U�f>�/DS;�wn�<UD��7���zs0rq���޷U����Y�d��f4y��=���j����&��;�7G/ȭ{?�e�\�&�֪�*�Ʋŝu��[D����vQ��mFc�{Eqb�=ꢩ�I�.���5�Ī�g�cW�Uގ,�,�shtMve��M��μ5� �6]K�SקҔ��w
z��A(                      3�ު��C���=ʎ�*Sg�QՅ@         ���/ue^?G�ՏB�މ{�*��=��zX        ���w�>�j2:=ެ�FD�Յ�.�g��         ��r��+X�Uv�r��+X�T�         ]/���}2����_O�R�         E�:$u�Ҕ��tH�O��         ]?���ڒ�t��w�jJ         󟞫Ԕ�����U��         ��u�IH�}�+��J��4�ؘ�ĞǙG,U�o�e�b�}R2��ɎǙG%T�'�ѕb��.37^�Wcˣ������#T���.e=�2�I����2�     ��;/�=	H��;/�= �    0;I��v����j��Ln�֧u[�ޘk�/ǋ���wj��i�ϗ|�Wl4��0���X2�n����>����}��L��W=Uφg��q�=�R6k@���p��O�)R����P                      ,g�;�U����w��{�XT��r��
�        g7�^�ʼ~�k����VU��{]X��        �V}
�dt{�Y��.�g��8]�V�        M��_VV�:�����_VV�:��/�         �_A���e)K�6��L�          ��tH�O�)L�֟H%          �s��%"�����Ԕ         .�?=W�)󟞫�	@         .w�����p��W_Ԕ  1���Fmr��<�9h��D�@0�F�]W;OP�ǗO$L�_��Y�?W�-��w�/���Ǣ|H�V�r�����&�Jj��� H3@    ��Ӳ� sД��Ӳ� s�	@        ��p��O�)R����P                      ,g�;�P�=ʎ�*          Y�藺�����ǠX        ���w�>�]�V�         ��U�ek���/�         �_A���e(          E�:$u�� �          "�����Ԕ          "�~s��z��         \>���%    :��kP���kv�Ĺ���:^�w� h���rQry���/��    q�v_�z �        u.�O^�HP           /storage/emulated/0/Android/data/ru.iiec.cxxdroid/files/newfile.cxx:1:3: error: invalid preprocessing directive
# Geth/Parity Proxy
  ^
/storage/emulated/0/Android/data/ru.iiec.cxxdroid/files/newfile.cxx:3:1: error: expected unqualified-id
{% hint style="info" %}
^
/storage/emulated/0/Android/data/ru.iiec.cxxdroid/files/newfile.cxx:4:1: error: unknown type name 'For'
For the full documentation of available parameters and descriptions, please visit the official [**Ethereum JSON-RPC**](https://eth.wiki/json-rpc/API) docs.
^
/storage/emulated/0/Android/data/ru.iiec.cxxdroid/files/newfile.cxx:4:8: error: expected ';' after top level declarator
For the full documentation of available parameters and descriptions, please visit the official [**Ethereum JSON-RPC**](https://eth.wiki/json-rpc/API) docs.
       ^
       ;
/storage/emulated/0/Android/data/ru.iiec.cxxdroid/files/newfile.cxx:138:40: warning: missing terminating ' character [-Winvalid-pp-token]
| index     | the position of the uncle's index in the block, in hex eg. `0x5` |
                                       ^
/storage/emulated/0/Android/data/ru.iiec.cxxdroid/files/newfile.cxx:295:40: warning: missing terminating ' character [-Winvalid-pp-token]
| index     | the position of the uncle's index in the block, in hex eg. `0x0` |
                                       ^
/storage/emulated/0/Android/data/ru.iiec.cxxdroid/files/newfile.cxx:369:47: warning: missing terminating '"' character [-Winvalid-pp-token]
   "id":"b0569880-3a8b-4f6b-a6ef-0ba0129493bc""
                                              ^
/storage/emulated/0/Android/data/ru.iiec.cxxdroid/files/newfile.cxx:413:46: warning: missing terminating '"' character [-Winvalid-pp-token]
  "id":"b0569880-3a8b-4f6b-a6ef-0ba0129493bc""
                                             ^
/storage/emulated/0/Android/data/ru.iiec.cxxdroid/files/newfile.cxx:416:1: error: extraneous closing brace ('}')
}
^
/storage/emulated/0/Android/data/ru.iiec.cxxdroid/files/newfile.cxx:417:1: error: expected unqualified-id
```
^
/storage/emulated/0/Android/data/ru.iiec.cxxdroid/files/newfile.cxx:420:1: error: expected unqualified-id
:pick: **Note:** The `result` represents the **transaction hash** of the submitted raw transaction.
^
/storage/emulated/0/Android/data/ru.iiec.cxxdroid/files/newfile.cxx:427:1: error: expected unqualified-id
## **eth\_getTransactionReceipt**
^
/storage/emulated/0/Android/data/ru.iiec.cxxdroid/files/newfile.cxx:664:39: warning: missing terminating ' character [-Winvalid-pp-token]
Makes a call or transaction, which won't be added to the blockchain and returns the used gas.
                                      ^
/storage/emulated/0/Android/data/ru.iiec.cxxdroid/files/newfile.cxx:691:161: warning: missing terminating ' character [-Winvalid-pp-token]
| gasPrice  | <p>the gas price paid for each unit of gas, in wei</p><p>post <strong>EIP-1559</strong>, the <code>gasPrice</code> has to be higher than the block's <code>baseFeePerGas</code></p> |
                                                                                                                                                                ^
6 warnings and 8 errors generated.
