# r2upload

`r2upload` 是一个适用于 Linux (Rocky / Ubuntu / Debian 等) 的命令行工具，用于将文件或目录上传到 [Cloudflare R2](https://developers.cloudflare.com/r2/)。  
它基于 **AWS CLI (S3 API 兼容)**，支持：

- ✅ 通过配置文件 (`~/.r2upload.conf`) 保存默认 Bucket 和 Endpoint（像 AWS CLI 一样免参数）
- ✅ 压缩 (`-z`) / 分卷 (`-S`) 上传
- ✅ AWS CLI 实时上传进度
- ✅ 自动重试
- ✅ 自动修正 Endpoint 格式
- ✅ 日志记录

---

## 🚀 一键安装
```bash
sudo curl -fsSL https://raw.githubusercontent.com/a23506/r2upload/main/r2upload -o /usr/local/bin/r2upload
sudo chmod +x /usr/local/bin/r2upload
```

---

## 📦 安装 AWS CLI（必需）
```bash
curl -fsSLO "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip"
unzip -q awscliv2.zip
sudo ./aws/install
aws --version
```

---

## 🔑 准备 Cloudflare R2 访问凭据

1. 登录 Cloudflare 控制台 → R2 → **Manage R2 API Tokens**  
2. 创建一个 S3 API 类型的访问密钥（包含 Access Key ID / Secret Access Key）  
3. 记下 **Account ID**，构造 Endpoint：  
   ```
   https://<accountid>.r2.cloudflarestorage.com
   ```
   **注意：不要在 endpoint 中加 bucket 名**（例如 `/file` 是错的）

4. 在服务器配置 AK/SK：
```bash
aws configure
# AWS Access Key ID     -> 你的 R2 Access Key
# AWS Secret Access Key -> 你的 R2 Secret
# Default region name   -> auto
# Default output format -> 回车
```

---

## ⚙️ 配置文件（推荐）

路径：
```
~/.r2upload.conf
```

内容示例：
```bash
R2_BUCKET="file"
R2_ENDPOINT="https://<accountid>.r2.cloudflarestorage.com"
# 可选统一前缀（会加在所有对象前面）
# R2_PREFIX="backup/"
```

这样就不必每次写 `-b` 和 `-e`。

---

## 🛠 用法
```bash
r2upload [选项] <文件或目录...>
```
- 无参数 → 显示帮助和示例
- 缺少必填参数 → 提示错误并显示用法
- 支持多路径同时上传

---

## 📋 选项说明

| 选项 | 含义 | 说明 |
|------|------|------|
| `-b, --bucket` | 存储桶名 | 可在配置文件中预设 |
| `-e, --endpoint` | Endpoint | 形如 `https://<accountid>.r2.cloudflarestorage.com` |
| `-p, --prefix` | 对象 Key 前缀 | 类似虚拟目录 |
| `-z, --compress` | 压缩目录 | 打包为 `.tar.gz` |
| `-S, --split` | 分卷大小 | 仅在压缩时有效（如 `1G` / `500M`） |
| `-t, --threads` | 并行度 | 默认 `4` |
| `-r, --retries` | 重试次数 | 默认 `3` |
| `-l, --log` | 日志路径 | 默认 `~/r2upload.log` |
| `-K, --keep-archive` | 保留压缩包 | 默认上传后删除 |
| `-h, --help` | 帮助 | 显示用法与示例 |

---

## 📚 示例

### 上传文件
```bash
r2upload kejilion.sh
```

### 上传目录
```bash
r2upload /opt/data
```

### 压缩目录上传
```bash
r2upload -z /var/log
```

### 压缩并按 1GB 分卷
```bash
r2upload -z -S 1G /srv/bigdir
```

### 临时覆盖配置文件
```bash
r2upload -b newbucket -e https://<accountid>.r2.cloudflarestorage.com file.txt
```

---

## 📜 日志 & 进度

- **进度**：AWS CLI 自带百分比、速率、剩余时间
- **日志**：默认保存在 `~/r2upload.log`，包含时间、Key、进度等
- 查看日志：
```bash
tail -f ~/r2upload.log
```

---

## ⏱ 自动化（cron / systemd）

### cron 示例（每天 03:00 压缩备份 /data）
```bash
0 3 * * * /usr/local/bin/r2upload -z -S 1G /data >> $HOME/r2upload.cron.log 2>&1
```

### systemd 示例
`/etc/systemd/system/r2upload-backup.service`
```ini
[Unit]
Description=Daily R2 Backup

[Service]
Type=oneshot
User=root
ExecStart=/usr/local/bin/r2upload -z -S 1G /data
```
`/etc/systemd/system/r2upload-backup.timer`
```ini
[Unit]
Description=Run R2 Backup daily at 03:00

[Timer]
OnCalendar=*-*-* 03:00:00
Persistent=true
Unit=r2upload-backup.service

[Install]
WantedBy=timers.target
```
启用：
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now r2upload-backup.timer
```

---

## ❓ FAQ

**Q:** 运行时报 *缺少 bucket/endpoint*  
**A:** 写好 `~/.r2upload.conf` 即可免参数。

**Q:** 400 Bad Request  
**A:** Endpoint 不要包含 bucket 名。

**Q:** 上传很慢  
**A:** 对小文件多的目录使用 `-z` 压缩再传。

---

## 🔒 安全建议
- 不要把 AK/SK 放进 GitHub
- 配置文件和日志建议：
```bash
chmod 600 ~/.r2upload.conf
chmod 600 ~/r2upload.log 2>/dev/null || true
```

---

## 🤝 开发与贡献
欢迎提 Issue / PR。

---

## 📄 License
MIT License - 详见 [LICENSE](LICENSE)
