# Cost Estimation

Running the complete stack on Hetzner Cloud is significantly cheaper than managed platforms.

## Comparison

| Provider | Small | Medium | Large |
|----------|-------|--------|-------|
| **Hetzner (self-hosted)** | ~€8/mo | ~€25/mo | ~€68/mo |
| Netlify | €19+ | €99+ | €299+ |
| Vercel | €20+ | €100+ | Custom |
| AWS (EKS + S3 + CloudFront) | €80+ | €200+ | €500+ |

## Hetzner Configurations

### Small (Personal / Small Business)

| Resource | Specification | Monthly Cost |
|----------|---------------|--------------|
| CX22 | 2 vCPU, 4GB RAM, 40GB SSD | ~€4.50 |
| Object Storage | 50GB S3 (Registry) | ~€2.50 |
| Domain | .de domain | ~€1.00 |
| **Total** | | **~€8/month** |

Suitable for:

- 1-5 sites
- <10k visitors/month
- Single-node K3S

### Medium (Agency / Multiple Sites)

| Resource | Specification | Monthly Cost |
|----------|---------------|--------------|
| CX32 (Control) | 4 vCPU, 8GB RAM, 80GB SSD | ~€8.50 |
| CX22 (Worker) | 2 vCPU, 4GB RAM | ~€4.50 |
| Object Storage | 250GB S3 | ~€6.00 |
| Load Balancer | LB11 | ~€6.00 |
| **Total** | | **~€25/month** |

Suitable for:

- 10-20 sites
- <100k visitors/month
- High availability

### Large (Enterprise / High Traffic)

| Resource | Specification | Monthly Cost |
|----------|---------------|--------------|
| CX42 (Control) | 8 vCPU, 16GB RAM | ~€16.50 |
| CX32 (Workers x3) | 4 vCPU, 8GB RAM each | ~€25.50 |
| Object Storage | 1TB S3 | ~€12.00 |
| Load Balancer | LB11 | ~€6.00 |
| Floating IPs | 2x IPv4 | ~€8.00 |
| **Total** | | **~€68/month** |

Suitable for:

- 50+ sites
- <1M visitors/month
- Full redundancy

## What's Included

All configurations include:

- Kubernetes cluster (K3S)
- Container registry with S3 backend
- ArgoCD for GitOps
- Automatic HTTPS (Let's Encrypt)
- Monitoring ready (Prometheus/Grafana optional)

## Additional Costs

| Service | Cost | Notes |
|---------|------|-------|
| GitHub Actions | Free | For public repos |
| Domain | ~€10-15/year | Varies by TLD |
| Backup Storage | ~€0.01/GB | Optional |

## Notes

- Prices are estimates based on Hetzner pricing (2024)
- Actual costs may vary based on traffic and storage
- No egress fees on Hetzner (unlike AWS/GCP)
- All prices exclude VAT
