---
title: curl 工具
date: 2025-02-08T17:26:06+08:00
updated: 2025-02-08T17:30:33+08:00
dg-publish: false
---

`curl`（Client URL）是一个功能强大的命令行工具，用于通过 URL 传输数据，支持 HTTP、HTTPS、FTP、FTPS、SCP、SFTP 等多种协议。以下是 `curl` 的详解和常见用法：

---

## **一、基础用法**

### 1. **下载文件**

```bash
curl -O http://example.com/file.zip  # 下载文件并保留原文件名
curl -o custom_name.zip http://example.com/file.zip  # 指定保存的文件名
```

### 2. **发送 GET 请求**

```bash
curl http://example.com/api/data  # 直接请求URL
curl "http://example.com/api/data?param1=value1&param2=value2"  # 带查询参数
```

### 3. **发送 POST 请求**

```bash
curl -X POST -d "key1=value1&key2=value2" http://example.com/api  # 表单数据
curl -X POST -H "Content-Type: application/json" -d '{"key":"value"}' http://example.com/api  # JSON数据
```

---

## **二、常用选项**

| 选项 | 描述 |
|------|------|
| `-o <文件名>` | 将输出保存到指定文件 |
| `-O` | 使用远程文件的名称保存 |
| `-L` | 自动跟随重定向（如 HTTP 301/302） |
| `-v` | 显示详细请求和响应信息（调试用） |
| `-s` | 静默模式（不显示进度和错误信息） |
| `-H` | 添加请求头（如 `-H "Authorization: Bearer token"`） |
| `-d` | 发送 POST 请求的数据（默认 Content-Type: `application/x-www-form-urlencoded`） |
| `-F` | 上传文件（表单格式，如 `-F "file=@/path/to/file"`） |
| `-u` | 用户认证（如 `-u username:password`） |
| `-k` | 忽略 SSL 证书验证（不推荐生产环境使用） |
| `-C -` | 断点续传（继续未完成的下载） |

---

## **三、高级功能**

### 1. **设置请求头**

```bash
curl -H "User-Agent: MyApp/1.0" -H "Accept: application/json" http://example.com
```

### 2. **处理 Cookie**

```bash
curl -c cookies.txt http://example.com/login  # 保存Cookie到文件
curl -b cookies.txt http://example.com/dashboard  # 使用Cookie发送请求
```

### 3. **代理设置**

```bash
curl -x http://proxy-server:port http://example.com  # 使用HTTP代理
curl --socks5 proxy-server:port http://example.com   # SOCKS5代理
```

### 4. **调试请求**

```bash
curl -v http://example.com  # 显示详细请求/响应头
curl --trace-ascii debug.txt http://example.com  # 将调试信息保存到文件
```

### 5. **限速下载**

```bash
curl --limit-rate 100K -O http://example.com/largefile.zip  # 限制下载速度（100KB/s）
```

---

## **四、安全相关**

### 1. **HTTPS 请求**

```bash
curl https://example.com  # 默认验证SSL证书
curl --cert client.pem --key key.pem https://example.com  # 使用客户端证书
```

### 2. **忽略证书验证（慎用）**

```bash
curl -k https://example.com  # 跳过SSL证书验证
```

### 3. **基本认证**

```bash
curl -u username:password http://example.com/protected
```

---

## **五、实用示例**

1. **下载多个文件**

```bash
curl -O http://example.com/file1.zip -O http://example.com/file2.zip
```

1. **上传文件（表单）**

```bash
curl -F "file=@/path/to/file.txt" -F "name=myfile" http://example.com/upload
```

1. **模拟浏览器请求**

```bash
curl -A "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36" http://example.com
```

1. **保存响应头到文件**

```bash
curl -D headers.txt -o content.html http://example.com
```

---

## **一、其他高级用法**

### 1. **处理 HTTP 状态码**

```bash
curl -o /dev/null -s -w "%{http_code}\n" http://example.com  # 仅输出HTTP状态码
```

