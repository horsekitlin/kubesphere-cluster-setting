要將 Docker Compose 中的 `volumes` 配置更改為使用本地目錄（例如 `./data`），而不是 Docker 管理的卷，您需要將 `volumes` 的定義修改為指向主機上的目錄。

以下是修改後的 `docker-compose.yml` 文件，將 `postgres` 和 `pgadmin` 的數據持久化目錄更改為本地的 `./data` 目錄：

### 修改後的 `docker-compose.yml`

```yaml
version: '3.5'

services:
  postgres:
    container_name: postgres_container
    image: postgres
    environment:
      POSTGRES_USER: ${POSTGRES_USER:-postgres}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-changeme}
      PGDATA: /data/postgres
    volumes:
      - ./data/postgres:/data/postgres # 這裡更改為本地目錄
    ports:
      - "5432:5432"
    networks:
      - postgres
    restart: unless-stopped
  
  pgadmin:
    container_name: pgadmin_container
    image: dpage/pgadmin4
    environment:
      PGADMIN_DEFAULT_EMAIL: ${PGADMIN_DEFAULT_EMAIL:-pgadmin4@pgadmin.org}
      PGADMIN_DEFAULT_PASSWORD: ${PGADMIN_DEFAULT_PASSWORD:-admin}
      PGADMIN_CONFIG_SERVER_MODE: 'False'
    volumes:
       - ./data/pgadmin:/var/lib/pgadmin # 這裡更改為本地目錄
    ports:
      - "${PGADMIN_PORT:-5050}:80"
    networks:
      - postgres
    restart: unless-stopped

networks:
  postgres:
    driver: bridge

volumes:
    postgres:
    pgadmin:
```

### 主要更改點：
1. **本地目錄映射**: 將 `volumes` 設置改為 `./data/postgres:/data/postgres` 和 `./data/pgadmin:/var/lib/pgadmin`，將 Docker 容器中的數據路徑映射到本地 `./data` 目錄。這樣，當您使用 Docker 容器時，數據將持久化到主機的 `./data` 目錄，而不是使用 Docker 管理的卷。
   
2. **確保目錄存在**: 在運行 Docker Compose 之前，請確保您創建了 `./data` 目錄（即確保路徑 `./data/postgres` 和 `./data/pgadmin` 存在，否則 Docker 可能會自動創建它們，但可能會導致某些權限問題）。

### 使用方式：
在修改 `docker-compose.yml` 文件後，您可以使用以下命令來啟動服務：

```bash
docker-compose up -d
```

這樣，PostgreSQL 和 pgAdmin 將會使用本地硬碟上的 `./data` 目錄來存儲其數據。
