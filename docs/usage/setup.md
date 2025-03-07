# Setup

At this stage of the project, our Walrus code is not yet public. Instead, we provide a pre-compiled
`walrus` client binary for macOS (Intel and Apple CPUs) and Ubuntu, which supports different usage
patterns (see [the next chapter](./interacting.md)). This chapter describes the
[prerequisites](#prerequisites), [installation](#installation), and [configuration](#configuration)
of the Walrus client.

## Prerequisites: Sui wallet and Testnet SUI {#prerequisites}

```admonish tip title="Quick wallet setup"
If you just want to set up a new Sui wallet for Walrus, you can skip this section and use the
`walrus generate-sui-wallet` command after [installing Walrus](#installation). In that case, make
sure to set the `wallet_config` parameter in the [Walrus
configuration](#advanced-configuration-optional) to the newly generated wallet. Also, make sure to
obtain some Testnet SUI tokens from the [Sui Testnet faucet](https://faucet.sui.io/?network=testnet).
```

Interacting with Walrus requires a valid Sui Testnet wallet with some amount of SUI tokens. The
normal way to set this up is via the Sui CLI; see the [installation
instructions](https://docs.sui.io/guides/developer/getting-started/sui-install) in the Sui
documentation.

After installing the Sui CLI, you need to set up a Testnet wallet by running `sui client`, which
prompts you to set up a new configuration. Make sure to point it to Sui Testnet, you can use the
full node at `https://fullnode.testnet.sui.io:443` for this. See
[here](https://docs.sui.io/guides/developer/getting-started/connect) for further details.

If you already have a Sui wallet configured, you can directly set up the Testnet environment (if you
don't have it yet),

```sh
sui client new-env --alias testnet --rpc https://fullnode.testnet.sui.io:443
```

and switch the active environment to it:

```sh
sui client switch --env testnet
```

After this, you should get something like this (everything besides the `testnet` line is optional):

```terminal
$ sui client envs
╭──────────┬─────────────────────────────────────┬────────╮
│ alias    │ url                                 │ active │
├──────────┼─────────────────────────────────────┼────────┤
│ devnet   │ https://fullnode.devnet.sui.io:443  │        │
│ localnet │ http://127.0.0.1:9000               │        │
│ testnet  │ https://fullnode.testnet.sui.io:443 │ *      │
│ mainnet  │ https://fullnode.mainnet.sui.io:443 │        │
╰──────────┴─────────────────────────────────────┴────────╯
```

Finally, make sure you have at least one gas coin with at least 1 SUI. You can obtain one from the
[Sui Testnet faucet](https://faucet.sui.io/?network=testnet) (you can find your address through the
`sui client active-address` command).

After some seconds, you should see your new SUI coins:

```terminal
$ sui client gas
╭─────────────────┬────────────────────┬──────────────────╮
│ gasCoinId       │ mistBalance (MIST) │ suiBalance (SUI) │
├─────────────────┼────────────────────┼──────────────────┤
│ 0x65dca966dc... │ 1000000000         │ 1.00             │
╰─────────────────┴────────────────────┴──────────────────╯
```

The system-wide wallet will be used by Walrus if no other path is specified. If you want to use a
different Sui wallet, you can specify this in the [Walrus configuration file](#configuration) or
when [running the CLI](./interacting.md).

## Installation

We currently provide the `walrus` client binary for macOS (Intel and Apple CPUs), Ubuntu, and
Windows:

| OS      | CPU                   | Architecture                                                                                                                 |
| ------- | --------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| Ubuntu  | Intel 64bit           | [`ubuntu-x86_64`](https://storage.googleapis.com/mysten-walrus-binaries/walrus-testnet-latest-ubuntu-x86_64)                 |
| Ubuntu  | Intel 64bit (generic) | [`ubuntu-x86_64-generic`](https://storage.googleapis.com/mysten-walrus-binaries/walrus-testnet-latest-ubuntu-x86_64-generic) |
| MacOS   | Apple Silicon         | [`macos-arm64`](https://storage.googleapis.com/mysten-walrus-binaries/walrus-testnet-latest-macos-arm64)                     |
| MacOS   | Intel 64bit           | [`macos-x86_64`](https://storage.googleapis.com/mysten-walrus-binaries/walrus-testnet-latest-macos-x86_64)                   |
| Windows | Intel 64bit           | [`windows-x86_64.exe`](https://storage.googleapis.com/mysten-walrus-binaries/walrus-testnet-latest-windows-x86_64.exe)       |

```admonish title="Windows"
We now offer a pre-built binary also for Windows. However, most of the remaining instructions assume
a UNIX-based system for the directory structure, commands, etc. If you use Windows, you may need to
adapt most of those.
```

You can download the latest build from our Google Cloud Storage (GCS) bucket (correctly setting the
`$SYSTEM` variable):

```sh
SYSTEM= # set this to your system: ubuntu-x86_64, ubuntu-x86_64-generic, macos-x86_64, macos-arm64, windows-x86_64.exe
curl https://storage.googleapis.com/mysten-walrus-binaries/walrus-testnet-latest-$SYSTEM -o walrus
chmod +x walrus
```

On Ubuntu, you should generally use the `ubuntu-x86_64` version. However, this is incompatible with
old hardware and certain virtualized environments (throwing an "Illegal instruction (core dumped)"
error); in these cases you can use the `ubuntu-x86_64-generic` version.

To be able to run it simply as `walrus`, move the binary to any directory included in your `$PATH`
environment variable. Standard locations are `/usr/local/bin/`, `$HOME/bin/`, or
`$HOME/.local/bin/`.

```admonish warn
Previously, this guide recommended placing the binary in `$HOME/.local/bin/`. If you install the
latest binary somewhere else, make sure to clean up old versions. You can find the binary in use by
calling `which walrus` and its version through `walrus -V`.
```

Once this is done, you should be able to simply type `walrus` in your terminal. For example you can
get usage instructions (see [the next chapter](./interacting.md) for further details):

```terminal
$ walrus --help
Walrus client

Usage: walrus [OPTIONS] <COMMAND>

Commands:
⋮
```

```admonish tip
Our latest Testnet Walrus binaries are also available on Walrus itself, namely on
<https://bin.walrus.site>, for example, <https://bin.walrus.site/walrus-testnet-latest-ubuntu-x86_64>.
Note that due to DoS protection, it may not be possible to download the binaries with `curl` or
`wget`.
```

### Previous versions (optional)

In addition to the latest version of the `walrus` binary, the GCS bucket also contains previous
versions. An overview in XML format is available at
<https://storage.googleapis.com/mysten-walrus-binaries/>.

## Configuration

The Walrus client needs to know about the Sui objects that store the Walrus system and staking
information, see the [developer guide](../dev-guide/sui-struct.md#system-and-staking-information).
These need to be configured in a file `~/.config/walrus/client_config.yaml`. Additionally, a
`subsidies` object can be specified, which will subsidize storage bought with the client.
Finally, exchange objects are needed to swap SUI for WAL.

The current Testnet deployment uses the following objects:

```yaml
system_object: 0x98ebc47370603fe81d9e15491b2f1443d619d1dab720d586e429ed233e1255c1
staking_object: 0x20266a17b4f1a216727f3eef5772f8d486a9e3b5e319af80a5b75809c035561d
exchange_objects:
  - 0x59ab926eb0d94d0d6d6139f11094ea7861914ad2ecffc7411529c60019133997
  - 0x89127f53890840ab6c52fca96b4a5cf853d7de52318d236807ad733f976eef7b
  - 0x9f9b4f113862e8b1a3591d7955fadd7c52ecc07cf24be9e3492ce56eb8087805
  - 0xb60118f86ecb38ec79e74586f1bb184939640911ee1d63a84138d080632ee28a
subsidies_object: 0x4b23c353c35a4dde72fe862399ebe59423933d3c2c0a3f2733b9f74cb3b4933d
```

<!-- markdownlint-disable code-fence-style -->
~~~admonish tip
The easiest way to obtain the latest configuration is by downloading it from GitHub:

```sh
curl https://raw.githubusercontent.com/MystenLabs/walrus-docs/refs/heads/main/docs/client_config.yaml \
    -o ~/.config/walrus/client_config.yaml
```
~~~
<!-- markdownlint-enable code-fence-style -->

### Custom path (optional) {#config-custom-path}

By default, the Walrus client will look for the `client_config.yaml` (or `client_config.yml`)
configuration file in the current directory, `$XDG_CONFIG_HOME/walrus/`, `~/.config/walrus/`, or
`~/.walrus/`. However, you can place the file anywhere and name it anything you like; in this case
you need to use the `--config` option when running the `walrus` binary.

### Advanced configuration (optional)

The configuration file currently supports the following parameters:

```yaml
# These are the only mandatory fields. These objects are specific for a particular Walrus
# deployment but then do not change over time.
system_object: 0x98ebc47370603fe81d9e15491b2f1443d619d1dab720d586e429ed233e1255c1
staking_object: 0x20266a17b4f1a216727f3eef5772f8d486a9e3b5e319af80a5b75809c035561d

# The exchange objects are used to swap SUI for WAL. If multiple ones are defined (as below), a
# random one is chosen for the exchange.
exchange_objects:
  - 0x59ab926eb0d94d0d6d6139f11094ea7861914ad2ecffc7411529c60019133997
  - 0x89127f53890840ab6c52fca96b4a5cf853d7de52318d236807ad733f976eef7b
  - 0x9f9b4f113862e8b1a3591d7955fadd7c52ecc07cf24be9e3492ce56eb8087805
  - 0xb60118f86ecb38ec79e74586f1bb184939640911ee1d63a84138d080632ee28a

# The subsidies object allows the client to use the subsidies contract to purchase storage
# which will reduce the cost of obtaining a storage resource and extending blobs and also
# adds subsidies to the rewards of the staking pools.
subsidies_object: 0x4b23c353c35a4dde72fe862399ebe59423933d3c2c0a3f2733b9f74cb3b4933d

# You can define a custom path to your Sui wallet configuration here. If this is unset or `null`,
# the wallet is configured from `./sui_config.yaml` (relative to your current working directory), or
# the system-wide wallet at `~/.sui/sui_config/client.yaml` in this order.
wallet_config: null

# The following parameters can be used to tune the networking behavior of the client. There is no
# risk in playing around with these values. In the worst case, you may not be able to store/read
# blob due to timeouts or other networking errors.
communication_config:
  max_concurrent_writes: null
  max_concurrent_sliver_reads: null
  max_concurrent_metadata_reads: 3
  max_concurrent_status_reads: null
  max_data_in_flight: null
  reqwest_config:
    total_timeout_millis: 30000
    pool_idle_timeout_millis: null
    http2_keep_alive_timeout_millis: 5000
    http2_keep_alive_interval_millis: 30000
    http2_keep_alive_while_idle: true
  request_rate_config:
    max_node_connections: 10
    backoff_config:
      min_backoff_millis: 1000
      max_backoff_millis: 30000
      max_retries: 5
  disable_proxy: false
  disable_native_certs: true
  sliver_write_extra_time:
    factor: 0.5
    base_millis: 500
  registration_delay_millis: 200
  max_total_blob_size: 1073741824
```

```admonish warning title="Important"
If you specify a wallet path, make sure your wallet is set up for Sui **Testnet**.
```

## Testnet WAL faucet

The Walrus Testnet uses Testnet WAL tokens to buy storage and stake. Testnet WAL tokens have no
value and can be exchanged (at a 1:1 rate) for some Testnet SUI tokens, which also have no value,
through the following command:

```sh
walrus get-wal
```

You can check that you have received Testnet WAL by checking the Sui balances:

```sh
sui client balance
╭─────────────────────────────────────────╮
│ Balance of coins owned by this address  │
├─────────────────────────────────────────┤
│ ╭─────────────────────────────────────╮ │
│ │ coin  balance (raw)     balance     │ │
│ ├─────────────────────────────────────┤ │
│ │ Sui   8869252670        8.86 SUI    │ │
│ │ WAL   500000000         0.50 WAL    │ │
│ ╰─────────────────────────────────────╯ │
╰─────────────────────────────────────────╯
```

By default, 0.5 SUI are exchanged for 0.5 WAL, but a different amount of SUI may be exchanged using
the `--amount` option (the value is in MIST/FROST), and a specific SUI/WAL exchange object may be
used through the `--exchange-id` option. The `walrus get-wal --help` command provides more
information about those.