### 2. **多线程下载（分块并发）**

```bash
curl --parallel --parallel-max 4 -O http://example.com/file1 -O http://example.com/file2
```

### 3. **处理 FTP 操作**

```bash
curl -u username:password ftp://example.com/file.txt  # 下载FTP文件
curl -T localfile.txt ftp://example.com/upload/        # 上传文件到FTP
```

### 4. **显示进度条（大文件下载）**

```bash
curl -# -O http://example.com/largefile.zip  # 用`-#`显示进度条
```

### 5. **自定义输出格式**

```bash
curl -w "Time: %{time_total}s\nSize: %{size_download} bytes\n" http://example.com
```

### 6. **处理 JSON 响应**

```bash
curl -s http://example.com/api | jq .  # 结合`jq`工具解析JSON（需安装jq）
```

### 7. **压缩传输（节省带宽）**

```bash
curl --compressed http://example.com  # 请求服务器启用gzip压缩
```

### 8. **超时控制**

```bash
curl --max-time 10 http://example.com  # 10秒超时
curl --connect-timeout 5 http://example.com  # 连接阶段5秒超时
```

### 9. **IPv4/IPv6 指定**

```bash
curl -4 http://example.com  # 强制使用IPv4
curl -6 http://example.com  # 强制使用IPv6
```

---

## **二、注意事项**

### 1. **敏感信息泄露**

- **避免在命令中明文显示密码**：

	```bash
  # 错误示例（密码会出现在历史记录中）：
  curl -u user:password http://example.com

  # 正确做法（从文件或环境变量读取）：
  curl -u user:$(cat password.txt) http://example.com
  ```

### 2. **重定向次数限制**

```bash
curl -L --max-redirs 5 http://example.com  # 限制最多5次重定向
```

### 3. **SSL/TLS 版本控制**

```bash
curl --tlsv1.2 https://example.com  # 强制使用TLS 1.2
curl --tlsv1.3 https://example.com  # 强制使用TLS 1.3
```

### 4. **文件覆盖风险**

- 使用 `-O` 或 `-o` 时，若本地已有同名文件会直接覆盖，建议先检查文件是否存在。

### 5. **特殊字符转义**

- 如果 URL 或数据包含特殊字符（如空格、`&`、`?`），需用引号包裹：

	```bash
  curl "http://example.com?name=John Doe&age=30"
  ```

### 6. **性能优化**

- 复用 TCP 连接（HTTP Keep-Alive）：

	```bash
  curl --http1.1 --keepalive-time 30 http://example.com
  ```

### 7. **跨平台差异**

- Windows 和 Linux/macOS 的 `curl` 在某些选项上可能不同（如换行符处理），建议测试跨平台脚本。

### 8. **错误处理**

- 使用 `-f` 或 `--fail` 让 `curl` 在 HTTP 错误时返回非零状态码：

	```bash
  curl -f -o /dev/null http://example.com/404-page
  ```

---

## **三、实用技巧**

### 1. **模拟浏览器行为**

```bash
curl -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)" \
     -H "Accept-Language: en-US" \
     http://example.com
```

### 2. **调试 API 接口**

```bash
curl -v -X POST -H "Content-Type: application/json" -d '{"test":123}' http://example.com/api
```

### 3. **批量下载文件**

```bash
for i in {1..10}; do
  curl -O "http://example.com/file_${i}.txt"
done
```

### 4. **自动化测试**

- 结合脚本检查网站是否在线：

	```bash
  if curl -s --head http://example.com | grep "200 OK"; then
    echo "Website is up!"
  else
    echo "Website is down!"
  fi
  ```

---

## **六、学习资源**

- 官方文档：`man curl` 或 [curl.se/docs](https://curl.se/docs/)
- 社区支持：`curl --help` 查看所有选项。

通过灵活组合选项，`curl` 几乎可以完成所有 HTTP 相关的自动化操作！
