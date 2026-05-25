<p align="center">
  <a href="https://swe-agent.com/latest/">
    <img src="assets/swe-agent-banner.png" alt="swe-agent.com" style="height: 12em" />
  </a>
</p>

# SWE-agent for SusVibes

A fork of [SWE-agent](https://github.com/SWE-agent/SWE-agent) for the SusVibes project. The repository is organized into two branches:

- **`sv`** — the default SWE-agent version, the baseline agent.
- **`sv-env-setup`** — extends SWE-agent to automatically set up environments for Python software repositories and execute their test suites.

## What `sv` does

The `sv` branch tracks the default SWE-agent version and focuses on:

- **Python version support** — supports running across all Python versions.
- **Maintenance and configuration** — ongoing upkeep and configuration of the baseline agent.

## What `sv-env-setup` does

The `sv-env-setup` branch automates the following workflow for any Python repository:

1. **Install dependencies** — automatically resolves and installs repository dependencies
2. **Reproduce the testing workflow** — discovers and runs the project's test suite
3. **Build a Docker image** — invokes Docker commands to create a new image with the successful installation and testing steps baked in

## Origin

This project is built on top of [SWE-agent](https://github.com/SWE-agent/SWE-agent), an academic project from Princeton University and Stanford University that enables LMs to autonomously use tools to fix issues in real GitHub repositories.

## License

MIT. See `LICENSE`.
