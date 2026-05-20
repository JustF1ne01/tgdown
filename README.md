# tgdown — Telegram 视频下载

基于 Telethon 的 Telegram 群视频自动下载工具：监听指定群内的视频消息并自动下载到本地，支持 Web 状态面板、下载记录分页查询、可选代理与 AI 命名。

---

## 功能概览

- **群消息监听**：在指定 Telegram 群中，有视频消息时自动加入下载队列并下载
- **链接解析**：群消息正文中的 `t.me/用户名/消息ID` 链接会自动解析并加入下载队列
- **状态推送**：下载开始/完成/失败时可推送到该群；进程启动完成后也会推送一条「已启动」及当前时间
- **Web 面板**：实时查看正在下载、未下载列表、最近记录；数据库记录支持分页
- **持久化**：下载成功记录与未完成任务写入 SQLite，支持分页查询；容器重启后可自动恢复未完成下载任务
- **临时目录分离**：下载中的临时文件统一放入独立临时目录
- **可选代理**：支持 SOCKS5/SOCKS4/HTTP 代理
- **可选 AI 命名**：配置 OpenAI 兼容 API 后，可根据消息文案或原文件名生成更友好的本地文件名

---

## 环境要求

- Python 3.10+
- Telegram API 凭证：在 [my.telegram.org](https://my.telegram.org) 申请 `api_id` 与 `api_hash`

---

## 配置说明

所有配置存放在 **`data/config.json`**（本地运行时为项目下的 `data` 目录；Docker 运行时为挂载的 `/data`）。

### 字段说明

| 配置项 | 是否必填 | 默认值 | 说明 |
|--------|----------|--------|------|
| `api_id` | 是 | 无 | Telegram 应用 `api_id` |
| `api_hash` | 是 | 无 | Telegram 应用 `api_hash` |
| `tg_device_name` | 否 | `"tgdown"` | 登录后在 Telegram「设置 → 隐私与安全 → 活跃会话」中显示的设备名，便于区分多台机器；也支持旧键名 `device_model` |
| `tg_system_version` | 否 | `""` | 可选，会话里显示的系统版本；留空则使用 Telethon 默认 |
| `tg_app_version` | 否 | `""` | 可选，会话里显示的应用版本；留空则使用 Telethon 默认 |
| `tg_message_prefix` | 否 | `"[tgdown]"` | 脚本自动发往群的消息会在**首行**加该标识（与正文换行分隔），便于与人工消息区分；设为 `""` 则不加 |
| `target_group_name` | 否 | `"tgdown"` | 监听的目标群名称，需与群标题一致 |
| `web_port` | 否 | `8765` | Web 面板端口 |
| `concurrent_downloads` | 否 | `3` | 并发下载数 |
| `openai_api_key` | 否 | `""` | OpenAI 兼容接口的 API Key，用于 AI 命名 |
| `openai_base_url` | 否 | `""` | OpenAI 兼容接口地址，例如 `https://api.openai.com/v1` |
| `tg_proxy_type` | 否 | `""` | Telegram 代理类型，可选 `socks5`、`socks4`、`http` |
| `tg_proxy_host` | 否 | `""` | Telegram 代理地址 |
| `tg_proxy_port` | 否 | `0` | Telegram 代理端口 |
| `tg_proxy_username` | 否 | `""` | Telegram 代理用户名，没有可留空 |
| `tg_proxy_password` | 否 | `""` | Telegram 代理密码，没有可留空 |


### 配置示例


```json
{
  "api_id": 36684684,
  "api_hash": "",
  "tg_device_name": "tgdown",
  "tg_system_version": "",
  "tg_app_version": "",
  "tg_message_prefix": "[tgdown]",
  "web_port": 8765,
  "target_group_name": "tgdown",
  "concurrent_downloads": 3,
  "openai_api_key": "",
  "openai_base_url": "",
  "tg_proxy_type": "http",
  "tg_proxy_host": "",
  "tg_proxy_port": 7893,
  "tg_proxy_username": "",
  "tg_proxy_password": ""
}

```

---

## 部署

## 命令行 Docker 部署

```bash
cd tgdown
# 第一次使用需要初始化session信息，运行脚本后，输入手机号然后发送验证码，用验证码登录成功后获取到session就可以了
# 第一次使用先手动在挂载目录下创建配置文件 config.json 然后运行容器
# downloads、temp_downloads、session、日志都会默认落到 挂载目录 下
# 默认amd架构, 如果需要arm架构 运行的时候修改镜像版本 例如: xxgl/tgdown:1.1-arm
docker run --rm -it \
  --name tgdown \
  --network host \
  -v "/data/server/downapp/data:/data" \
  xxgl/tgdown:1.3
```
![init](/docs/images/init.png)
![yzm](/docs/images/yzm.png)
![session](/docs/images/session.png)
```bash
cd tgdown
# 运行时只需要挂载 data 目录，下载目录和临时目录默认都在 /data 下
# 运行时需要把挂载路径替换为自己的实际路径
# 如果视频保存目录在另一个文件夹，可以在加一个挂载路径 挂载到 /data/downloads 路径下
docker run -d \
  --name tgdown \
  --network host \
  -v "/data/server/downapp/data:/data" \
  --restart=always \
  xxgl/tgdown:1.3
```


## Docker 面板部署
1、在挂载目录创建 config.json 配置文件，配置文件填写正确配置信息后，创建运行容器时挂载到容器的/data 目录下然后运行容器。
![init](/docs/images/fn1.png)
![init](/docs/images/fn2.png)
2、打开容器的 bash 命令行需要手动执行命令(python tgdown.py)进行初始化

3、输入账号和登录验证码信息

4、在面板重启容器。
![init](/docs/images/fn3.png)
![init](/docs/images/fn4.png)

下载文件命名规则
- 1、AI 命名成功（优先消息文案，其次中文原文件名）：`ai_AI文件名_时间.mp4`
- 2、未走 AI 命名：`清洗后的原文件名_时间.mp4`


![qun](/docs/images/qun.png)
![down](/docs/images/down.png)

## 许可证

见项目内 [LICENSE](LICENSE) 文件。
