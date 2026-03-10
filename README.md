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
