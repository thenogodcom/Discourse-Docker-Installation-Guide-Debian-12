# Discourse 安裝指南 (Debian 12, Root 用戶)

本指南介紹如何在 Debian 12 上使用 **root 用戶** 安裝 Discourse。

**重要提示 (安全風險):**

*   **直接以 root 用戶身分執行所有命令存在安全風險。** 如果 Discourse 或 Docker 存在安全漏洞，攻擊者可能會更容易獲得系統的 root 權限。
*   **強烈建議** 建立一個普通用戶，並使用 `sudo` 來執行需要 root 權限的命令。請參考其他更安全的安裝教程。
*   只有在你 *完全了解* 安全風險，並且有充分的理由這樣做的情況下，才直接以 root 用戶身分執行所有命令。

## 安裝步驟

### 1. 系統準備

*   通過 SSH 連接到你的 VPS，並以 root 用戶身分登錄。
*   更新系統：

    ```bash
    apt update
    apt upgrade -y
    ```

*   安裝 `sudo` (如果未安裝，但後續命令可能會用到)：

    ```bash
    apt install -y sudo
    ```

### 2. 安裝 Docker

*   安裝依賴：

    ```bash
    apt update
    apt install -y apt-transport-https ca-certificates curl gnupg lsb-release
    ```

*   添加 Docker 官方 GPG 金鑰：

    ```bash
    curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    ```

*   設定 Docker 穩定版倉庫：

    ```bash
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```

*   安裝 Docker Engine, containerd, 和 Docker Compose 外掛程式：

    ```bash
    apt update
    apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
    ```

*   驗證 Docker 安裝：

    ```bash
    docker run hello-world
    ```

### 3. 安裝 Git

```bash
apt update
apt install -y git
```

### 4. 配置 Docker 日誌 (可選，但建議)
```bash
nano /etc/docker/daemon.json
```
*   將以下內容複製到文件中：
```bash
{
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "50m",
        "max-file": "5"
    }
}
```
*   儲存並退出 (Ctrl+X, Y, Enter)。

*   重新啟動 Docker：
```bash
systemctl restart docker
```

### 5. 安裝 Discourse

*   建立 Discourse 安裝目錄：
```bash
mkdir -p /var/discourse
cd /var/discourse
```

*   克隆 Discourse Docker 倉庫：
```bash
git clone https://github.com/discourse/discourse_docker.git .
```

*   複製並編輯配置檔案：
```bash
cp samples/standalone.yml containers/app.yml
nano containers/app.yml
```

*   修改 app.yml 配置檔案 (重要)：
```bash
DISCOURSE_HOSTNAME: 你的論壇域名 (例如 forum.example.com)。

DISCOURSE_DEVELOPER_EMAILS: 管理員電子郵件地址 (多個地址用逗號分隔)。

DISCOURSE_SMTP_ADDRESS: SMTP 伺服器地址。

DISCOURSE_SMTP_PORT: SMTP 伺服器端口。

DISCOURSE_SMTP_USER_NAME: SMTP 用戶名。

DISCOURSE_SMTP_PASSWORD: SMTP 密碼。

db_shared_buffers: 根據你的記憶體大小修改，2.5G記憶體，建議設定為683MB.

UNICORN_WORKERS: 根據你的 CPU 核心數修改, 2 核 CPU 建議設定為 2。
```
*  (可選) 添加 自動跳過信箱啟用註冊外掛程式 (例如 discourse-auth-no-email-verification):
```bash
            - git clone https://github.com/thenogodcom/discourse-auth-no-email-verification.git
```

*  儲存並關閉 app.yml 檔案。

* 构建 Discourse 容器：
```bash
./launcher bootstrap app
```

* 啟動 Discourse 容器：
```bash
./launcher start app
```

### 6. 完成 Discourse 設定嚮導

在瀏覽器中訪問你的論壇域名 (你在 app.yml 中設定的 DISCOURSE_HOSTNAME)。

按照 Discourse 設定嚮導的提示完成初始設定。

若選擇添加 自動跳過信箱啟用註冊外掛程式，註冊完成後訪問:
```bash
https://DISCOURSE_HOSTNAME/t/welcome-to-discourse/5
```
再次強調：使用 root 用戶執行所有命令存在安全風險。請謹慎操作。
