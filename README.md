# Description

Contains a docker-compose file to spin up [clair-local-scan](https://github.com/arminc/clair-local-scan) with the latest `clair-db`.

This gives you:
- A postgres database with the latest vulnerabililties.
- A clair process which listens on `localhost:6060` which you can query.

There's 2 popular tools which you can use to interact with clair.

1. [klar](https://github.com/optiopay/klar)
2. [clair-scanner](https://github.com/arminc/clair-scanner)

I would recommend using `klar` for now becuase it supports the v3 API from Clair (you can't actully build `clair-scanner` against v3) 

# Usage

## Download Klar

https://github.com/optiopay/klar/releases

```
wget https://github.com/optiopay/klar/releases/download/v2.3.0/klar-2.3.0-linux-amd64
chmod +x ./klar-2.3.0-linux-amd64
mv ./klar-2.3.0-linux-amd64 ./klar
```

## Run docker-compose 
```
docker-compose up
```

## Scan example images

```
CLAIR_ADDR=localhost ./klar python:3.6-slim 
```

## Turn debugging on
```
KLAR_TRACE=1 CLAIR_ADDR=localhost ./klar python:3.6-slim
```

## JSON output
```
JSON_OUTPUT=1 CLAIR_ADDR=localhost ./klar python:3.6-slim  | jq . 
```

### Show issues that have been fixed
```
JSON_OUTPUT=1 CLAIR_OUTPUT=Unknown CLAIR_ADDR=0.0.0.0 \
    klar node:6.9.4-alpine \
        | jq -r '.Vulnerabilities[] | .[] | select(.FixedBy != null)'
```

### Show issues for which there's no fix

This is potentially very uesful - if there's no such fix, there's not much you can do except monitor the CVE (so maybe this changes your deployment strategy)
```
JSON_OUTPUT=1 CLAIR_OUTPUT=Unknown CLAIR_ADDR=0.0.0.0 \
    klar node:6.9.4-alpine \
        | jq -r '.Vulnerabilities[] | .[] | select(.FixedBy == null)'
```

## Ignore low-risk vulnerabilities
```
CLAIR_OUTPUT=High CLAIR_ADDR=localhost ./klar python:3.6-slim
```
