# Coolify Deployment

Use this setup when Crawl4AI is deployed behind Coolify and LLM traffic should go through LiteLLM.

## Resource

- Type: Docker Compose
- Repository: `https://github.com/nanw01/crawl4ai.git`
- Compose file: `docker-compose.coolify.yml`
- Application port: `11235`
- Health check path: `/health`

## Environment

Set these in Coolify, not in the repository:

```env
OPENAI_BASE_URL=http://host.docker.internal:14000/v1
OPENAI_API_KEY=sk-your-litellm-key
LLM_PROVIDER=openai/aiclient-gemini-2-5-flash
LLM_TEMPERATURE=0
REDIS_TASK_TTL=3600
CRAWL4AI_HOST_PORT=11235
CRAWL4AI_HOOKS_ENABLED=false
CRAWL4AI_MAX_PAGES=20
```

This WSL Coolify installation runs the proxy separately from application containers. The compose file includes a small `crawl4ai-host-bridge` service that publishes host port `11235` and forwards traffic to the internal Crawl4AI service.

Use the local LiteLLM bridge instead of `https://litellm.nanlab.xyz/v1` from Crawl4AI containers. The public domain can be blocked by Cloudflare for container-originated traffic, while `http://host.docker.internal:14000/v1` stays inside the WSL host.

The WSL host bridge can be managed with systemd:

```bash
sudo tee /etc/systemd/system/litellm-bridge.service >/dev/null <<'EOF'
[Unit]
Description=Bridge LiteLLM from WSL host to local containerd network
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=/usr/bin/socat -d -d TCP4-LISTEN:14000,fork,reuseaddr,bind=0.0.0.0 TCP4:10.0.3.2:4000
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable --now litellm-bridge.service
```

If LiteLLM is in the same Coolify private network, prefer the internal service URL:

```env
OPENAI_BASE_URL=http://litellm:4000/v1
```

## Validation

After deployment, verify:

- `https://your-crawl4ai-domain/health`
- `https://your-crawl4ai-domain/playground`
- `https://your-crawl4ai-domain/dashboard`

Run an LLM extraction request from the playground using the same model name configured in `LLM_PROVIDER`.

## Optional Hardening

For a public domain, enable Coolify basic auth or turn on Crawl4AI JWT:

```env
CRAWL4AI_SECURITY_ENABLED=true
CRAWL4AI_JWT_ENABLED=true
CRAWL4AI_API_TOKEN=use-a-long-random-token
SECRET_KEY=use-a-long-random-secret
CRAWL4AI_TRUSTED_HOSTS=your-crawl4ai-domain.example.com
```

Keep `/health` public for Coolify health checks. API endpoints such as `/crawl`, `/md`, `/html`, `/pdf`, and `/ask` require a bearer token when JWT is enabled.
