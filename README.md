## Go ddengle

이 문서는 이더소셜 및 이더리움의 소스 분석을 위한 문서입니다.

## 제네시스 블록 만들기


## Building the source

gesc는 go-ethersocial의 줄임말입니다.

gesc를 빌딩하기 위해서는 Golang 1.7 버전 이상이 과 C 컴파일러가 필요합니다.

그 이후 아래 명령어를 입력하면 gesc가 빌딩이 됩니다.

    $ make gesc

gesc 이외에 관련 모든 유틸을 모두 컴파일하시려면 다음을 입력합니다.


    $ make all

## Executables

The go-ethereum project comes with several wrappers/executables found in the `cmd` directory.

| Command    | Description |
|:----------:|-------------|
| **`gesc`** | Our main EtherSocial CLI client. It is the entry point into the EtherSocial network (main-, test- or private net), capable of running as a full node (default) archive node (retaining all historical state) or a light node (retrieving data live). It can be used by other processes as a gateway into the EtherSocial network via JSON RPC endpoints exposed on top of HTTP, WebSocket and/or IPC transports. `gesc --help` and the [CLI Wiki page](https://github.com/ethersocial/go-ddengle/wiki/Command-Line-Options) for command line options. |
| `abigen` | Source code generator to convert EtherSocial contract definitions into easy to use, compile-time type-safe Go packages. It operates on plain [EtherSocial contract ABIs](https://github.com/ethereumsocial/wiki/wiki/EtherSocial-Contract-ABI) with expanded functionality if the contract bytecode is also available. However it also accepts Solidity source files, making development much more streamlined. Please see our [Native DApps](https://github.com/ethersocial/go-ddengle/wiki/Native-DApps:-Go-bindings-to-EtherSocial-contracts) wiki page for details. |
| `bootnode` | Stripped down version of our EtherSocial client implementation that only takes part in the network node discovery protocol, but does not run any of the higher level application protocols. It can be used as a lightweight bootstrap node to aid in finding peers in private networks. |
| `evm` | Developer utility version of the EVM (EtherSocial Virtual Machine) that is capable of running bytecode snippets within a configurable environment and execution mode. Its purpose is to allow isolated, fine-grained debugging of EVM opcodes (e.g. `evm --code 60ff60ff --debug`). |
| `gethrpctest` | Developer utility tool to support our [ethereum/rpc-test](https://github.com/ethereumsocial/rpc-tests) test suite which validates baseline conformity to the [EtherSocial JSON RPC](https://github.com/ethereumsocial/wiki/wiki/JSON-RPC) specs. Please see the [test suite's readme](https://github.com/ethereumsocial/rpc-tests/blob/master/README.md) for details. |
| `rlpdump` | Developer utility tool to convert binary RLP ([Recursive Length Prefix](https://github.com/ethereumsocial/wiki/wiki/RLP)) dumps (data encoding used by the EtherSocial protocol both network as well as consensus wise) to user friendlier hierarchical representation (e.g. `rlpdump --hex CE0183FFFFFFC4C304050583616263`). |
| `swarm`    | swarm daemon and tools. This is the entrypoint for the swarm network. `swarm --help` for command line options and subcommands. See https://swarm-guide.readthedocs.io for swarm documentation. |
| `puppeth`    | a CLI wizard that aids in creating a new EtherSocial network. |

## Running gesc

Going through all the possible command line flags is out of scope here (please consult our
[CLI Wiki page](https://github.com/ethersocial/go-ddengle/wiki/Command-Line-Options)), but we've
enumerated a few common parameter combos to get you up to speed quickly on how you can run your
own Gesc instance.

### Full node on the main EtherSocial network

By far the most common scenario is people wanting to simply interact with the EtherSocial network:
create accounts; transfer funds; deploy and interact with contracts. For this particular use-case
the user doesn't care about years-old historical data, so we can fast-sync quickly to the current
state of the network. To do so:

```
$ gesc --fast --cache=512 console
```

This command will:

 * Start gesc in fast sync mode (`--fast`), causing it to download more data in exchange for avoiding
   processing the entire history of the EtherSocial network, which is very CPU intensive.
 * Bump the memory allowance of the database to 512MB (`--cache=512`), which can help significantly in
   sync times especially for HDD users. This flag is optional and you can set it as high or as low as
   you'd like, though we'd recommend the 512MB - 2GB range.
 * Start up Gesc's built-in interactive [JavaScript console](https://github.com/ethersocial/go-ddengle/wiki/JavaScript-Console),
   (via the trailing `console` subcommand) through which you can invoke all official [`web3` methods](https://github.com/ethereumsocial/wiki/wiki/JavaScript-API)
   as well as Gesc's own [management APIs](https://github.com/ethersocial/go-ddengle/wiki/Management-APIs).
   This too is optional and if you leave it out you can always attach to an already running Gesc instance
   with `gesc attach`.

### Full node on the EtherSocial test network

Transitioning towards developers, if you'd like to play around with creating EtherSocial contracts, you
almost certainly would like to do that without any real money involved until you get the hang of the
entire system. In other words, instead of attaching to the main network, you want to join the **test**
network with your node, which is fully equivalent to the main network, but with play-Ether only.

```
$ gesc --testnet --fast --cache=512 console
```

The `--fast`, `--cache` flags and `console` subcommand have the exact same meaning as above and they
are equally useful on the testnet too. Please see above for their explanations if you've skipped to
here.

Specifying the `--testnet` flag however will reconfigure your Gesc instance a bit:

 * Instead of using the default data directory (`~/.esc` on Linux for example), Gesc will nest
   itself one level deeper into a `testnet` subfolder (`~/.esc/testnet` on Linux). Note, on OSX
   and Linux this also means that attaching to a running testnet node requires the use of a custom
   endpoint since `gesc attach` will try to attach to a production node endpoint by default. E.g.
   `gesc attach <datadir>/testnet/gesc.ipc`. Windows users are not affected by this.
 * Instead of connecting the main EtherSocial network, the client will connect to the test network,
   which uses different P2P bootnodes, different network IDs and genesis states.
   
*Note: Although there are some internal protective measures to prevent transactions from crossing
over between the main network and test network, you should make sure to always use separate accounts
for play-money and real-money. Unless you manually move accounts, Gesc will by default correctly
separate the two networks and will not make any accounts available between them.*

### Configuration

As an alternative to passing the numerous flags to the `gesc` binary, you can also pass a configuration file via:

```
$ gesc --config /path/to/your_config.toml
```

To get an idea how the file should look like you can use the `dumpconfig` subcommand to export your existing configuration:

```
$ gesc --your-favourite-flags dumpconfig
```

*Note: This works only with gesc v1.6.0 and above.*

#### Docker quick start

One of the quickest ways to get EtherSocial up and running on your machine is by using Docker:

```
docker run -d --name ethereumsocial-node -v /Users/alice/ethereumsocial:/root \
           -p 9545:9545 -p 50505:50505 \
           ethereum/client-go --fast --cache=512
```

This will start gesc in fast sync mode with a DB memory allowance of 512MB just as the above command does.  It will also create a persistent volume in your home directory for saving your blockchain as well as map the default ports. There is also an `alpine` tag available for a slim version of the image.

Do not forget `--rpcaddr 0.0.0.0`, if you want to access RPC from other containers and/or hosts. By default, `gesc` binds to the local interface and RPC endpoints is not accessible from the outside.

### Programatically interfacing Gesc nodes

As a developer, sooner rather than later you'll want to start interacting with Gesc and the EtherSocial
network via your own programs and not manually through the console. To aid this, Gesc has built in
support for a JSON-RPC based APIs ([standard APIs](https://github.com/ethereumsocial/wiki/wiki/JSON-RPC) and
[Gesc specific APIs](https://github.com/ethersocial/go-ddengle/wiki/Management-APIs)). These can be
exposed via HTTP, WebSockets and IPC (unix sockets on unix based platforms, and named pipes on Windows).

The IPC interface is enabled by default and exposes all the APIs supported by Gesc, whereas the HTTP
and WS interfaces need to manually be enabled and only expose a subset of APIs due to security reasons.
These can be turned on/off and configured as you'd expect.

HTTP based JSON-RPC API options:

  * `--rpc` Enable the HTTP-RPC server
  * `--rpcaddr` HTTP-RPC server listening interface (default: "localhost")
  * `--rpcport` HTTP-RPC server listening port (default: 9545)
  * `--rpcapi` API's offered over the HTTP-RPC interface (default: "eth,net,web3")
  * `--rpccorsdomain` Comma separated list of domains from which to accept cross origin requests (browser enforced)
  * `--ws` Enable the WS-RPC server
  * `--wsaddr` WS-RPC server listening interface (default: "localhost")
  * `--wsport` WS-RPC server listening port (default: 9546)
  * `--wsapi` API's offered over the WS-RPC interface (default: "eth,net,web3")
  * `--wsorigins` Origins from which to accept websockets requests
  * `--ipcdisable` Disable the IPC-RPC server
  * `--ipcapi` API's offered over the IPC-RPC interface (default: "admin,debug,eth,miner,net,personal,shh,txpool,web3")
  * `--ipcpath` Filename for IPC socket/pipe within the datadir (explicit paths escape it)

You'll need to use your own programming environments' capabilities (libraries, tools, etc) to connect
via HTTP, WS or IPC to a Gesc node configured with the above flags and you'll need to speak [JSON-RPC](http://www.jsonrpc.org/specification)
on all transports. You can reuse the same connection for multiple requests!

**Note: Please understand the security implications of opening up an HTTP/WS based transport before
doing so! Hackers on the internet are actively trying to subvert EtherSocial nodes with exposed APIs!
Further, all browser tabs can access locally running webservers, so malicious webpages could try to
subvert locally available APIs!**

###  사설네트워크 생성 방법

사설 네트워크를 생성하기 위해서는 공식네트워크보다 많은 설정을 필요로 합니다

#### 사설네트워크 생성 정보 정의하기

첫번째로는 당신의 네트워크에 생성 정보를 정의하고, 모든 노드가 생성 정보를 공유해야합니다. 
아래는 생성정보가 정의된 JSON 파일입니다.(e.g. call it `genesis.json`):

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

With the genesis state defined in the above JSON file, you'll need to initialize **every** Gesc node
with it prior to starting it up to ensure all blockchain parameters are correctly set:

```
$ gesc init path/to/genesis.json
```

#### Creating the rendezvous point

With all nodes that you want to run initialized to the desired genesis state, you'll need to start a
bootstrap node that others can use to find each other in your network and/or over the internet. The
clean way is to configure and run a dedicated bootnode:

```
$ bootnode --genkey=boot.key
$ bootnode --nodekey=boot.key
```

With the bootnode online, it will display an [`enode` URL](https://github.com/ethereumsocial/wiki/wiki/enode-url-format)
that other nodes can use to connect to it and exchange peer information. Make sure to replace the
displayed IP address information (most probably `[::]`) with your externally accessible IP to get the
actual `enode` URL.

*Note: You could also use a full fledged Gesc node as a bootnode, but it's the less recommended way.*

#### Starting up your member nodes

With the bootnode operational and externally reachable (you can try `telnet <ip> <port>` to ensure
it's indeed reachable), start every subsequent Gesc node pointed to the bootnode for peer discovery
via the `--bootnodes` flag. It will probably also be desirable to keep the data directory of your
private network separated, so do also specify a custom `--datadir` flag.

```
$ gesc --datadir=path/to/custom/data/folder --bootnodes=<bootnode-enode-url-from-above>
```

*Note: Since your network will be completely cut off from the main and test networks, you'll also
need to configure a miner to process transactions and create new blocks for you.*

#### Running a private miner

Mining on the public EtherSocial network is a complex task as it's only feasible using GPUs, requiring
an OpenCL or CUDA enabled `ethminer` instance. For information on such a setup, please consult the
[EtherMining subreddit](https://www.reddit.com/r/EtherMining/) and the [Genoil miner](https://github.com/Genoil/cpp-ethereum)
repository.

In a private network setting however, a single CPU miner instance is more than enough for practical
purposes as it can produce a stable stream of blocks at the correct intervals without needing heavy
resources (consider running on a single thread, no need for multiple ones either). To start a Gesc
instance for mining, run it with all your usual flags, extended by:

```
$ gesc <usual-flags> --mine --minerthreads=1 --etherbase=0x0000000000000000000000000000000000000000
```

Which will start mining blocks and transactions on a single CPU thread, crediting all proceedings to
the account specified by `--etherbase`. You can further tune the mining by changing the default gas
limit blocks converge to (`--targetgaslimit`) and the price transactions are accepted at (`--gasprice`).

## Contribution

Thank you for considering to help out with the source code! We welcome contributions from
anyone on the internet, and are grateful for even the smallest of fixes!

If you'd like to contribute to go-ethereum, please fork, fix, commit and send a pull request
for the maintainers to review and merge into the main code base. If you wish to submit more
complex changes though, please check up with the core devs first on [our gitter channel](https://gitter.im/ethereumsocial/go-ethereum)
to ensure those changes are in line with the general philosophy of the project and/or get some
early feedback which can make both your efforts much lighter as well as our review and merge
procedures quick and simple.

Please make sure your contributions adhere to our coding guidelines:

 * Code must adhere to the official Go [formatting](https://golang.org/doc/effective_go.html#formatting) guidelines (i.e. uses [gofmt](https://golang.org/cmd/gofmt/)).
 * Code must be documented adhering to the official Go [commentary](https://golang.org/doc/effective_go.html#commentary) guidelines.
 * Pull requests need to be based on and opened against the `master` branch.
 * Commit messages should be prefixed with the package(s) they modify.
   * E.g. "eth, rpc: make trace configs optional"

Please see the [Developers' Guide](https://github.com/ethersocial/go-ddengle/wiki/Developers'-Guide)
for more details on configuring your environment, managing project dependencies and testing procedures.

## License

The go-ethereum library (i.e. all code outside of the `cmd` directory) is licensed under the
[GNU Lesser General Public License v3.0](https://www.gnu.org/licenses/lgpl-3.0.en.html), also
included in our repository in the `COPYING.LESSER` file.

The go-ethereum binaries (i.e. all code inside of the `cmd` directory) is licensed under the
[GNU General Public License v3.0](https://www.gnu.org/licenses/gpl-3.0.en.html), also included
in our repository in the `COPYING` file.
