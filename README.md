# schema-registry-amr64-images

Nieoficjalne obrazy Docker [Confluent Schema Registry](https://github.com/confluentinc/schema-registry) dla architektury `linux/arm64`, budowane natywnie na ARM64 runnerach GitHub Actions — bez QEMU, bez emulacji.

Obrazy publikowane są do [GHCR](https://github.com/orgs/nucleospl/packages?repo_name=schema-registry-amr64-images) i podpisywane cosign (keyless OIDC). Razem z obrazami publikowany jest Helm chart do instalacji Schema Registry w Kubernetes.

## Dlaczego ten projekt?

Bitnami zaprzestało publikowania obrazów `bitnami/schema-registry`. Ten projekt zastępuje je obrazami budowanymi bezpośrednio z kodu źródłowego Confluent, z pełnym wsparciem ARM64 i gotowym Helm chartem.

## Automatyczne releasy

Releasy tworzone są **w pełni automatycznie**:

1. Workflow `check-release.yml` odpytuje Docker Hub (`confluentinc/cp-schema-registry`) **co 6 godzin**
2. Gdy pojawi się nowy tag (np. `8.3.0`), workflow odnajduje odpowiadający branch i commit SHA w [confluentinc/schema-registry](https://github.com/confluentinc/schema-registry)
3. Automatycznie odpala `build.yml`, który:
   - Buduje obraz Docker na natywnym ARM64 runnerze
   - Skanuje Trivym
   - Pushuje do GHCR i podpisuje cosign
   - Pakuje i publikuje Helm chart do GHCR OCI
   - Tworzy GitHub Release z chartem jako załącznikiem
   - Aktualizuje `versions.json`

Pierwsze uruchomienie (lub ponowny build konkretnej wersji) można wyzwolić ręcznie przez **Actions → Build → Run workflow**.

## Użycie

### Docker

```bash
docker pull ghcr.io/nucleospl/schema-registry-amr64-images/schema-registry:8.2.0
```

Wymagane zmienne środowiskowe:

```bash
docker run --rm \
  -e SCHEMA_REGISTRY_HOST_NAME=localhost \
  -e SCHEMA_REGISTRY_KAFKASTORE_BOOTSTRAP_SERVERS=PLAINTEXT://kafka:9092 \
  -e SCHEMA_REGISTRY_LISTENERS=http://0.0.0.0:8081 \
  -p 8081:8081 \
  ghcr.io/nucleospl/schema-registry-amr64-images/schema-registry:8.2.0
```

### Helm chart

```bash
helm install schema-registry \
  oci://ghcr.io/nucleospl/schema-registry-amr64-images/charts/schema-registry \
  --version 8.2.0 \
  --set kafka.bootstrapServers=PLAINTEXT://kafka:9092
```

Podstawowe `values.yaml`:

```yaml
image: ghcr.io/nucleospl/schema-registry-amr64-images/schema-registry
imageTag: "8.2.0"

kafka:
  bootstrapServers: "PLAINTEXT://kafka-headless:9092"

replicaCount: 1

resources:
  requests:
    cpu: 100m
    memory: 512Mi
  limits:
    memory: 768Mi
```

Pełna lista opcji: [`charts/schema-registry/values.yaml`](charts/schema-registry/values.yaml).

### Weryfikacja podpisu (cosign)

```bash
cosign verify \
  --certificate-identity-regexp="https://github.com/nucleospl/schema-registry-amr64-images/.*" \
  --certificate-oidc-issuer="https://token.actions.githubusercontent.com" \
  ghcr.io/nucleospl/schema-registry-amr64-images/schema-registry:8.2.0
```

## Mapowanie wersji

Wersje obrazów odpowiadają tagom `confluentinc/cp-schema-registry` z Docker Hub. Każdemu tagowi odpowiada branch o tej samej nazwie w [confluentinc/schema-registry](https://github.com/confluentinc/schema-registry).

| Docker Hub / GHCR tag | Branch źródłowy |
|-----------------------|-----------------|
| `8.2.0` | [`8.2.0`](https://github.com/confluentinc/schema-registry/tree/8.2.0) |
| `8.1.2` | [`8.1.2`](https://github.com/confluentinc/schema-registry/tree/8.1.2) |

## Powiązane projekty

- [nucleospl/harbor-arm64-images](https://github.com/nucleospl/harbor-arm64-images) — Harbor ARM64
- [confluentinc/schema-registry](https://github.com/confluentinc/schema-registry) — kod źródłowy
- [confluentinc/cp-helm-charts](https://github.com/confluentinc/cp-helm-charts) — oryginalne charty Confluent

## Licencja

Kod budujący (Dockerfile, workflow, Helm chart): Apache 2.0.
Schema Registry: [Confluent Community License](https://github.com/confluentinc/schema-registry/blob/master/LICENSE-ConfluentCommunity).
