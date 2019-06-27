# Description

Contains a docker-compose file to spin up [clair-local-scan](https://github.com/arminc/clair-local-scan) with the latest `clair-db` and a local docker-registry.

This gives you:
- A postgres database with the latest vulnerabililties.
- A clair process which listens on `localhost:6060` which you can query.
- A local docker-registry so you can easily test local images

There's 2 popular tools which you can use to interact with clair.

1. [klar](https://github.com/optiopay/klar)
2. [clair-scanner](https://github.com/arminc/clair-scanner)

I would recommend using `klar` (at least at the time of writing this).

# Usage

## Download Klar

https://github.com/optiopay/klar/releases

```
export KLAR_VERSION=2.4.0

wget "https://github.com/optiopay/klar/releases/download/v${KLAR_VERSION}/klar-${KLAR_VERSION}-linux-amd64"

chmod +x ./klar-${KLAR_VERSION}-linux-amd64

mv ./klar-${KLAR_VERSION}-linux-amd64 ~/bin/klar
```

## Run docker-compose
```
docker-compose up
```

## Scanning public images

```
CLAIR_ADDR=localhost klar python:3.7-slim
```

## Scanning local images

If you're working on images locally, you can push them to the local registry (already) setup by `docker-compose`)

```
docker pull python:3.7-slim
docker tag python:3.7-slim localhost:5000/python:3.7-slim-my-local-image

CLAIR_ADDR=localhost klar localhost:5000/python:3.7-slim:my-local-image
```


## Turn debugging on
```
KLAR_TRACE=1 CLAIR_ADDR=localhost klar python:3.6-slim
```

## JSON output
```
JSON_OUTPUT=1 CLAIR_ADDR=localhost klar python:3.6-slim  | jq .
```

### Show issues that have been fixed
```
JSON_OUTPUT=1 CLAIR_OUTPUT=Unknown CLAIR_ADDR=localhost \
    klar node:6.9.4-alpine \
        | jq -r '.Vulnerabilities[] | .[] | select(.FixedBy != null)'
```

### Show issues for which there's no fix

This is potentially very uesful - if there's no such fix, there's not much you can do except monitor the CVE (so maybe this changes your deployment strategy)
```
JSON_OUTPUT=1 CLAIR_OUTPUT=Unknown CLAIR_ADDR=localhost \
    klar node:6.9.4-alpine \
        | jq -r '.Vulnerabilities[] | .[] | select(.FixedBy == null)'
```

## Ignore low-risk vulnerabilities
```
CLAIR_OUTPUT=High CLAIR_ADDR=localhost klar python:3.6-slim
```