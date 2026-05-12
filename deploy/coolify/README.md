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
OPENAI_BASE_URL=https://litellm.nanlab.xyz/v1
OPENAI_API_KEY=sk-your-litellm-key
LLM_PROVIDER=openai/your-litellm-model-name
LLM_TEMPERATURE=0
REDIS_TASK_TTL=3600
CRAWL4AI_HOST_PORT=11235
CRAWL4AI_HOOKS_ENABLED=false
CRAWL4AI_MAX_PAGES=20
```

This WSL Coolify installation runs the proxy separately from application containers. The compose file includes a small `crawl4ai-host-bridge` service that publishes host port `11235` and forwards traffic to the internal Crawl4AI service.

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
