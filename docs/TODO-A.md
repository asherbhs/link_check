# TODO-A: Docker Compose MkDocs with Nginx - Implementation Tasks

Based on PLAN-A.md, broken down into isolated, testable tasks.

## Phase 1: Basic Setup

### Task 1.1: Create Example MkDocs Project
- [ ] Create `tmp/example-mkdocs/` directory structure
- [ ] Create `tmp/example-mkdocs/mkdocs.yml` with basic config
- [ ] Create `tmp/example-mkdocs/docs/index.md` with test content
- [ ] Create `tmp/example-mkdocs/docs/about.md` with test content
- [ ] Verify structure with manual `mkdocs build` (if mkdocs installed locally, otherwise skip)

**Test**: Example project has valid structure for mkdocs

**Files Created**:
- `tmp/example-mkdocs/mkdocs.yml`
- `tmp/example-mkdocs/docs/index.md`
- `tmp/example-mkdocs/docs/about.md`

---

### Task 1.2: Create MkDocs Builder Dockerfile
- [ ] Create `docker/mkdocs-builder/` directory
- [ ] Create `docker/mkdocs-builder/requirements.txt` with mkdocs and mkdocs-material
- [ ] Create `docker/mkdocs-builder/Dockerfile` with Python 3.12 slim base
- [ ] Add steps to install requirements
- [ ] Set working directory to `/docs`
- [ ] Set default command to build site

**Test**: Build image successfully with `docker build -t mkdocs-builder docker/mkdocs-builder/`

**Files Created**:
- `docker/mkdocs-builder/Dockerfile`
- `docker/mkdocs-builder/requirements.txt`

---

### Task 1.3: Test Builder Image Standalone
- [ ] Build the mkdocs-builder image
- [ ] Test mkdocs version: `docker run mkdocs-builder mkdocs --version`
- [ ] Test Python version: `docker run mkdocs-builder python --version`
- [ ] Test with mounted docs: `docker run -v ./tmp/example-mkdocs:/docs -v ./tmp/test-output:/output mkdocs-builder mkdocs build --site-dir /output/site`
- [ ] Verify output in `tmp/test-output/site/index.html`

**Test**: Builder can successfully build a MkDocs site from mounted volume

**No files committed** (test outputs go to tmp/)

---

### Task 1.4: Create Nginx Dockerfile
- [ ] Create `docker/nginx/` directory
- [ ] Create `docker/nginx/Dockerfile` with nginx:alpine base
- [ ] Copy nginx.conf into image
- [ ] Expose port 80

**Test**: Build image successfully with `docker build -t mkdocs-nginx docker/nginx/`

**Files Created**:
- `docker/nginx/Dockerfile`

---

### Task 1.5: Create Basic Nginx Configuration
- [ ] Create `docker/nginx/nginx.conf`
- [ ] Configure server block for port 80
- [ ] Set root to `/usr/share/nginx/html`
- [ ] Add basic MIME types
- [ ] Configure index file handling
- [ ] Add basic logging

**Test**: Validate config syntax by building image and running `docker run mkdocs-nginx nginx -t`

**Files Created**:
- `docker/nginx/nginx.conf`

---

### Task 1.6: Test Nginx Image Standalone
- [ ] Build the nginx image
- [ ] Test config syntax: `docker run mkdocs-nginx nginx -t`
- [ ] Copy test HTML to tmp directory
- [ ] Run nginx with volume: `docker run -p 8080:80 -v ./tmp/test-html:/usr/share/nginx/html mkdocs-nginx`
- [ ] Curl `http://localhost:8080` and verify response

**Test**: Nginx can serve static content from mounted volume

**No files committed** (test outputs go to tmp/)

---

### Task 1.7: Create Basic Docker Compose File
- [ ] Create `docker/docker-compose.yml`
- [ ] Define `builder` service using mkdocs-builder Dockerfile
- [ ] Define `web` service using nginx Dockerfile
- [ ] Add basic volume configuration (no shared volumes yet)
- [ ] Set web service to depend on builder
- [ ] Expose port 8080 on host → 80 on nginx

**Test**: `docker compose config` validates without errors

**Files Created**:
- `docker/docker-compose.yml`

---

### Task 1.8: Integration Test - Basic Build and Serve
- [ ] Run `docker compose -f docker/docker-compose.yml build`
- [ ] Verify both images build successfully
- [ ] Create test script to validate both services can start
- [ ] Document any issues encountered

**Test**: Both services build successfully

**No files committed** (validation only)

---

## Phase 2: Volume Integration

