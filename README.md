# rcabench-data

Pre-built, per-category cache shards for the [FSE'26 RCABench](https://arxiv.org/abs/2510.04711) dataset — 1,430 fault-propagation-aware microservice RCA cases on TrainTicket.

## Why this repository exists

The canonical archive (`rcabench-absolute_anomaly.tar.gz`, 13.4 GB, Zenodo record [17105974](https://zenodo.org/records/17105974), CC-BY-4.0) must be downloaded and converted before the benchmark can run. The conversion (Parquet → normalised `case.json`) is expensive and inflates the data ~3.7× (~48.9 GB). This repository hosts the converted output pre-split into 7 fault-category shards, so the benchmark downloads only the categories it needs — no Zenodo download, no conversion, no ~49 GB materialisation.

## Shard format

Each shard is `rcabench-<category>.tar.gz` and archives its cases at `<datapack>/case.json`. Extracting a shard reproduces the flat `<datapack>/case.json` layout that the [micro-kinetic-ts](https://github.com/AgentiX-E/micro-kinetic-ts) FSE'26 runner discovers directly — no re-conversion and no engine change required.

## Fault categories

| Category | Fault types | Count |
|---|---|---|
| Pod | PodKill, PodFailure, ContainerKill | 3 |
| Resource | MemoryStress, CPUStress, JVMCPUStress, JVMMemoryStress | 4 |
| HTTP | HTTPRequestAbort, HTTPResponseAbort, HTTPRequestDelay, HTTPResponseDelay, HTTPResponseReplaceBody, HTTPResponsePatchBody, HTTPRequestReplacePath, HTTPRequestReplaceMethod, HTTPResponseReplaceCode | 9 |
| DNS | DNSError, DNSRandom | 2 |
| Time | TimeSkew | 1 |
| Network | NetworkDelay, NetworkLoss, NetworkDuplicate, NetworkCorrupt, NetworkBandwidth, NetworkPartition | 6 |
| JVM | JVMLatency, JVMReturn, JVMException, JVMGarbageCollector, JVMMySQLLatency, JVMMySQLException | 6 |

## Manifest

`manifest.json` (a Release asset alongside the shards) describes every shard:

```json
{
  "totalCases": 1430,
  "shards": [
    { "category": "HTTP", "cases": 404, "shard": "rcabench-HTTP.tar.gz", "bytes": 123456 }
  ]
}
```

## Consuming the shards

```bash
# download the manifest to learn the available categories
curl -L --fail -o manifest.json \
  "https://github.com/AgentiX-E/rcabench-data/releases/latest/download/manifest.json"

# extract one category (reproduces <datapack>/case.json layout)
curl -L --fail -o rcabench-HTTP.tar.gz \
  "https://github.com/AgentiX-E/rcabench-data/releases/latest/download/rcabench-HTTP.tar.gz"
tar xzf rcabench-HTTP.tar.gz
```

## Build

The shards are produced by `.github/workflows/build-cache.yml`, which clones the converter and sharder from [micro-kinetic-ts](https://github.com/AgentiX-E/micro-kinetic-ts), downloads the archive, converts, shards, and uploads the result to a Release using the repo's own `GITHUB_TOKEN`.

## License

The underlying dataset is CC-BY-4.0 (see the Zenodo record). This repository only re-hosts the converted cache.
