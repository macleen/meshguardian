# Contributing to MeshGuardian

[Full documentation available at https://meshguardian.com](https://meshguardian.com)

We’re thrilled that you’re considering contributing to **MeshGuardian**, a resilient, secure, and modular mesh networking protocol built for Delay-Tolerant Networks (DTNs), off-grid communities, and interplanetary systems. MeshGuardian enables reliable communication in challenging environments, from crisis zones to deep space, using protocols like LoRa, DTN Bundle Protocol, and Asynchronous Delay-Tolerant Consensus (ADTC) with features like multi-blockchain logging and slashing logic for node accountability. Whether you’re a developer, researcher, documenter, or space tech enthusiast, your contributions are vital to shaping the future of decentralized networking.

This guide outlines how to contribute to MeshGuardian, from setting up your environment to submitting code, documentation, or community ideas. We welcome all contributions—code, documentation, testing, bug reports, feature suggestions, and community engagement. Let’s build a vibrant community together!

## Table of Contents
- [Code of Conduct](#code-of-conduct)
- [Ground Rules](#ground-rules)
- [How to Contribute](#how-to-contribute)
  - [Setting Up Your Development Environment](#setting-up-your-development-environment)
  - [Finding Issues to Work On](#finding-issues-to-work-on)
  - [Contribution Types](#contribution-types)
- [Submitting Contributions](#submitting-contributions)
  - [Reporting Bugs](#reporting-bugs)
  - [Suggesting Features](#suggesting-features)
  - [Submitting Pull Requests](#submitting-pull-requests)
- [Contribution Checklist](#contribution-checklist)
- [Coding Standards](#coding-standards)
- [Running Tests](#running-tests)
- [Join the Discussion](#join-the-discussion)
- [Contact Us](#contact-us)

## Code of Conduct
We are committed to fostering an inclusive and respectful community. All contributors must adhere to our [Code of Conduct](CODE_OF_CONDUCT.md), which outlines expectations for respectful communication and procedures for reporting unacceptable behavior. Please read it before contributing.

## Ground Rules
- **Be Respectful and Inclusive**: Treat all contributors with empathy and kindness, valuing diverse perspectives.
- **Follow the Code of Conduct**: Ensure all interactions align with our [Code of Conduct](CODE_OF_CONDUCT.md).
- **Open Issues for Large Changes**: Discuss significant changes via issues before starting work to align with project goals.
- **Write Clear Commit Messages and PR Descriptions**: Use descriptive messages (e.g., `feat: add QUIC protocol plugin`, `fix: correct PBFT timestamp validation`) for clarity.

## How to Contribute

### Setting Up Your Development Environment
To contribute to MeshGuardian, set up your local environment as follows:

1. **Fork the Repository**:
   - Navigate to the [MeshGuardian GitHub repository](https://github.com/macleen/meshguardian).
   - Click **Fork** to create a copy under your GitHub account.

2. **Clone Your Fork**:
   ```bash
   git clone https://github.com/YOUR_USERNAME/meshguardian.git
   cd meshguardian