### Task 2.1: Add Shared Volume for Built Site
- [ ] Update `docker-compose.yml` to define `built-site` volume
- [ ] Configure builder service to output to `/output/site`
- [ ] Configure nginx service to serve from `/usr/share/nginx/html`
- [ ] Mount shared volume to both services

**Test**: `docker compose config` validates volume configuration

**Files Modified**:
- `docker/docker-compose.yml`

---

### Task 2.2: Add Source Documentation Volume Mount
- [ ] Update `docker-compose.yml` to mount host directory to builder
- [ ] Use `./tmp/example-mkdocs` as default source
- [ ] Allow override via environment variable `DOCS_SOURCE_PATH`
- [ ] Document the mount in comments

**Test**: Builder service can read from mounted source directory

**Files Modified**:
- `docker/docker-compose.yml`

---

### Task 2.3: Update Builder Command for Shared Volume
- [ ] Modify builder Dockerfile or compose override to build to `/output/site`
- [ ] Ensure builder uses `--clean` flag
- [ ] Verify builder creates all necessary subdirectories

**Test**: Builder outputs to shared volume correctly

**Files Modified**:
- `docker/mkdocs-builder/Dockerfile` or `docker/docker-compose.yml`

---

### Task 2.4: Configure Service Dependencies
- [ ] Add `depends_on` in compose file (web depends on builder)
- [ ] Research and implement service_completed_successfully condition if needed
- [ ] Consider adding health check to builder (exits 0 on success)

**Test**: Services start in correct order

**Files Modified**:
- `docker/docker-compose.yml`

---

### Task 2.5: Integration Test - End-to-End Build and Serve
- [ ] Start services: `docker compose -f docker/docker-compose.yml up`
- [ ] Verify builder completes successfully
- [ ] Verify nginx starts after builder
- [ ] Curl `http://localhost:8080/` and verify MkDocs content
- [ ] Check for expected HTML structure
- [ ] Verify CSS and JS assets load

**Test**: Complete workflow from source docs to served site works

**No files committed** (validation only)

---

### Task 2.6: Test with Different Source Documentation
- [ ] Create second example MkDocs project in `tmp/example-mkdocs-2/`
- [ ] Use environment variable to point to new source
- [ ] Rebuild and verify correct content served
- [ ] Clean up test artifacts

**Test**: System can handle different source directories

**No files committed** (validation only)

---

## Phase 3: Configuration Refinement

### Task 3.1: Optimise Nginx Configuration for MkDocs
- [ ] Add gzip compression for text assets
- [ ] Configure compression level and types
- [ ] Add caching headers for static assets
- [ ] Configure ETags
- [ ] Test compression with curl `-I` to check headers

**Test**: Headers show compression and caching enabled

**Files Modified**:
- `docker/nginx/nginx.conf`

---

### Task 3.2: Handle 404s and Error Pages
- [ ] Add 404 error page configuration
- [ ] Test 404 handling with curl to non-existent page
- [ ] Verify appropriate response code and content

**Test**: 404s return appropriate error page

**Files Modified**:
- `docker/nginx/nginx.conf`

---

### Task 3.3: Configure Logging
- [ ] Set up access log format
- [ ] Set up error log level
- [ ] Test logs are accessible via `docker compose logs`
- [ ] Ensure logs show useful information for debugging

**Test**: Logs are clear and informative

**Files Modified**:
- `docker/nginx/nginx.conf`

---

### Task 3.4: Add MIME Type Coverage
- [ ] Ensure all MkDocs asset types have correct MIME types
- [ ] Test with various file types (CSS, JS, fonts, images)
- [ ] Verify Content-Type headers with curl

**Test**: All asset types serve with correct MIME types

**Files Modified**:
- `docker/nginx/nginx.conf`

---

## Phase 4: Development Experience

### Task 4.1: Add Environment Variables
- [ ] Define environment variables in compose file
- [ ] Add `MKDOCS_THEME` variable for builder
- [ ] Add `NGINX_HOST_PORT` variable with default 8080
- [ ] Add `DOCS_SOURCE_PATH` variable
- [ ] Document variables in comments

**Test**: Environment variables can override defaults

**Files Modified**:
- `docker/docker-compose.yml`

---

### Task 4.2: Create Development Compose Override
- [ ] Create `docker/docker-compose.dev.yml`
- [ ] Add watch mode configuration for builder
- [ ] Add volume mounts optimised for development
- [ ] Consider adding `mkdocs serve` instead of build for live reload
- [ ] Document usage in comments

**Test**: Development mode enables rapid iteration

