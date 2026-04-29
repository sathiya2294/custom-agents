# custom-agents

A repository for managing and maintaining custom GitHub Copilot agents.

## Overview

This repository serves as the central hub for creating, organizing, and maintaining custom agents built on top of GitHub Copilot's coding agent capabilities.

## Copilot Setup

This repository includes a [`copilot-setup-steps.yml`](.github/workflows/copilot-setup-steps.yml) workflow that configures the Copilot cloud agent environment before it begins working on tasks in this repository.

## Structure

```
.
├── .github/
│   └── workflows/
│       └── copilot-setup-steps.yml   # Copilot agent environment setup
└── README.md
```

## Getting Started

1. Clone this repository.
2. Add your custom agent definitions and configurations.
3. The Copilot setup steps workflow will automatically prepare the environment when Copilot works on tasks in this repository.
