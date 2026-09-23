# Yandex CloudCost

Chart 2.5.29-yc.6 / exporter 1.121.1-yc.5 adds daily resource billing through
Yandex Billing Usage API. UI remains 1.121.1-yc.1.

```yaml
opencost:
  exporter:
    replicas: 1
    yandexCloud:
      enabled: true
      currency: RUB
      serviceAccountKey:
        existingSecret: opencost-yc-key
  cloudCost:
    enabled: true
    yandex:
      enabled: true
      billingAccountId: YOUR_ACCOUNT_ID
      usageEndpoint: billing.api.cloud.yandex.net:443
      billingTimezone: Europe/Moscow
      folderIds: []
```

The account ID is mandatory. Empty folderIds selects the whole account. The
service account needs billing.accounts.viewer on that account and k8s.viewer
for MKS discovery. The mounted authorized key is reused; no key contents enter
the integration JSON. Prefer an existing Secret: inline key JSON is stored in
Helm release data.

Do not combine these convenience settings with cloudIntegrationJSON or
cloudIntegrationSecret. For multiple providers use a manual integration file
with a yandex.usageAPI array, as documented in the exporter provider README.
The convenience mode requires one replica and uses Recreate updates. Separate
deployments behind one NAT share the provider's API limit but not a distributed
limiter. Keep ingestion for this account enabled in one installation.

Daily buckets preserve YC billing dates. ListCost is pre-credit cost; NetCost
includes monetary grants. Invoiced and amortized metrics are expense-based
approximations, not finalized invoices or commitment amortization. A fully
grant-covered account displays zero on the default amortized-net view; choose
List Cost for pre-credit spend. These costs overlap Kubernetes allocations.

History uses the upstream in-memory CloudCost repository: 30 days reload after
restart, with temporary gaps during the rate-limited backfill. A PVC does not
persist this repository. Check /cloudCost/status and logs. Resource metadata may
be incomplete, and unmatched Kubernetes attribution means unclassified.

Endpoints, folder filters, currency and timezone are configurable. Non-RUB
accounts require an explicit appropriate timezone; KZ is not validated.
