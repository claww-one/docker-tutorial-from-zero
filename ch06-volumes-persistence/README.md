# 第 6 章：Docker 卷期與資料持久化

## 觀念講解 (Concepts)

### 1. 為什麼需要持久化？
容器本身是「臨時的 (Ephemeral)」，當容器被刪除時，寫入層的所有資料也會隨之消失。
為了保存資料 (如資料庫文件、使用者上傳檔案)，我們需要將容器外部的儲存掛載 (Mount) 到容器內部。

### 2. 持久化的三種方式
- **Volumes (Docker 卷期)**：由 Docker 管理，儲存在主機檔案系統的特定區域 (通常是 /var/lib/docker/volumes/)。**強烈建議在生產環境中使用**。

```mermaid
graph LR
    subgraph Host[Host Machine]
        V_Store[/var/lib/docker/volumes/]
        B_Store[/home/user/my-app/]
    end
    subgraph Container[Container]
        C_Path_1[/app/data]
        C_Path_2[/app/source]
    end
    V_Store -- "Managed by Docker" --> C_Path_1
    B_Store -- "Manual Bind" --> C_Path_2
```

#### 掛載連結說明 (Mount Link Meanings)
*   **V_Store → C_Path_1 (Docker Volumes)**：**全託管映射**。連結代表 Docker 引擎完全控制該目錄的生命週期與權限，適用於資料庫等對 I/O 與穩定性要求高的場景。
*   **B_Store → C_Path_2 (Bind Mounts)**：**路徑直連**。連結代表主機檔案系統與容器內部的目錄「鏡像對齊」，任何一端的修改都會即時同步，最適合開發階段的程式碼熱加載 (Hot Reloading)。

- **Bind Mounts (綁定掛載)**：將主機上的「任意」路徑掛載到容器。常用於開發環境，因為可以在主機直接編輯程式碼，並即時反應在容器中。
- **tmpfs Mounts (記憶體掛載)**：將資料儲存在主機的記憶體中，讀取速度最快但重新開機後會消失。

### 3. Volumes 的優點
- 跨平台移植性更好。
- 可由 Docker CLI 直接操作與管理。
- 支援卷期驅動程式 (Volume Drivers)，可以掛載雲端儲存 (如 AWS S3, NFS)。

---

## 實作演練 (Implementation)

### 1. 使用 Docker Volumes (管理卷期)
這是最安全且最推薦的方式：

```bash
# 1. 建立一個具名的卷期
docker volume create db-data

# 2. 啟動資料庫容器並掛載此卷期 (-v <volume_name>:<container_path>)
docker run -d --name mysql-db -e MYSQL_ROOT_PASSWORD=password -v db-data:/var/lib/mysql mysql:8.0

# 3. 測試持久化：刪除容器並重新建立
docker rm -f mysql-db
docker run -d --name mysql-db-new -e MYSQL_ROOT_PASSWORD=password -v db-data:/var/lib/mysql mysql:8.0
# 預期結果：進入新容器後，原本資料庫內的資料依然存在。
```

### 2. 使用 Bind Mounts (開發同步)
這種方式方便在主機編輯檔案：

```bash
# 將當前目錄 (.) 下的 html 資料夾掛載到 nginx 預設目錄
docker run -d --name dev-nginx -p 8080:80 -v /home/node/.openclaw/workspace/html:/usr/share/nginx/html nginx
# 此時你修改主機上的 html/index.html，重新整理瀏覽器就會看到變化。
```

### 3. 管理與清理卷期

```bash
# 列出所有卷期
docker volume ls

# 查看卷期在主機的真實路徑
docker volume inspect db-data

# 移除未使用的卷期
docker volume prune

# 刪除特定卷期 (需先停止掛載該卷期的所有容器)
docker volume rm db-data
```

---
*Last updated: 2026-03-13 by SiaSia 🦞*
