# Plan A: Docker Compose Setup for MkDocs with Nginx

## Overview

Create a Docker Compose setup that mounts MkDocs source documentation at runtime, builds the static site, and serves it via nginx.

## Architecture

### Multi-Stage Build Approach

**Stage 1: Builder Container**
- Base: Python slim image
- Install MkDocs and required plugins
- Mount documentation source as volume
- Build static site to output directory
- Output: Built HTML/CSS/JS in `site/` directory

**Stage 2: Nginx Server**
- Base: nginx:alpine
- Copy built site from builder stage or shared volume
- Serve static content
- Configure nginx for optimal MkDocs delivery

### Volume Strategy

Two possible approaches:

**Option A: Shared Volume**
- Source docs mounted to builder
- Builder outputs to shared volume
- Nginx reads from shared volume
- Pros: True runtime mounting, easier development iteration
- Cons: Requires orchestration to ensure build completes before nginx starts

**Option B: Build-Time Copy**
- Build happens in builder stage
- Output copied to nginx image
- Pros: Simpler, self-contained nginx container
- Cons: Requires rebuild for doc changes (less suitable for "runtime mounting")

**Recommendation**: Option A for true runtime mounting requirement.

## Component Breakdown

### 1. MkDocs Builder Service

**Dockerfile** (`docker/mkdocs-builder/Dockerfile`):
- Base image: `python:3.12-slim`
- Install: `mkdocs`, `mkdocs-material` (or other theme)
- Install: common plugins (`mkdocs-minify-plugin`, etc.)
- Working directory: `/docs`
- Command: `mkdocs build --clean --site-dir /output/site`

**Responsibilities**:
- Read source from `/docs` (mounted volume)
- Build to `/output/site` (shared volume)
- Exit when build completes (or run in watch mode for development)

### 2. Nginx Service

**Dockerfile** (`docker/nginx/Dockerfile`):
- Base image: `nginx:alpine`
- Copy custom nginx configuration
- Expose port 80

**Configuration** (`docker/nginx/nginx.conf`):
- Serve from `/usr/share/nginx/html`
- Compression enabled (gzip)
- Cache headers for static assets
- Handle MkDocs routing (SPA-style if needed)
- Proper MIME types for all assets

**Responsibilities**:
- Serve built site from shared volume
- Handle HTTP requests efficiently
- Provide appropriate caching headers

### 3. Docker Compose Orchestration

**Services**:
- `builder`: MkDocs build service
- `web`: Nginx serving service

**Volumes**:
- `docs-source`: Mount point for MkDocs source (host → builder)
- `built-site`: Shared volume for built output (builder → nginx)

