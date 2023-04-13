# Local Clickhouse Cluster with Docker Compose

Sets up a local Clickhouse cluster with 3 nodes (with embedded Clickhouse Keeper as a replacement for Zookeeper).

To run it:

```bash
docker compose up -d
```

There is no authentication, just log in with the `default` user with no password at all.

There is a cluster already set up called `localcluster`.

Prometheus metrics is available at `/metrics` endpoint on port `9363` of each node.

To connect with Go:

```go
package main

import (
    "context"

    "github.com/ClickHouse/clickhouse-go/v2"
)

func main() {
    clickhouseOptions, err := clickhouse.ParseDSN("clickhouse://default:@127.0.0.1:19000,127.0.0.1:29000,127.0.0.1:39000/default?dial_timeout=200ms&max_execution_time=60&debug=true")
    if err != nil {
        // handle your error
    }

    clickhouseConnection, err := clickhouse.Open(clickhouseOptions)
    if err != nil {
        // handle your error
    }
    defer func () {
        err := clickhouseConnection.Close()
        if err != nil {
            // handle your error
        }
    }()

    // To query with to the "localcluster"
    err = clickhouseConnection.Exec(
        context.Background(),
        `CREATE DATABASE IF NOT EXISTS "sampledatabase" ON CLUSTER "localcluster"`,
    )
    if err != nil {
        // handle your error
    }
}
```