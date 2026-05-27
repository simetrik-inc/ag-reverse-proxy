# ag-reverse-proxy

Nginx reverse proxy sidecar for the Agents (AG) project ECS services.

Adds mandatory security headers and strips proxy-disclosure headers from upstream responses.

## Security headers added

| Header | Value |
|--------|-------|
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains; preload` |
| `X-Content-Type-Options` | `nosniff` |
| `X-Frame-Options` | `DENY` |
| `X-XSS-Protection` | `1; mode=block` |
| `Referrer-Policy` | `strict-origin-when-cross-origin` |

## Headers stripped from upstream

`Via`, `X-Powered-By`, `Server`, `X-Proxy-Cache`, `X-Cache`, `X-Cache-Hit`

## Build args

| Arg | Default | Description |
|-----|---------|-------------|
| `PROXY_HOST` | `127.0.0.1` | Upstream host (localhost in ECS awsvpc) |
| `PROXY_PORT` | `8000` | Upstream app port |
| `SERVER_NAME` | `_` | nginx `server_name` directive |

## Usage in Harness (BuildAndPushECR)

```yaml
buildArgs:
  PROXY_HOST: 127.0.0.1
  PROXY_PORT: <app_port>
  SERVER_NAME: <domain_name>
```

## Deploying to a new environment

1. In the service IaC repo (`*-ag-us-iac/services.tf`), set:
   ```hcl
   create_with_proxy = "true"
   repository_proxy  = "https://github.com/simetrik-inc/ag-reverse-proxy.git"
   branch_proxy      = "main"
   ```
2. Apply Terraform — this creates the ECR repo for the proxy.
3. Run the Harness `ag_backend_proxy_build` pipeline pointing to this repo and the target ECR.
4. Redeploy the ECS service so the task definition picks up the new proxy image.
