# NushyPool Miner

`npminer` is the NushyPool GPU miner. It is a Rust-based miner with CUDA and OpenCL backends and currently supports these algorithms:

- `memhash` for [Vecno Foundation](https://vecnofoundation.org/)
- `cryptix` for [Cryptix Network](https://cryptix-network.org/)
- `hoohash` for [Hoosat Network](https://network.hoosat.fi/)
- `xelishash` for [Xelis Network](https://xelis.io/)
- `capstash` for [CapStash Chain](https://capstash.org/)

It supports standard **Stratum** pool mining and solo **gRPC/RPC** mining.

## Key Features

- Low CPU usage during normal GPU mining.
- `0%` devfee on supported NushyPool fee-exempt pools.
- Supports solo mining directly against a blockchain node over gRPC/RPC.
- Supports Stratum mining on any compatible pool.
- Supports Linux, Windows, HiveOS, and mmpOS.
- Supports NVIDIA GPUs through CUDA and AMD GPUs through OpenCL.
- Supports multiple algorithms in one miner: `memhash`, `hoohash`, `cryptix`, `xelishash`, and `capstash`.
- Includes autotune with cache reuse to keep tuned launch settings across restarts.
- Supports multi-GPU rigs with simple per-GPU selection via `--devices`.
- Includes watchdog-based worker supervision and recovery-oriented startup checks.
- Optional signed auto-update with `--enable-autoupdate`.
- Local stats API which expose read-only HTTP stats for HiveOS, mmpOS, and XMRig-compatible dashboards.

> [!NOTE]
> **CapStash:** CapStash support is currently experimental. We are evaluating miner interest, and if demand is strong enough, NushyPool may launch a dedicated CapStash pool.

> [!CAUTION]
> - **Drivers:** Always install the latest GPU drivers to achieve optimal hashrate performance.
> - **AMD / OpenCL cards:** Support is experimental — optimal performance is not guaranteed.

## Credits
🙏 Special thanks to @ivchobow, @galindo9856, @MrGodzillaa166, and Mike

## npminer

Main executable:

- Linux (inc. HiveOS/mmpOS): `npminer`
- Windows: `npminer.exe`

Typical pool mining command:

```bash
./npminer -a memhash -o stratum+tcp://POOL:PORT -u vecno:YOUR_WALLET -w worker1
```

Typical solo node mining command (gRPC/RPC):

```bash
./npminer -a hoohash -o grpc://127.0.0.1:42420 -u hoosat:YOUR_WALLET
./npminer -a hoohash --url grpc://127.0.0.1:42420 --user hoosat:YOUR_WALLET
./npminer -a xelishash -o rpc://127.0.0.1:PORT -u xel:YOUR_WALLET
./npminer -a capstash -o rpc://127.0.0.1:8332 -u cap1YOUR_WALLET --rpc-user capstashrpc --rpc-pass YOUR_RPC_PASSWORD
```

CapStash pool mining command:

```bash
./npminer -a capstash -o stratum+tcp://eu.capspool.io:7333 -u cap1YOUR_WALLET -w worker1
```

Useful GPU selection commands:

```bash
./npminer --list-gpus
./npminer -a memhash -o stratum+tcp://POOL:PORT -u cryptix:YOUR_WALLET -d 0,2
```

`-d, --devices` uses the GPU `Index` values printed by `--list-gpus`.

It does not use `Bus ID`.

Example `--list-gpus` output:

```text
========================================
 Index | Bus ID | Name
========================================
 0     | 2      | GeForce RTX 3080 Ti
 1     | 3      | GeForce RTX 3080 Ti
 2     | 7      | Radeon RX 6800 XT
========================================
```

In that example:

- `-d 0` means the first GPU
- `-d 0,2` means the first and third GPUs
- `-d 2` selects the GPU with `Index = 2`, not the GPU with `Bus ID = 2`

More examples:

```bash
./npminer -a hoohash -o grpc://POOL:PORT -u hoosat:YOUR_WALLET -d 1
./npminer -a memhash -o stratum+tcp://POOL:PORT -u vecno:YOUR_WALLET -d 0
./npminer -a memhash -o stratum+tcp://POOL:PORT -u vecno:YOUR_WALLET --devices 0,2,3
./npminer -a xelishash -o rpc://POOL:PORT -u xel:YOUR_WALLET --http-enabled --http-port 42330
./npminer -a capstash -o stratum+tcp://eu.capspool.io:7333 -u cap1YOUR_WALLET -w worker1 --opencl-disable
./npminer -a capstash -o stratum+tcp://eu.capspool.io:7333 -u cap1YOUR_WALLET -w worker1 --cuda-disable --opencl-enable
```

If `--devices` is not provided, all detected GPUs are used.

## Supported Platforms and Hardware

Primary targets:

- NVIDIA GPUs through CUDA
- AMD GPUs through OpenCL

Current backend coverage:

| Backend | Vendor | Algorithms | OS |
|---|---|---|---|
| CUDA | NVIDIA | `memhash`, `hoohash`, `cryptix`, `xelishash`, `capstash` | Windows, Linux, HiveOS, mmpOS |
| OpenCL | AMD | `memhash`, `hoohash`, `cryptix`, `xelishash`, `capstash` | Windows, Linux, HiveOS, mmpOS |

Notes:

- OpenCL is limited to one OpenCL platform per miner process.
- `--opencl-platform` can be used when the default OpenCL platform is not the one you want.
- NVIDIA OpenCL is intentionally not the primary path; NVIDIA cards are expected to use CUDA.
- `--devices` is the recommended end-user way to select GPUs. Use `--cuda-device` and `--opencl-device` only when backend-specific selection is required.


## Help Output

Print short CLI help with:

```bash
./npminer -h
```

Print detailed CLI help with:

```bash
./npminer --help
```

Use `-h` when you only need the option list. Use `--help` when you want the longer descriptions and examples for options such as `--url`, `--user`, `--devices`, and the HTTP API flags.

## Command Reference

The current supported `npminer` command-line options are:

### General

- `-h, --help`
  Print help.
- `-v, --version`
  Print version.
- `-a, --algo <memhash|hoohash|cryptix|xelishash|capstash>`
  Select algorithm explicitly.
- `--mining-address <ADDRESS>`
  Mining reward address.
- `-u, --user <USER>`
  Standard pool-style wallet field, for example `vecno:...` or `vecno:....worker`.
- `--rpc-user <USER>`
  JSON-RPC username for direct-node solo mining, currently used by CapStash RPC.
- `--rpc-pass <PASSWORD>`
  JSON-RPC password for direct-node solo mining, currently used by CapStash RPC.
- `--list-gpus`
  Print the miner GPU inventory and exit.
- `-d, --devices <LIST>`
  Comma-separated GPU `Index` values from `--list-gpus`, not `Bus ID`.
- `--reset-autotune`
  Delete cached autotune state and force a fresh retune.
- `--enable-autoupdate`
  Automatically download, verify, install, and restart into a newer npminer release when one is available.

### Stratum Pool and Solo Node Mining

- `-o, --url <URL>`
  Mining server URL, for example `stratum+tcp://host:port` for Stratum, `grpc://host:port` for Vecno-style solo mining, or `rpc://host:port` for XelisHash/CapStash direct-node solo mining.
- `-w, --worker <WORKER>`
  Standard worker name.
- `-p, --pass <PASSWORD>`
  Standard worker password.

Legacy compatibility Stratum options are also supported:

- `--stratum-server <HOST>`
- `--stratum-port <PORT>`
- `--stratum-worker <WORKER>`
- `--stratum-password <PASSWORD>`

### API Reference
- `--api-port <PORT>`
   Starts the local HTTP stats API on the selected port.
- `--api-bind <ADDR>`
   Bind address for the API listener. The default is localhost.
- `--api-token <TOKEN>`
  Bearer token used to protect API responses. Required for non-local binds.
- `--api-disable`
  Disables the API, including when a mining OS package would normally enable it.
- `--api-worker-id <ID>`
  Overrides the worker ID shown in API responses.
- `--api-id <ID>`
  Overrides the miner instance ID shown in API responses.
- `--http-enabled, --http-host, --http-port, --http-access-token`
  XMRig-compatible aliases for monitoring software that already knows miner HTTP flags.
- `--http-no-restricted`
  Accepted for CLI compatibility. npminer does not expose write or control endpoints in v1.

### CUDA Options

- `--cuda-disable`
  Disable CUDA workers.
- `--cuda-device <LIST>`
  Backend-specific CUDA device list.
- `--cuda-workload <VALUE>`
  CUDA workload ratio.
- `--cuda-workload-absolute`
  Treat CUDA workload as an absolute nonce count.
- `--cuda-no-blocking-sync`
  Disable blocking-sync mode and actively poll.
- `--cuda-nonce-gen <lean|xoshiro>`
  CUDA nonce generator.
- `--cuda-lock-core-clocks <LIST>`
  Optional GPU clock lock values.
- `--cuda-lock-mem-clocks <LIST>`
  Optional GPU memory clock lock values.
- `--cuda-power-limits <LIST>`
  Optional GPU power limits.

### OpenCL Options

- `--opencl-amd-disable`
  Disable OpenCL AMD mining.
- `--opencl-device <LIST>`
  Backend-specific OpenCL device list on the selected platform.
- `--opencl-platform <INDEX>`
  Select one OpenCL platform.
- `--opencl-workload <VALUE>`
  OpenCL workload ratio.
- `--opencl-workload-absolute`
  Treat OpenCL workload as an absolute nonce count.
- `--opencl-nonce-gen <lean|xoshiro>`
  OpenCL nonce generator.
- `--opencl-no-amd-binary`
  Disable fetching/using the precompiled AMD OpenCL binary.

### Notes on Option Behavior

- `--devices` is the recommended end-user selector and maps to the `Index` column from `--list-gpus`.
- `--list-gpus` prints both `Index` and `Bus ID`; `--devices` always uses `Index`.
- Do not combine `--devices` with `--cuda-device` or `--opencl-device`.
- If no device selection flags are provided, all detected GPUs are used.
- `--debug` exists in debug builds only and is not part of the normal release-user CLI surface.

## Auto-Update

npminer checks the official downloads manifest at startup. If a newer version is available and `--enable-autoupdate` is not passed, the miner prints a warning and continues mining with the current version.

Enable automatic updates explicitly:

```bash
./npminer --algo hoohash --url stratum+tcp://POOL:PORT --user hoosat:YOUR_WALLET --worker Hashvilly --enable-autoupdate
```

When enabled, npminer downloads the matching release archive, verifies the archive `sha256`, size, and Ed25519 signature from `downloads.json`, installs the update into the current miner directory, and restarts itself. If the update check, download, verification, or install step fails, npminer keeps running the current version.

HiveOS and mmpOS packages use their own package markers so auto-update selects the matching HiveOS or mmpOS archive instead of the generic Linux archive. The install directory must be writable by the miner process.

## Devfee

| Algorithm | Stratum | Solo gRPC/RPC | NushyPool |
|-----------|--------:|----------:|----------:|
| `memhash` | `1%` | `1%` | `0%` |
| `hoohash` | `1%` | `1%` | `0%` |
| `cryptix` | `1%` | `1%` | `0%` |
| `xelishash` | `1%` | `1%` | `0%` |
| `capstash` | `2%` | `2%` | `n/a` |

> **NushyPool fee-exempt pool users mine with no dev fee.**
>
> CapStash direct RPC node mining uses the solo RPC devfee setting.
> 

## One-Line Installation

Replace `<VERSION>` with the release version you want, for example `0.0.1`.

### Windows

Download and extract the Windows package in one line from PowerShell:

```powershell
$v="<VERSION>"; Invoke-WebRequest "https://github.com/nushypool/npminer/releases/download/v$v/npminer-windows-x86_64-v$v.tar.gz" -OutFile "npminer.tar.gz"; tar -xzf .\npminer.tar.gz
```

### Linux

Download and extract the Linux package in one line:

```bash
VERSION=<VERSION> && curl -L "https://github.com/nushypool/npminer/releases/download/v${VERSION}/npminer-linux-x86_64-v${VERSION}.tar.gz" | tar -xz
```

### Setup for HiveOS
Example HiveOS configurations
| Hoohash | Vecno | Cpay |
|---|---|---|
| <img width="675" height="694" alt="image" src="https://github.com/user-attachments/assets/e0dcad8c-c14a-402f-a5c7-9b03a65a6c7a" /> | <img width="673" height="699" alt="image" src="https://github.com/user-attachments/assets/89a2a8b0-9217-4e5b-be4d-4a88836e1e6e" /> | <img width="679" height="697" alt="image" src="https://github.com/user-attachments/assets/162ab204-4969-4acf-b006-ba5d969fdd5c" /> |


## Notes

- `--list-gpus` is the reference output for end-user GPU selection.
- `--devices` applies the same GPU selection model across enabled backends.
- Do not combine `--devices` with `--cuda-device` or `--opencl-device`.
- If no GPU-selection flags are provided, the miner uses all detected GPUs.
