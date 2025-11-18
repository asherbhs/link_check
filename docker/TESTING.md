# Docker Setup Testing Instructions

This document describes how to test the Docker setup once Docker daemon is running.

## Prerequisites

- Docker Desktop must be running
- Docker daemon must be accessible

## Task 1.3: Test Builder Image Standalone

Build the mkdocs-builder image:

```bash
docker build -t mkdocs-builder docker/mkdocs-builder/
```

Verify mkdocs is installed:

```bash
docker run mkdocs-builder mkdocs --version
```

Expected output: `mkdocs, version 1.6.1`

Verify Python version:

```bash
docker run mkdocs-builder python --version
```

Expected output: `Python 3.12.x`

Test with mounted docs:

```bash
mkdir -p tmp/test-output
docker run -v ./tmp/example-mkdocs:/docs -v ./tmp/test-output:/output mkdocs-builder mkdocs build --site-dir /output/site
```

Verify output exists:

```bash
ls tmp/test-output/site/index.html
```

Expected: File exists with HTML content

## Task 1.6: Test Nginx Image Standalone

Build the nginx image:

```bash
docker build -t mkdocs-nginx docker/nginx/
```

Test config syntax:

```bash
docker run mkdocs-nginx nginx -t
```

Expected output: `nginx: configuration file /etc/nginx/nginx.conf test is successful`

Create test HTML:

```bash
mkdir -p tmp/test-html
echo "<html><body><h1>Test</h1></body></html>" > tmp/test-html/index.html
```

Run nginx with volume:

```bash
docker run -d -p 8080:80 -v ./tmp/test-html:/usr/share/nginx/html --name test-nginx mkdocs-nginx
```

Test serving:

```bash
curl http://localhost:8080
```

Expected: HTML content returned

Clean up:

```bash
docker stop test-nginx
docker rm test-nginx
```

## Task 1.8: Integration Test - Basic Build and Serve

Navigate to docker directory:

```bash
cd docker
```

Build services:

```bash
docker compose build
```

Expected: Both services build successfully

Start services:

```bash
docker compose up
```

Expected: Builder runs and exits, nginx continues running

Test in another terminal:

```bash
curl http://localhost:8080/
```

Expected: MkDocs site homepage with "Welcome to Example MkDocs Site"

Check for index page:

```bash
curl -I http://localhost:8080/
```

Expected: `HTTP/1.1 200 OK`

Check for about page:

```bash
curl http://localhost:8080/about/
```

Expected: About page content

Stop services:

```bash
docker compose down
```

Clean up volumes:

```bash
docker compose down -v
```

## Success Criteria

Phase 1 is complete when:

- Builder image builds successfully
- Nginx image builds successfully
- Docker compose configuration is valid
- Services can build and serve a simple MkDocs site
- All pages are accessible via nginx
- Assets load correctly
