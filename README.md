<p align="center">
  <a href="https://swe-agent.com/latest/">
    <img src="assets/swe-agent-banner.png" alt="swe-agent.com" style="height: 12em" />
  </a>
</p>

# Env-agent

**Env-agent** is a fork of [SWE-agent](https://github.com/SWE-agent/SWE-agent) designed to automatically set up environments for Python software repositories and execute their test suites.

> **Important:** This project lives on the `env-setup` branch. Make sure to check out that branch:
> ```bash
> git checkout env-setup
> ```

## What it does

Env-agent automates the following workflow for any Python repository:

1. **Install dependencies** — automatically resolves and installs repository dependencies
2. **Reproduce the testing workflow** — discovers and runs the project's test suite
3. **Build a Docker image** — invokes Docker commands to create a new image with the successful installation and testing steps baked in

## Origin

This project is built on top of [SWE-agent](https://github.com/SWE-agent/SWE-agent), an academic project from Princeton University and Stanford University that enables LMs to autonomously use tools to fix issues in real GitHub repositories.

## License

MIT. See `LICENSE`.

---

<sup>**Footnote — patches for older Python bases (2026-05).** To support Env-agent running against base images with very old system Python (e.g. SusVibes' `dind_py:2.7` / `dind_py:3.5` from `python:2.7-buster` / `python:3.5-buster`), the following local patches were applied:</sup>

<sup>• `tools/edit_anthropic/install.sh` — `pip` → `/root/python3.11/bin/pip3`. The default `pip` resolves to the base image's Python, which on 2.7/3.5 can't install `tree-sitter==0.21.3`.</sup>

<sup>• Tool script shebangs in `tools/edit_anthropic/bin/*`, `tools/registry/bin/*`, `tools/review_on_submit_m/bin/submit` — `#!/usr/bin/env python3` → `#!/root/python3.11/bin/python3`. Pins tool execution to the swerex-provided standalone Python 3.11 regardless of base.</sup>

<sup>Companion patches on the **swerex** side (`swerex/deployment/docker.py`, `glibc_dockerfile` property) make the standalone Python 3.11 portable across buster/bullseye/bookworm targets:</sup>

<sup>1. Builder stage `FROM python:3.11.9-slim-bookworm` → `FROM debian:buster-slim`. Compiling Python 3.11 against glibc 2.28 (buster) instead of 2.36 (bookworm) yields binaries that load on the oldest base we care about; glibc is forward-compatible so the same binary still runs on bullseye (2.31) and bookworm (2.36).</sup>

<sup>2. Inside the new builder, redirect apt sources to `archive.debian.org` and bypass valid-until checks (buster is EOL and its main mirror returns 404). Without this, `apt-get update` fails before the build deps (`wget`, `gcc`, `make`, `zlib1g-dev`, `libssl-dev`) can be installed.</sup>

<sup>3. After `make install`, `cp /usr/lib/x86_64-linux-gnu/libssl.so.1.1 /usr/lib/x86_64-linux-gnu/libcrypto.so.1.1 /root/python3.11/lib/`. Python's `_ssl.so` is linked against OpenSSL 1.1.1 (buster's version) — bundling the matching `.so`s into the standalone's `lib/` makes `import ssl` work on bookworm targets too, where the system ships only `libssl.so.3`. Discoverable via the existing `LD_LIBRARY_PATH=/root/python3.11/lib` set in the production stage.</sup>
