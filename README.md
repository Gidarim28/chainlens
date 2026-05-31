# chainlens

A lightweight, dependency-light **on-chain data analysis toolkit** for EVM chains
(Ethereum, Optimism, Base, Arbitrum, Polygon, and any other JSON-RPC endpoint).

`chainlens` lets you pull balances, blocks, gas stats, ERC-20 metadata, and
`Transfer` event flows straight into [pandas](https://pandas.pydata.org/) for
analysis — from a clean Python API or the command line. It depends only on
`requests` and `pandas`: no `web3`, no compiled extensions, no node required.

## Why

Most on-chain tooling is either a heavyweight framework or a hosted API behind a
paywall. `chainlens` is a small, readable building block: point it at any public
RPC and start analysing. The ABI codec and a pure-Python Keccak-256 (for EIP-55
checksums) are implemented in-repo, so there are no surprising transitive
dependencies.

## Install

```bash
pip install chainlens          # from PyPI (once published)
# or, from source:
git clone https://github.com/Gidarim28/chainlens
cd chainlens && pip install -e .
```

Requires Python 3.9+.

## Quick start (library)

```python
from chainlens import EvmClient, ERC20, analysis

client = EvmClient("https://eth.llamarpc.com")

# native balance (vitalik.eth)
wei = client.get_balance("0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045")
print(client.block_number(), wei)

# ERC-20 metadata + balance (USDC)
usdc = ERC20(client, "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48")
print(usdc.info())                       # TokenInfo(symbol='USDC', decimals=6, ...)
print(usdc.balance_of_units("0xd8dA..."))

# analyse recent Transfer events into a DataFrame
latest = client.block_number()
logs = client.get_logs(
    address=usdc.address,
    from_block=latest - 5,
    to_block=latest,
    topics=[analysis.utils.TRANSFER_TOPIC],
)
df = analysis.transfers_to_dataframe(logs, decimals=6)
print(analysis.summarize(df))
print(analysis.top_holders_by_volume(df, n=10))
```

## Quick start (CLI)

```bash
# gas price + latest-block utilisation
chainlens gas --rpc https://eth.llamarpc.com

# gas stats for a specific block
chainlens block 19000000 --rpc https://eth.llamarpc.com

# native balance
chainlens balance 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045

# ERC-20 metadata and a holder balance
chainlens token 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48 \
    --holder 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045

# decode + rank recent Transfer events for a token
chainlens transfers 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48 --blocks 5
```

The default RPC is a public Ethereum endpoint; override it with `--rpc` to point
at any chain.

## What's inside

| Module | Purpose |
| --- | --- |
| `chainlens.client` | `EvmClient` — JSON-RPC wrapper (blocks, balances, logs, `eth_call`) |
| `chainlens.tokens` | `ERC20` — symbol / decimals / balanceOf helpers |
| `chainlens.analysis` | pandas DataFrames: transfer decoding, net flows, gas stats |
| `chainlens.utils` | unit conversion, EIP-55 checksums, minimal ABI codec |

The analysis layer is **pure** — every function takes already-fetched data and
returns a DataFrame or dict, so it is fully unit-testable offline.

## Development

```bash
pip install -e ".[dev]"
pytest -q
```

## Roadmap

- Multicall batching for faster balance snapshots
- Optional matplotlib plotting helpers
- Block-range streaming with automatic chunking for large `eth_getLogs` queries
- Built-in adapters for popular L2s (Base, Optimism, Arbitrum)

Contributions welcome — see the issues tab.

## License

MIT — see [LICENSE](LICENSE).
