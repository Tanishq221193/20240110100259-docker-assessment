# FestPass Docker & Container Security Assessment

## Student and project

- **Name:** Tanishq Garg
- **REGNO:** 202401100100259
- **PORT:** 8059
- **Docker Hub username:** `tanishqgarg`
- **GitHub repository:** [Tanishq221193/20240110100259-docker-assessment](https://github.com/Tanishq221193/20240110100259-docker-assessment)

FestPass is a three-tier registration application: an Nginx frontend, a Python
Flask API, and a PostgreSQL database. The frontend proxies `/api/` requests to
the backend. The backend and database share a private network; the frontend
does not join the database network. PostgreSQL data is stored in the named
`pgdata` volume.

## Run the application

Requirements: Docker Engine and the Docker Compose plugin.

Copy the example environment file, set a private database password, and start
the stack:

```sh
cp .env.example .env
# Edit .env and replace DB_PASSWORD with a strong password.
docker compose up -d
```

Open [http://localhost:8059](http://localhost:8059). The API health endpoint is
[http://localhost:8059/api/health](http://localhost:8059/api/health). Check
service status and stop the stack with:

```sh
docker compose ps
docker compose down
```

To confirm registrations persist, create registrations in the UI, run
`docker compose down`, start the stack again with `docker compose up -d`, and
check the UI. The named database volume is retained by `docker compose down`;
do not use `docker compose down -v` if you want to keep its data.

For terminal evidence, set your Bash prompt to include your registration
number in each new terminal:

```sh
export PS1='[202401100100259] \w $ '
```

Commit each completed assessment stage separately and push the completed work
at the end. Take your own full-window screenshots and save them in
`screenshots/`.

## Build and container security

The backend image is configured to run as an unprivileged user. Its Dockerfile
installs pinned Python dependencies before copying the application source. The
Compose configuration uses a PostgreSQL health check and waits for the database
before starting the backend. Only the frontend publishes a host port.

Build the backend locally with a version label:

```sh
docker build \
  --build-arg APP_VERSION=0.2 \
  -t fest-backend:0.2 \
  ./backend
```

Run vulnerability scans with Trivy and save the actual output in `reports/`:

```sh
mkdir -p reports
trivy image fest-backend:0.1 --format table --output reports/trivy-before.txt
trivy image fest-backend:0.2 --format table --output reports/trivy-after.txt
trivy image --severity CRITICAL --ignore-unfixed --exit-code 1 fest-backend:0.2
```

The final command should exit with code `0` when no fixable CRITICAL findings
are present. Record the actual command result and scan totals below; no scan
reports are currently included in this repository. Use a trusted Trivy release
and installation source; do not install versions `0.69.4`, `0.69.5`, or
`0.69.6`.

### Trivy results and image size

Fill in this table from the saved Trivy reports and measured local image sizes.
Do not treat unrecorded results as zero findings.

| Image | CRITICAL | HIGH | Image size |
| --- | ---: | ---: | ---: |
| `fest-backend:0.1` (before fixes) | Not scanned/recorded | Not scanned/recorded | Not measured |
| `fest-backend:0.2` (after fixes) | Not scanned/recorded | Not scanned/recorded | Not measured |

**CRITICAL gate result for `fest-backend:0.2`:** Not run/recorded.

## Publish and verify the backend image

The Compose file references
[`tanishqgarg/fest-backend:1.0.0`](https://hub.docker.com/r/tanishqgarg/fest-backend).
Record the digest from the successful Docker Hub push or registry manifest for
the exact `1.0.0` image:

- **`1.0.0` digest:** Not recorded in this repository.

The assessment also requires publishing the `1.0` tag and verifying that a
fresh Compose start pulls and runs `1.0.0`. Record evidence of those actions
before marking them complete.

## Sign and verify with Cosign

The assessment requires signing the `1.0.0` image by digest, verifying it with
the public key, and demonstrating that an unsigned `1.0.1` image is rejected.
No Cosign public key or verification evidence is currently included here.
Never commit `cosign.key`, passwords, access tokens, or `.env`.

## Assessment progress and evidence

Assessment screenshots belong in `screenshots/`, named with the task evidence
codes (for example, `S1-2.png` or `S5-2_S5-3.png`). Trivy output reports belong
in `reports/`. Terminal screenshots should show the registration-number prompt
and command; browser screenshots should show the address bar. Do not edit or
reuse another student's evidence.

Not yet evidenced in this repository:

- Backend build/run screenshots for Stage 1.
- Compose networking, UI registration, and persistence screenshots for Stage 2.
- Trivy before/after reports, measured image sizes, vulnerability totals, and gate result for Stage 3.
- Docker Hub push digest and registry-run screenshot for Stage 4.
- `cosign.pub`, signature verification, and unsigned-image rejection evidence for Stage 5.

## AI tool disclosure

GitHub Copilot in VS Code was used to review the provided assessment
requirements and draft this README. Commands, scans, image publication, and
signing must be run and verified by the student; this README does not claim
unperformed results.

## Project layout

```text
.
├── backend/
│   ├── .dockerignore
│   ├── Dockerfile
│   ├── app.py
│   └── requirements.txt
├── frontend/
│   ├── index.html
│   └── nginx.conf
├── .env.example
├── .gitignore
├── compose.yaml
└── README.md
```

## Assessment reference

Structure and evidence requirements are based on the
[Docker & Container Security — Practical Assessment](https://github.com/greatvand/KIET/blob/main/Docker/Docker%20%26%20Container%20Security%20%E2%80%94%20Practical%20Assessment.pdf).
