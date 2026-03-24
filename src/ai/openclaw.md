# openclaw

* url:  `https://docs.openclaw.ai/providers/google`


```
docker build -t openclaw:local -f Dockerfile .
docker compose run --rm openclaw-cli onboard
docker compose up -d openclaw-gateway

# （可选）配置模型
docker compose run --rm openclaw-cli config

# 绑定服务器
docker compose run --rm openclaw-cli dashboard --no-open
docker compose run --rm openclaw-cli devices list
docker compose run --rm openclaw-cli devices approve <requestId>
```

```shell
#.env
OPENCLAW_IMAGE=ghcr.io/openclaw/openclaw:2026.3.22
OPENCLAW_CONFIG_DIR=./.openclaw/config
OPENCLAW_WORKSPACE_DIR=./.openclaw/workspace
OPENCLAW_GATEWAY_TOKEN=$(生成)  # 强烈建议设置
OPENCLAW_TZ=Asia/Shanghai
```

```shell
# 加权
chown -R 1000:1000 .openclaw/
```

```json
  "gateway": {
    "port": 18789,
    "mode": "local",
    "bind": "loopback",
    "controlUi": {
      "allowedOrigins": ["http://127.0.0.1:18789"]
    },
```

