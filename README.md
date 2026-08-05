# AI Agents and n8n Immersion

![n8n](https://img.shields.io/badge/n8n-Automation-EA4B71?style=for-the-badge&logo=n8n&logoColor=white)
![Artificial Intelligence](https://img.shields.io/badge/AI-Agents-6C63FF?style=for-the-badge)
![Docker](https://img.shields.io/badge/Docker-Local%20Environment-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Status](https://img.shields.io/badge/Status-In%20Development-F2C94C?style=for-the-badge)

A collection of projects developed during Hashtag Treinamentos' **AI Agents and n8n Immersion**. Each lesson produces an independent project that is documented and adapted to a distinct scenario.

This repository contains only original implementations, sanitized workflows, project-specific prompts, and technical documentation. Course handouts, answer keys, and proprietary teaching materials are not reproduced.

## Projects

| Lesson | Project | Technologies | Status |
|---|---|---|---|
| 1 | [AI agent for initial email support](projetos/projeto-01-agente-email/) | n8n, Gmail, Google Gemini, Docker | Completed |
| 2 | Coming soon | — | Planned |
| 3 | Coming soon | — | Planned |
| 4 | Coming soon | — | Planned |

## Project 1

The first project monitors Gmail messages, applies a filter condition, and uses a Google Gemini agent to draft initial customer-support responses. The implementation was adapted for web-design inquiries.

Key concepts:

- Gmail, n8n, and Google Gemini integration
- A prompt with clear limits on commercial authority
- Gmail-compatible HTML output
- Conversation context separated by `threadId`
- Protections against malicious instructions embedded in email content
- Workflow sanitization before publication

![First project workflow](assets/imagens/primeiro_projeto.png)

Read the complete documentation in [`projetos/projeto-01-agente-email`](projetos/projeto-01-agente-email/).

## Repository Structure

```text
.
├── assets/
│   └── imagens/
├── docs/
│   └── abrir-projeto-n8n-com-docker.md
├── projetos/
│   └── projeto-01-agente-email/
│       ├── prompt/
│       └── workflow/
├── .gitignore
└── README.md
```

## Local Execution

For instructions on running the workflows with Docker Desktop, see:

- [Run and import n8n workflows with Docker](docs/abrir-projeto-n8n-com-docker.md)

## Study Materials

<https://drive.google.com/drive/folders/1k5Efky38YB1H9qrVbMEbBb3o1iOBUFVf?usp=drive_link>

## Security

Never commit credentials, tokens, API keys, or real messages. Published workflows are reviewed to remove execution data, credential references, and personal information.

## Author

Projects developed and adapted by **Liane Ferreira Heidemann** for learning and portfolio work in automation, systems integration, and AI agents.
