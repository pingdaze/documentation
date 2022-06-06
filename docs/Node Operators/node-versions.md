---
layout: nodes.liquid
section: nodeOperator
date: Last Modified
title: "Node Versions and Upgrades"
permalink: "docs/node-versions/"
whatsnext: {"Running a Chainlink Node":"/docs/running-a-chainlink-node/"}
metadata:
  title: "Node Versions and Release Notes"
  description: "Details about various node versions and how to migrate between them."
---

You can find a list of release notes for Chainlink nodes in the [smartcontractkit GitHub repository](https://github.com/smartcontractkit/chainlink/releases). Docker images are available in the [Chainlink Docker hub](https://hub.docker.com/r/smartcontract/chainlink/tags).

## Changes to v1.5.0

**[v1.5.0 release notes](https://github.com/smartcontractkit/chainlink/releases/tag/v1.5.0)**

- Chainlink will not boot if the Postgres database password is missing or insecure. Passwords must conform to the following rules:
    - Must be longer than 12 characters
    - Must comprise at least 3 of the following items:
        - lowercase characters
        - uppercase characters
        - numbers
        - symbols
    - Must not comprise:
    	  - More than three identical consecutive characters
    	  - Leading or trailing whitespace (note that a trailing newline in the password file, if present, will be ignored)

  For backward compatibility, you can bypass this check at your own risk by setting `SKIP_DATABASE_PASSWORD_COMPLEXITY_CHECK=true`.

- The following ENV variables have been deprecated, replaced by command line arguments, and will be removed in a future release:

  - `INSECURE_SKIP_VERIFY`: Replaced by the `--insecure-skip-verify` CLI argument
  - `CLIENT_NODE_URL`: Replaced by the `--remote-node-url URL` CLI argument
  - `ADMIN_CREDENTIALS_FILE`: Replaced by the `--admin-credentials-file FILE` CLI argument

  Run `./chainlink --help` to learn more about these CLI arguments.

- The `Optimism2` `GAS_ESTIMATOR_MODE` has been renamed to `L2Suggested`. The old name is still supported for now.

- The `p2pBootstrapPeers` property on OCR2 job specs has been renamed to `p2pv2Bootstrappers`.

### Added

- Added `OPERATOR_FACTORY_ADDRESS` config that represents the address of the official LINK token contract on the current chain.

- Added `ETH_USE_FORWARDERS` config option to enable transactions forwarding contracts.

- In the `directrequest` job pipeline, three new block variables are available:
  - `$(jobRun.blockReceiptsRoot)` : the root of the receipts trie of the block (hash)
  - `$(jobRun.blockTransactionsRoot)` : the root of the transaction trie of the block (hash)
  - `$(jobRun.blockStateRoot)` : the root of the final state trie of the block (hash)

- `ethtx` tasks can now be configured to error if the transaction reverts on-chain. You must set `failOnRevert=true` on the task to enable this behavior:

    `foo [type=ethtx failOnRevert=true ...]`

    The `ethtx` task now works as follows:

    - If `minConfirmations == 0`, task always succeeds and nil is passed as output.
    - If `minConfirmations > 0`, the receipt is passed through as output.
    - If `minConfirmations > 0` and `failOnRevert=true` then the `ethtx` task will error on revert.
    - If `minConfirmations` is not set on the task, the chain default will be used which is usually 12 and always greater than 0.

- `http` task now allows specification of request headers. Use it like the following example:

    `foo [type=http headers="[\\"X-Header-1\\", \\"value1\\", \\"X-Header-2\\", \\"value2\\"]"]`.

### Fixed

- Fixed `max_unconfirmed_age` metric. Previously this would incorrectly report the max time since the last rebroadcast, capping the upper limit to the EthResender interval. This now reports the correct value of total time elapsed since the _first_ broadcast.

- Correctly handle the case where bumped gas would exceed the RPC node's configured maximum on Fantom. Note that node operators should check their Fantom RPC node configuration and remove the fee cap if there is one.

### Removed

- The `Optimism` OVM 1.0 `GAS_ESTIMATOR_MODE` has been removed and the `Optimism2` `GAS_ESTIMATOR_MODE` has been renamed to `L2Suggested`.

- `MIN_OUTGOING_CONFIRMATIONS` has been removed and no longer has any effect. The [`ETH_FINALITY_DEPTH` environment variable](/docs/configuration-variables/#eth_finality_depth) is now used as the default for `ethtx` confirmations instead. You can override this on a per-task basis by setting `minConfirmations` in the task definition. For example, `foo [type=ethtx minConfirmations=42 ...]`.

    This setting might have a minor impact on performance for very high throughput chains. If you don't care about reporting task status in the UI, set `minConfirmations=0` in your job specs. For more details, see the [relevant section of the performance tuning guide](https://www.notion.so/chainlink/EVM-performance-configuration-handbook-a36b9f84dcac4569ba68772aa0c1368c#e9998c2f722540b597301a640f53cfd4).

## Changes to v1.4.1

**[v1.4.1 release notes](https://github.com/smartcontractkit/chainlink/releases/tag/v1.4.1)**

- Added a fix to ensure that a failed `EthSubscribe` does not register `(*rpc.ClientSubscription)(nil)`, which leads to a panic when unsubscribing.
- Fix parsing of float values on job specs.

## Changes to v1.4.0

**[v1.4.0 release notes](https://github.com/smartcontractkit/chainlink/releases/tag/v1.4.0)**

- JSON parse tasks in TOML now support a custom `separator` parameter to substitute for the default `,`.
- Slow SQL queries are now logged.
- Updated the block explorer URLs to include FTMScan and SnowTrace.
- Keeper upkeep order can now be shuffled. See [KEEPER_TURN_FLAG_ENABLED](/docs/configuration-variables/#keeper_turn_flag_enabled) for details.
- Several fixes. See the [release notes](https://github.com/smartcontractkit/chainlink/releases/tag/v1.4.0) for a full list of changes.

## Changes to v1.3.0 nodes

**[v1.3.0 release notes](https://github.com/smartcontractkit/chainlink/releases/tag/v1.3.0)**

- Added disk rotating logs. See the [Node Logging](/docs/configuration-variables/#logging) and [LOG_FILE_MAX_SIZE](/docs/configuration-variables/#log_file_max_size) documentation for details.
- Added support for the `force` flag on the `chainlink blocks replay` CLI command. If set to true, already consumed logs that would otherwise be skipped will be rebroadcasted.
- Added a version compatibility check when using the CLI to login to a remote node. The `bypass-version-check` flag skips this check.
- Changed default locking mode to "dual". See the [DATABASE_LOCKING_MODE](/docs/configuration-variables/#database_locking_mode) documentation for details.
- Specifying multiple EVM RPC nodes with the same URL is no longer supported. If you see `ERROR 0106_evm_node_uniqueness.sql: failed to run SQL migration`, you have multiple nodes specified with the same URL and you must fix this before proceeding with the upgrade.
- EIP-1559 is now enabled by default on the Ethereum Mainnet. See the [EVM_EIP1559_DYNAMIC_FEES](/docs/configuration-variables/#evm_eip1559_dynamic_fees) documentation for details.
- Added new Keepers feature that includes gas price in calls to `checkUpkeep()`. To enable the feature, set [KEEPER_CHECK_UPKEEP_GAS_PRICE_FEATURE_ENABLED](/docs/configuration-variables#keeper_check_upkeep_gas_price_feature_enabled) to `true`. Use this setting *only* on Polygon networks.

## Changes to v1.2.0 nodes

**[v1.2.0 release notes](https://github.com/smartcontractkit/chainlink/releases/tag/v1.2.0)**

> 🚧 Not for use on Solana or Terra
>
> Although this release provides `SOLANA_ENABLED` and `TERRA_ENABLED` environment variables, these are not intended for use on Solana or Terra mainnets.

Significant changes:

- Added support for the [Nethermind Ethereum client](https://nethermind.io/).
- Added support for batch sending telemetry to the ingress server to improve performance.
- New environment variables: See the [release notes](https://github.com/smartcontractkit/chainlink/releases/tag/v1.2.0) for details.
- Removed the `deleteuser` CLI command.
- Removed the `LOG_TO_DISK` environment variable.

See the [v1.2.0 release notes](https://github.com/smartcontractkit/chainlink/releases/tag/v1.2.0) for a complete list of changes and fixes.

## Changes to v1.1.0 nodes

**[v1.1.0 release notes](https://github.com/smartcontractkit/chainlink/releases/tag/v1.1.0)**

The v1.1.0 release includes several substantial changes to the way you configure and operate Chainlink nodes:

- **Legacy environment variables**: Legacy environment variables are supported, but they might be removed in future node versions. See the [Configuring Chainlink Nodes](/docs/configuration-variables/#evmethereum-legacy-environment-variables) page to learn how to migrate your nodes away from legacy environment variables and use the API, CLI, or GUI exclusively to administer chains and nodes.
- **Full EIP1559 Support**: Chainlink nodes include experimental support for submitting transactions using type 0x2 (EIP-1559) envelope. EIP-1559 mode is off by default, but can be enabled either globally or on a per-chain basis.
- **New log level added**:
  - [crit]: Critical level logs are more severe than [error] and require quick action from the node operator.
- **Multichain support (Beta)**: Chainlink now supports connecting to multiple different EVM chains simultaneously. This is disabled by default. See the [v1.1.0 Changelog](https://github.com/smartcontractkit/chainlink/blob/v1.1.0/docs/CHANGELOG.md#multichain-support-added) for details.

With multliple chain support, eth node configuration is stored in the database.

The following environment variables are DEPRECATED:

- ETH_URL
- ETH_HTTP_URL
- ETH_SECONDARY_URLS

Setting ETH_URL will cause Chainlink to automatically overwrite the database records with the given ENV values every time Chainlink boots. This behavior is used mainly to ease the process of upgrading from older versions, and on subsequent runs (once your old settings have been written to the database) it is recommended to unset these ENV vars and use the API commands exclusively to administer chains and nodes.

If you wish to continue using these environment variables (as it used to work in 1.0.0 and below) you must ensure that the following are set:

- ETH_CHAIN_ID (mandatory)
- ETH_URL (mandatory)
- ETH_HTTP_URL (optional)
- ETH_SECONDARY_URLS (optional)

If, instead, you wish to use the API/CLI/GUI to configure your chains and eth nodes (recommended) you must REMOVE the following environment variables:

- ETH_URL
- ETH_HTTP_URL
- ETH_SECONDARY_URLS

This will cause Chainlink to use the database for its node configuration.

NOTE: ETH_CHAIN_ID does not need to be removed, since it now performs the additional duty of specifying the default chain in a multichain environment (if you leave ETH_CHAIN_ID unset, the default chain is simply the "first").

For more information on configuring your node, check the [configuration variables in the docs](https://docs.chain.link/docs/configuration-variables/).

Before you upgrade your nodes to v1.1.0, be aware of the following requirements:

- If you are upgrading from a previous version, you **MUST** first upgrade the node to at least [v0.10.15](https://github.com/smartcontractkit/chainlink/releases/tag/v0.10.15).
- Always take a Database snapshot before you upgrade your Chainlink nodes. You must be able to roll the node back to a previous version in the event of an upgrade failure.

See the [v1.1.0 release notes](https://github.com/smartcontractkit/chainlink/releases/tag/v1.1.0) for a complete list of changes and fixes.

## Changes to v1.0.0 and v1.0.1 nodes

**[v1.0.0 release notes](https://github.com/smartcontractkit/chainlink/releases/tag/v1.0.0)**
**[v1.0.1 release notes](https://github.com/smartcontractkit/chainlink/releases/tag/v1.0.1)**

Before you upgrade your nodes to v1.0.0 or v1.0.1, be aware of the following requirements:

- If you are upgrading from a previous version, you **MUST** first upgrade the node to at least [v0.10.15](https://github.com/smartcontractkit/chainlink/releases/tag/v0.10.15).
- Always take a Database snapshot before you upgrade your Chainlink nodes. You must be able to roll the node back to a previous version in the event of an upgrade failure.
