# How to launch Stihi

## Requirements

Cyberway node should be installed. [See documentation](https://github.com/fincubator/cyberway.launch/tree/master/docs).

## Stihi-backend (with remote cyberway node)

Clone repository

```
git clone https://github.com/fincubator/stihi-backend-1.0
cd stihi-backend-1.0
git checkout docker
```

Generate JWT keys. Run inside `stihi-backend-1.0` directory

```
ssh-keygen -t rsa -b 4096 -m PEM -f configs/priv_key.pem
# Don't add passphrase
openssl rsa -in configs/priv_key.pem -pubout -outform PEM -out configs/pub_key.pem
cat configs/priv_key.pem
cat configs/pub_key.pem
```

Prepare configs.

- `configs/db_config.yaml`
```
host: '127.0.0.1'
port: 5432
dbname: 'stihi'
user: 'postgres'
password: 'postgres'
```
- `configs/redis_config.yaml`
```
reconfigure_mode: false
main_servers:
- addr: "127.0.0.1:6379"
  db: 0
  password: ''
  priority: 1.0
```
- `configs/mongo_db_config.yaml`
```
host: 'REMOTE_CYBERWAY_MONGODB_HOST'
port: REMOTE_CYBERWAY_MONGODB_PORT
dbname: '_CYBERWAY_gls_publish'
user: ''
password: ''
```
- `/configs/stihi_backend_config.yaml`

```
cors_origin: "*"
golos:
  creator: cyberfounder
  creator_active_key: "<Active key>"
  creator_posting_key: "<Posting key>"
  create_account_fee:
    amount: 10.000
    symbol: GOLOS
  payments_to: stihi-io
listen:
  host: 0.0.0.0
  port: 9001
jwt:
  private_key_path: /configs/priv_key.pem
  public_key_path: /configs/pub_key.pem
  issuer: test.stihi.io
  expire: 60
  renew_time: 5
l10n_errors:
  - lang: ru
    file_name: /configs/l10n/errors_ru.yaml
  - lang: en
    file_name: /configs/l10n/errors_en.yaml
cyberway:
  host: 'REMOTE_CYBERWAY_NODEOS_HOST'
  port: REMOTE_CYBERWAY_NODEOS_PORT
  uri: v1
  procs_count: 4
```

Create volumes
```
docker volume create stihi-redis
docker volume create stihi-postgres
```

Run shihi-backend

```
docker-compose up -d
```

Check that all containers have run

```
docker ps
```

output

```
CONTAINER ID        IMAGE                                 COMMAND                  CREATED             STATUS         PORTS   NAMES
<id>         stihi-base:latest                            "stihi-backend -db_c…"   2 days ago          Up 2 days              stihi-backend-10_stihi-backend_1
<id>         stihi-base:latest                            "blockchain_loader_c…"   2 days ago          Up 2 days              stihi-backend-10_blockchain-loader-cyberway_1
<id>         stihi-base:latest                            "scan_blockchain_cyb…"   2 days ago          Up 2 days              stihi-backend-10_scan_blockchain_cyberway_1
<id>         postgres:11.5-alpine                         "docker-entrypoint.s…"   2 days ago          Up 2 days              stihi-backend-10_postgres_1
<id>         redis:6.0.9                                  "docker-entrypoint.s…"   2 days ago          Up 2 days              stihi-backend-10_redis_1
...
```

Check status with api call

```
curl http://localhost:9001/api/v2/get_info
```

Output
```
{"info":{"articles":{"count":"-","last_time":"-"},"comments":{"count":"-","last_time":"-"},"follows":{"count":"-"},"last_block":"-","tags":{"count":"-"},"users":{"count":"-"},"votes":{"count":"-","last_time":"-"}},"status":"ok"}
```
