# Discourse 安装指南 (Debian 12, Root 用户)

本指南介绍如何在 Debian 12 上使用 **root 用户** 安装 Discourse。

**重要提示 (安全风险):**

*   **直接以 root 用户身份运行所有命令存在安全风险。** 如果 Discourse 或 Docker 存在安全漏洞，攻击者可能会更容易获得系统的 root 权限。
*   **强烈建议** 创建一个普通用户，并使用 `sudo` 来执行需要 root 权限的命令。请参考其他更安全的安装教程。
*   只有在你 *完全了解* 安全风险，并且有充分的理由这样做的情况下，才直接以 root 用户身份运行所有命令。

## 安装步骤

### 1. 系统准备

*   通过 SSH 连接到你的 VPS，并以 root 用户身份登录。
*   更新系统：

    ```bash
    apt update
    apt upgrade -y
    ```

*   安装 `sudo` (如果未安装，但后续命令可能会用到)：

    ```bash
    apt install -y sudo
    ```

### 2. 安装 Docker

*   安装依赖：

    ```bash
    apt update
    apt install -y apt-transport-https ca-certificates curl gnupg lsb-release
    ```

*   添加 Docker 官方 GPG 密钥：

    ```bash
    curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    ```

*   设置 Docker 稳定版仓库：

    ```bash
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```

*   安装 Docker Engine, containerd, 和 Docker Compose 插件：

    ```bash
    apt update
    apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
    ```

*   验证 Docker 安装：

    ```bash
    docker run hello-world
    ```

### 3. 安装 Git

```bash
apt update
apt install -y git
```

### 4. 配置 Docker 日志 (可选，但建议)
```bash
nano /etc/docker/daemon.json
```
*   将以下内容复制到文件中：
```bash
{
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "50m",
        "max-file": "5"
    }
}
```
*   保存并退出 (Ctrl+X, Y, Enter)。

*   重启 Docker：
```bash
systemctl restart docker
```

### 5. 安装 Discourse

*   创建 Discourse 安装目录：
```bash
mkdir -p /var/discourse
cd /var/discourse
```

*   克隆 Discourse Docker 仓库：
```bash
git clone https://github.com/discourse/discourse_docker.git .
```

*   复制并编辑配置文件：
```bash
cp samples/standalone.yml containers/app.yml
nano containers/app.yml
```

*   修改 app.yml 配置文件 (重要)：
```bash
DISCOURSE_HOSTNAME: 你的论坛域名 (例如 forum.example.com)。

DISCOURSE_DEVELOPER_EMAILS: 管理员电子邮件地址 (多个地址用逗号分隔)。

DISCOURSE_SMTP_ADDRESS: SMTP 服务器地址。

DISCOURSE_SMTP_PORT: SMTP 服务器端口。

DISCOURSE_SMTP_USER_NAME: SMTP 用户名。

DISCOURSE_SMTP_PASSWORD: SMTP 密码。

db_shared_buffers: 根据你的内存大小修改，2.5G内存，建议设置为683MB.

UNICORN_WORKERS: 根据你的CPU核心数修改, 2核CPU建议设置为2.
```
*  (可选) 添加 自动跳过邮箱激活注册插件 (例如 discourse-auth-no-email-verification):
```bash
            - git clone https://github.com/thenogodcom/discourse-auth-no-email-verification.git
```

*  保存并关闭 app.yml 文件。

* 构建 Discourse 容器：
```bash
./launcher bootstrap app
```

* 启动 Discourse 容器：
```bash
./launcher start app
```

### 6. 完成 Discourse 设置向导

在浏览器中访问你的论坛域名 (你在 app.yml 中设置的 DISCOURSE_HOSTNAME)。

按照 Discourse 设置向导的提示完成初始设置。

若选择添加 自动跳过邮箱激活注册插件，注册完成后访问:
```bash
https://DISCOURSE_HOSTNAME/t/welcome-to-discourse/5
```
再次强调：使用 root 用户运行所有命令存在安全风险。请谨慎操作。
