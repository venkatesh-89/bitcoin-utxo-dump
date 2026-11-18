# bitcoin-utxo-dump

Export the unspent transaction outputs from your local Bitcoin Core chainstate database to CSV. Inspect output values, locking scripts, and addresses without parsing LevelDB records yourself.

**Warning:** This tool may corrupt the chainstate database it opens. Stop Bitcoin Core and wait for it to exit, then copy the database and run the tool against the copy. If your live database becomes corrupted, you may need to run `bitcoind -reindex-chainstate`; rebuilding can take around a day.

## Features

- Decode UTXO records directly from Bitcoin Core's chainstate LevelDB database.
- Export transaction IDs, output indexes, and amounts in satoshis to CSV.
- Select output fields, including block height, coinbase status, and locking script details.
- Choose a chainstate directory and an output filename.
- Derive addresses for P2PK outputs with the optional `-p2pkaddresses` flag.

## Installation

You need Go, LevelDB development files, and a synced Bitcoin Core node with a local chainstate database. On Debian or Ubuntu, install the LevelDB dependency with:

```bash
sudo apt install libleveldb-dev
```

From the root of this project's source checkout, build and install the executable:

```bash
go build -o bitcoin-utxo-dump .
mkdir -p "$HOME/.local/bin"
install -m 755 bitcoin-utxo-dump "$HOME/.local/bin/bitcoin-utxo-dump"
```

Make sure `$HOME/.local/bin` is on your `PATH` before running the commands below.

## Usage

Stop Bitcoin Core before copying its database. LevelDB is not designed for concurrent access by multiple programs.

```bash
bitcoin-cli stop
```

Wait until Bitcoin Core has fully exited, then copy the chainstate directory and export from that copy:

```bash
rsync --delete -av ~/.bitcoin/chainstate/ ~/bitcoin-chainstate-clone/
bitcoin-utxo-dump -db ~/bitcoin-chainstate-clone/ -o utxodump.csv
```

The paths above assume the usual Linux Bitcoin Core data directory. Adjust them if your node uses another location. The export reflects the UTXO set in the copied database, not a history of spent outputs.

By default, the tool writes `utxodump.csv` in the current directory with these columns:

```csv
count,txid,vout,amount,type,address
```

The `amount` field is in satoshis. To reduce the output size, select only the fields you need:

```bash
bitcoin-utxo-dump -db ~/bitcoin-chainstate-clone/ -o utxodump.csv -f txid,vout,amount,address
```

Available fields:

| Field | Contents |
| --- | --- |
| `count` | Sequential UTXO count |
| `txid` | Transaction ID containing the output |
| `vout` | Output index within the transaction |
| `height` | Block height where the transaction was mined |
| `coinbase` | Whether the output comes from a coinbase transaction |
| `amount` | Output value in satoshis |
| `script` | Decoded locking-script data; its representation depends on the script type |
| `type` | Locking-script type, such as P2PKH or P2SH |
| `address` | Address associated with the output, where available |

Use `-p2pkaddresses` to derive addresses for P2PK outputs. These scripts contain public keys rather than addresses, so this conversion is disabled by default.

Use `-v` to also display results in the terminal. Verbose output can make the export roughly three times slower. For the full option list:

```bash
bitcoin-utxo-dump -h
```

## License

MIT. This project is based on [in3rsha/bitcoin-utxo-dump](https://github.com/in3rsha/bitcoin-utxo-dump), which is also MIT-licensed. Retain the upstream copyright and permission notice in copies or substantial portions of the software.