**Networks**:
- Not strictly required (nginx doesn't call builder)
- Could use default bridge network

**Service Dependencies**:
- `web` depends on `builder` completing
- Use `depends_on` with health checks or init containers

## Development Workflow

### Initial Build
1. `docker compose up --build`
2. Builder service builds site
3. Nginx service starts and serves content
4. Access at `http://localhost:8080`

### Development Mode (Watch)
1. Builder runs `mkdocs serve` in watch mode
2. Or: Builder runs `mkdocs build` in loop watching for changes
3. Nginx auto-reloads when files change
4. Hot reload for documentation authors

### Production Mode
1. Builder runs once, exits
2. Nginx continues serving
3. Restart builder to rebuild

## Testing Strategy

### Unit Testing

**Test 1: Builder Dockerfile**
- Build the builder image standalone
- Verify mkdocs is installed: `docker run builder mkdocs --version`
- Verify Python version
- Verify required plugins present

**Test 2: Nginx Dockerfile**
- Build nginx image standalone
- Verify nginx config syntax: `nginx -t`
- Verify config file present

**Test 3: Volume Mounts**
- Create test MkDocs project in `tmp/test-docs/`
- Run builder with volume mount
- Verify output appears in `tmp/test-output/`

### Integration Testing

**Test 4: Full Stack Smoke Test**
- `docker compose up`
- Curl `http://localhost:8080`
- Verify 200 response
- Verify HTML content contains expected text

**Test 5: Content Verification**
- Mount sample docs with known structure
- Build and serve
- Verify index page accessible
- Verify navigation works
- Verify assets (CSS, JS) load correctly

**Test 6: Rebuild on Change**
- Start services
- Modify source documentation
- Trigger rebuild (manually or via watch)
- Verify changes reflected in served content

### Performance Testing

**Test 7: Build Time**
- Measure build time for various doc sizes
- Small (10 pages), medium (100 pages), large (1000 pages)
- Establish baseline

**Test 8: Serving Performance**
- Use `ab` (Apache Bench) or similar
- Test concurrent requests
- Verify nginx handles load appropriately

## Iteration Plan

### Phase 1: Basic Setup
1. Create `docker/mkdocs-builder/Dockerfile`
2. Create `docker/nginx/Dockerfile`
3. Create basic `nginx.conf`
4. Create `docker-compose.yml` with both services
5. Test with minimal MkDocs project

**Validation**: Can build and serve a simple "Hello World" MkDocs site

### Phase 2: Volume Integration
1. Configure proper volume mounts in compose file
2. Add shared volume for built output
3. Add dependency management (builder → nginx)
4. Test with mounted source directory

**Validation**: Can mount external docs and serve them

### Phase 3: Configuration Refinement
1. Optimise nginx config for MkDocs
2. Add compression, caching headers
3. Handle 404s appropriately
4. Configure logging

**Validation**: Production-ready nginx configuration

### Phase 4: Development Experience
1. Add watch mode option for builder
2. Add environment variables for config
3. Create separate dev and prod compose files
4. Add health checks

**Validation**: Smooth developer experience with hot reload

### Phase 5: Documentation & Testing
1. Write README for docker setup
2. Add example MkDocs project for testing
3. Create integration tests
4. Document common issues and solutions

**Validation**: New team member can run setup without assistance

## File Structure

```
docker/
├── mkdocs-builder/
│   ├── Dockerfile
│   └── requirements.txt          # MkDocs and plugins
├── nginx/
│   ├── Dockerfile
│   └── nginx.conf
└── docker-compose.yml            # Main orchestration
└── docker-compose.dev.yml        # Development overrides

docs/
└── example-mkdocs/               # Test MkDocs project
    ├── mkdocs.yml
    └── docs/
        └── index.md

tests/
└── test_docker_setup.py          # Integration tests
```

## Environment Variables

**Builder**:
- `MKDOCS_THEME`: Theme to use (default: material)
- `MKDOCS_STRICT`: Strict mode on/off
- `WATCH_MODE`: Enable watch mode for development

**Nginx**:
- `NGINX_PORT`: Port to expose (default: 80)
- `NGINX_WORKER_PROCESSES`: Worker process count

**Compose**:
- `DOCS_SOURCE_PATH`: Path to MkDocs source on host
- `NGINX_HOST_PORT`: Port to expose on host (default: 8080)

## Potential Issues & Solutions

### Issue 1: Timing - Nginx starts before build completes
**Solution**: Use health check on builder, or init container pattern

### Issue 2: Permissions on shared volume
**Solution**: Match UIDs between builder and nginx, or use appropriate chmod in builder

### Issue 3: Large documentation causes slow builds
**Solution**: Implement caching strategy, consider incremental builds

### Issue 4: Assets not loading (MIME types)
**Solution**: Ensure nginx.conf includes proper MIME type mappings

### Issue 5: Hot reload not working in development
**Solution**: Implement proper file watching mechanism, consider websocket support

## Success Criteria

1. ✓ Can mount any MkDocs project source directory
2. ✓ Builds successfully on `docker compose up`
3. ✓ Serves on specified port (default 8080)
4. ✓ All assets load correctly (CSS, JS, images)
5. ✓ Navigation works as expected
6. ✓ Rebuild workflow is clear and simple
7. ✓ Development mode supports rapid iteration
8. ✓ Production mode is optimised for serving
9. ✓ All tests pass
10. ✓ Documentation is clear and complete

## Next Steps

After approval of this plan:
1. Create feature branch: `docker-mkdocs-nginx-setup`
2. Implement Phase 1 (Basic Setup)
3. Write tests for Phase 1
4. Get approval before proceeding to Phase 2
5. Iterate through remaining phases with test-driven approach
