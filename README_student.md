## About

Nginx exam submission completed by Uwe Rosner as part of the Liora MLOps training.

## Architecture

```
                 HTTP :8080 -> 301 redirect to HTTPS
Client ──HTTPS :443-> nginx ──┬-> api-v1  (3 replicas, round robin)
                              └-> api-v2  (only with header X-Experiment-Group: debug)

nginx :8081/nginx_status -> nginx-exporter -> Prometheus :9090 -> Grafana :3000
```

Nginx is the only entry point; the APIs are not published to the host.
On `/predict` it applies:

| Feature       | Implementation                                                 |
| ------------- | -------------------------------------------------------------- |
| HTTPS         | Self-signed certificate in `deployments/nginx/certs/`          |
| Basic auth    | `deployments/nginx/.htpasswd` (user `admin`, password `admin`) |
| Rate limiting | 10 requests/s per IP, burst of 5, then `429 Too Many Requests` |
| A/B testing   | `map` on header `X-Experiment-Group` selects the upstream      |

## Usage

Requirements: Docker with Compose v2, `make`, `curl`.

```bash
make start-project   # build and start; returns once all services are healthy
make test            # run tests/run_tests.sh
make stop-project
```

Example request (add `-H "X-Experiment-Group: debug"` to reach api-v2):

```bash
curl -X POST https://localhost/predict \
     -H "Content-Type: application/json" \
     -d '{"sentence": "Oh yeah, that was soooo cool!"}' \
     --user admin:admin \
     --cacert deployments/nginx/certs/nginx.crt
```

Grafana: http://localhost:3000 (`admin` / `admin`), Prometheus: http://localhost:9090

## Deviations from the target structure

- **No `requirements.txt`:** dependencies are managed with [uv](https://docs.astral.sh/uv/)
  (`pyproject.toml`, `uv.lock`); the API images install them from `uv.lock`.
- **Python 3.13 and scikit-learn 1.6.1:** `model.joblib` was created with
  scikit-learn 1.6.1, which has no wheels for newer Python versions.
- **No `data/` and `src/gen_model.py`:** the pre-trained model is provided,
  so no training is needed.
