# modsecurity-dashboard

A Grafana dashboard for JSON formatted kubernetes ingress-nginx modsecurity logs for use with Kuber-Pormetheus-Stack and Loki-Stack. This dashbboard is a heavily-modified derivative of the [NGINX ModSecurity OWASP CRS V0.0](https://grafana.com/grafana/dashboards/15495-nginx-modsecurity-owasp-crs-v0-0/) dashboard by [coffeeflash](https://github.com/coffeeflash). They discus more about it's creation in [this blog post](https://tobisyurt.net/modsecurity-nginx). I have modified the dashboard to use `JSON` formatted logs collected from `/dev/stdout` and changed some formatting for readability.

<img width="1620" alt="Screenshot 2023-05-09 at 15 48 26" src="https://github.com/cloudymax/modsecurity-dashboard/assets/84841307/5b203267-6fc1-48fe-b141-cb4c8f47cda1">

## Requirements

- [Kubernetes ingress-nginx](https://github.com/kubernetes/ingress-nginx) with metrics and modsecurity enabled
- Modesecurity log format set to `JSON`
- Modsecurity log output path `/dev/stdout`
- [Kube-Prometheus-Stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack) (Prometheus, Grafana)
- [Loki-Stack](https://github.com/grafana/helm-charts/tree/main/charts/loki-stack) (Promtail, Loki)

## Enable NGINX MOD Security + Metrics

Update the Nginx configmap:

```bash
kubectl edit configmap -n ingress-nginx ingress-nginx-controller
```

Enable modsecurity:

```yaml
apiVersion: v1
data:
  # ...
  allow-snippet-annotations: "true"
  enable-modsecurity: "true"
  enable-owasp-modsecurity-crs: "true"
  load-balance: ewma
  modsecurity-snippet: |-
    SecRuleEngine DetectionOnly
    SecAuditEngine RelevantOnly
    SecStatusEngine On
    SecRequestBodyAccess On
    SecAuditLog /dev/stdout
    SecAuditLogFormat JSON
  # ...
```

Expose metrics

```bash
helm upgrade ingress-nginx ingress-nginx \
--repo https://kubernetes.github.io/ingress-nginx \
--namespace ingress-nginx \
--set controller.metrics.enabled=true \
--set-string controller.podAnnotations."prometheus\.io/scrape"="true" \
--set-string controller.podAnnotations."prometheus\.io/port"="10254"
```
