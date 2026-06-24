# security-workflows

Reusable GitHub Actions workflows for Trivy container and dependency security scanning,
shared across all PHP/Docker repos under [@superhans65](https://github.com/superhans65).

## What it does

Each scan run does two things:

| Step | What Trivy scans | Why |
|------|-----------------|-----|
| Image scan | The built Docker image (Debian Bookworm + PHP 8.1 CLI) | Catches OS-level and runtime CVEs |
| Filesystem scan | `composer.lock` in the repo checkout | Catches PHP dependency CVEs not installed in the image |

Both steps fail the workflow on **HIGH** or **CRITICAL** findings (unfixed vulns are ignored).

---

## Adding a new repo

1. **Copy the caller workflow** into your repo:

   ```
   cp examples/security-scan.yml path/to/your-repo/.github/workflows/security-scan.yml
   ```

2. **Edit the `image-name` input** to match your project:

   ```yaml
   with:
     image-name: your-repo-name:${{ github.sha }}
   ```

3. **Commit and push.** The scan runs automatically on every push/PR to `main`/`master`
   and once a week (Monday 08:00 UTC) to catch newly published CVEs.

4. *(Optional)* If your `Dockerfile` isn't at the repo root, or your build context
   is a subdirectory, add the optional inputs:

   ```yaml
   with:
     image-name: your-repo-name:${{ github.sha }}
     dockerfile-path: docker/Dockerfile
     build-context: .
   ```

---

## Workflow inputs reference

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `image-name` | yes | — | Image name + tag to build and scan |
| `dockerfile-path` | no | `Dockerfile` | Path to Dockerfile relative to repo root |
| `build-context` | no | `.` | Docker build context directory |
| `severity` | no | `HIGH,CRITICAL` | Severities that fail the workflow |

---

## Suppressing a false positive

Add a `.trivyignore` file to the root of the **calling repo** listing CVE IDs to skip:

```
# .trivyignore
CVE-2023-XXXXX  # reason: not reachable in our configuration
```

Trivy picks this up automatically during both image and filesystem scans.

---

## Repos using this workflow

- [market-hub](https://github.com/superhans65/market-hub)
- *(add yours here)*
