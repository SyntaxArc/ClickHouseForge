# ClickHouseForge
ClickHouseForge: A Docker Compose toolkit for local ClickHouse testing. Forge single-node or 3-node clusters with ZooKeeper, featuring persistent storage, memory limits, and easy customization for robust, scalable setups.

## Directory Structure

```
.
├── single-node/              # Single-node configuration
│   ├── docker-compose.yml    # Single-node Docker Compose
│   └── clickhouse-config/    # Config directory
│       ├── config.xml        # Base configuration
│       ├── users.xml         # User settings
│       └── users.d/          # User config subdirectory
│           └── default-user.xml  # Default user override
├── cluster/                  # Cluster configuration
│   ├── docker-compose.yml    # Cluster Docker Compose
│   └── clickhouse-config/    # Cluster config directory
│       ├── config.xml        # Base configuration
│       ├── users.xml         # User settings
│       └── config.d/         # Cluster config subdirectory
│           └── cluster.xml   # Cluster definition
│       └── users.d/          # User config subdirectory
│           └── default-user.xml  # Default user override
└── README.md                 # This file
```

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) installed
- Free RAM:
  - Single-node: ~2-4GB
  - Cluster: ~8-10GB (2GB per ClickHouse node + ZooKeeper overhead)

## Single-Node Setup

### Features
- Single ClickHouse instance
- Ports: HTTP `8123`, TCP `9000`, Inter-server `9009`
- Persistent storage via volume
- Basic authentication (`admin`/`secretpassword`)

### Usage
1. Navigate to the single-node directory:
```bash
  cd single-node
```

2. Start the service:
```bash
  docker compose up -d
```

3. Verify:
```bash
   docker compose ps
```
Look for `clickhouse-server`.
4. Test connectivity:
   - HTTP: `curl http://localhost:8123` (returns "Ok")
   - Client: `clickhouse-client --host localhost --port 9000 --user admin --password secretpassword`

### Notes
- Add custom configs to `single-node/clickhouse-config/` if needed (e.g., `config.xml`) and uncomment
`./clickhouse-config:/etc/clickhouse-server` in docker compose.

## Cluster Setup

### Features
- Three-node ClickHouse cluster: `clickhouse-1`, `clickhouse-2`, `clickhouse-3`
- ZooKeeper for coordination
- Memory limit: 2GB per ClickHouse node
- Ports:
  - ZooKeeper: `2181`
  - `clickhouse-1`: HTTP `8123`, TCP `9000`
  - `clickhouse-2`: HTTP `8124`, TCP `9001`
  - `clickhouse-3`: HTTP `8125`, TCP `9002`
- Persistent storage for all nodes

### Usage
1. Navigate to the cluster directory:
   ```bash
   cd cluster
   ```
2. Ensure `clickhouse-config/config.d/cluster.xml` exists (see below).
3. Start the cluster:
   ```bash
   docker compose up -d
   ```
4. Verify:
   ```bash
   docker ps
   ```
   Look for `zookeeper`, `clickhouse-1`, `clickhouse-2`, and `clickhouse-3`.
5. Test connectivity:
   - ZooKeeper: `echo ruok | nc localhost 2181` (returns "imok")
   - ClickHouse: `curl http://localhost:8123` (node 1), `8124` (node 2), `8125` (node 3)
   - Client: `clickhouse-client --host localhost --port 9000 --user admin --password secretpassword`
6. Check cluster:
   ```sql
   SELECT * FROM system.clusters;
   ```
   Look for `my_cluster`.

## Stopping and Cleaning Up

- **Stop Services**
  - Single-node: `cd single-node && docker-compose down`
  - Cluster: `cd cluster && docker-compose down`

- **Remove Volumes (Reset Data)**
  - Single-node: `cd single-node && docker-compose down -v`
  - Cluster: `cd cluster && docker-compose down -v`

## Notes

- **Memory**: Adjust `deploy.resources.limits.memory` in `cluster/docker-compose.yml` if needed.
- **Replication**: Cluster setup defines shards but not replication. Use `<replicated_merge_tree>` tables for replication.
- **Security**: Update `secretpassword` in both `docker-compose.yml` files for production-like use.

## Troubleshooting

- **Container Exits**: Check logs with `docker logs <container-name>` (e.g., `clickhouse-1`).
- **Port Conflicts**: Ensure ports are free (e.g., `2181`, `8123-8125`, `9000-9002`).
- **Memory Issues**: Reduce memory limits or node count if RAM is insufficient.

## License

Provided as-is for educational purposes.