**Files Created**:
- `docker/docker-compose.dev.yml`

---

### Task 4.3: Add Health Checks
- [ ] Add health check to builder service
- [ ] Add health check to nginx service
- [ ] Test with `docker compose ps` to see health status
- [ ] Verify unhealthy services are detected

**Test**: Health checks accurately reflect service status

**Files Modified**:
- `docker/docker-compose.yml`

---

### Task 4.4: Test Rebuild Workflow
- [ ] Start services
- [ ] Modify source documentation
- [ ] Run rebuild command (document the command)
- [ ] Verify changes appear in served site
- [ ] Time the rebuild process

**Test**: Rebuild workflow is simple and fast

**No files committed** (validation and documentation)

---

## Phase 5: Documentation & Testing

### Task 5.1: Write Docker Setup README
- [ ] Create `docker/README.md`
- [ ] Document prerequisites
- [ ] Document how to build and run
- [ ] Document environment variables
- [ ] Document development vs production mode
- [ ] Add troubleshooting section
- [ ] Add examples

**Test**: New user can follow README successfully

**Files Created**:
- `docker/README.md`

---

### Task 5.2: Create Permanent Example MkDocs Project
- [ ] Create `docs/example-mkdocs/` (not in tmp)
- [ ] Create comprehensive example with multiple pages
- [ ] Add navigation structure
- [ ] Add images and other assets
- [ ] Include code blocks and tables for testing
- [ ] This becomes the reference test project

**Test**: Example project demonstrates all MkDocs features

**Files Created**:
- `docs/example-mkdocs/mkdocs.yml`
- `docs/example-mkdocs/docs/index.md`
- `docs/example-mkdocs/docs/...` (multiple pages)

---

### Task 5.3: Write Integration Tests
- [ ] Create `tests/test_docker_setup.py`
- [ ] Test: Builder image builds successfully
- [ ] Test: Nginx image builds successfully
- [ ] Test: Compose file is valid
- [ ] Test: Services start successfully
- [ ] Test: Site is accessible on localhost:8080
- [ ] Test: Index page contains expected content
- [ ] Test: Assets load correctly
- [ ] Use pytest for all tests

**Test**: All integration tests pass

**Files Created**:
- `tests/test_docker_setup.py`

---

### Task 5.4: Document Common Issues
- [ ] Add section to README for common issues
- [ ] Document permission issues and solutions
- [ ] Document port conflicts and solutions
- [ ] Document build failures and solutions
- [ ] Document cleanup procedures

**Test**: Common issues are well-documented

**Files Modified**:
- `docker/README.md`

---

### Task 5.5: Performance Baseline
- [ ] Measure build time for example project
- [ ] Measure nginx response time under load
- [ ] Document baseline metrics
- [ ] Add performance notes to README

**Test**: Performance metrics are documented

**Files Modified**:
- `docker/README.md`

---

## Final Validation

### Task 6.1: Full Test Suite
- [ ] Run all integration tests
- [ ] Run pytest with coverage
- [ ] Verify all tests pass
- [ ] Check for any pytest warnings

**Test**: Complete test suite passes with no warnings

---

### Task 6.2: Clean Test Run
- [ ] Remove all containers and volumes
- [ ] Run `docker compose up` from scratch
- [ ] Verify everything works without cached state
- [ ] Document any issues

**Test**: Clean build works perfectly

---

### Task 6.3: Documentation Review
- [ ] Review all README files for accuracy
- [ ] Verify all examples work as documented
- [ ] Check for typos and clarity
- [ ] Ensure UK English throughout

**Test**: Documentation is clear and complete

---

### Task 6.4: Code Quality
- [ ] Run black on any Python files
- [ ] Run ruff on any Python files
- [ ] Ensure pre-commit hooks pass
- [ ] Clean up any temporary files not in tmp/

**Test**: All code quality checks pass

---

## Success Criteria Checklist

- [ ] Can mount any MkDocs project source directory
- [ ] Builds successfully on `docker compose up`
- [ ] Serves on specified port (default 8080)
- [ ] All assets load correctly (CSS, JS, images)
- [ ] Navigation works as expected
- [ ] Rebuild workflow is clear and simple
- [ ] Development mode supports rapid iteration
- [ ] Production mode is optimised for serving
- [ ] All tests pass
- [ ] Documentation is clear and complete

## Notes

- All temporary test files should go in `tmp/` directory
- Only production-ready files should be committed
- Each task should be testable in isolation
- Run full test suite before marking phase complete
- Update this TODO as tasks are completed
- Remove completed tasks or mark with [x]
