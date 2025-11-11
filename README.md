# OpenStrand

> AI-native knowledge infrastructure for teams that want to own their data

[![MIT License](https://img.shields.io/badge/License-MIT-green.svg)](https://choosealicense.com/licenses/mit/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.3-blue)](https://www.typescriptlang.org/)
[![Next.js](https://img.shields.io/badge/Next.js-14-black)](https://nextjs.org/)

OpenStrand is a modern personal knowledge management system (PKMS) that combines the power of AI with local-first data ownership. Build your second brain while keeping complete control of your information.

## 🚀 Features

### Core Capabilities
- **Knowledge Graph Visualization** - See connections between your ideas in 2D/3D
- **AI-Powered Intelligence** - Smart tagging, auto-linking, and content suggestions
- **Multi-Format Import** - Support for 20+ file formats including Markdown, PDF, DOCX
- **Local-First Architecture** - Your data stays on your device with optional sync
- **Block-Level Organization** - Tag and connect individual paragraphs, not just documents

### Team Features
- **Real-time Collaboration** - Work together with presence awareness
- **Custom Domains** - Host your knowledge base on your domain
- **API Access** - Build custom integrations with our TypeScript SDK
- **Enterprise SSO** - SAML/OAuth integration for organizations

## 🏗️ Architecture

OpenStrand consists of three main packages:

| Package | Description | Repository |
|---------|-------------|------------|
| **openstrand-app** | Next.js frontend application | [GitHub](https://github.com/framersai/openstrand-app) |
| **@openstrand/teams-backend** | Fastify API server with Prisma | [GitHub](https://github.com/framersai/openstrand-teams-backend) |
| **@openstrand/sdk** | TypeScript SDK for integrations | [npm](https://www.npmjs.com/org/framers) |

## 🎯 Quick Start

### Option 1: Cloud Hosted (Easiest)
Visit [openstrand.ai](https://openstrand.ai) to start with a free account.

### Option 2: Local Development

```bash
# Clone the monorepo
git clone https://github.com/framersai/openstrand-monorepo.git
cd openstrand-monorepo

# Install dependencies
npm install

# Start development servers
npm run dev

# Visit http://localhost:3000
```

### Option 3: Docker

```bash
docker run -p 3000:3000 framersai/openstrand:latest
```

## 📦 Installation

### Using the SDK

```bash
npm install @openstrand/sdk
```

```typescript
import { OpenStrandClient } from '@openstrand/sdk';

const client = new OpenStrandClient({
  apiUrl: 'https://api.openstrand.ai'
});

// Create a strand (unit of knowledge)
const strand = await client.strands.create({
  title: 'My First Strand',
  content: { markdown: '# Hello World' },
  tags: ['getting-started']
});
```

## 🔧 Configuration

### Environment Variables

```env
# Required
DATABASE_URL=postgresql://...
NEXTAUTH_SECRET=your-secret-key

# Optional
OPENAI_API_KEY=sk-...
S3_BUCKET=your-bucket
REDIS_URL=redis://...
```

See our public repo at [github.com/framersai/openstrand](https://github.com/framersai/openstrand) for configuration references.

## 🗺️ Roadmap

### Q1 2025
- [ ] Mobile apps (iOS/Android via Capacitor)
- [ ] Offline-first sync engine
- [ ] Plugin marketplace

### Q2 2025
- [ ] End-to-end encryption
- [ ] Advanced AI features (GPT-4 Vision support)
- [ ] Public knowledge sharing

See our [full roadmap](https://github.com/framersai/openstrand-monorepo/blob/main/docs/ROADMAP.md) for more details.

## 🤝 Contributing

We welcome contributions! Please see our [Contributing Guide](CONTRIBUTING.md) for details.

```bash
# Fork and clone
git clone https://github.com/YOUR_USERNAME/openstrand-monorepo.git

# Create a branch
git checkout -b feature/amazing-feature

# Make changes and test
npm test

# Submit a PR
```

## 📚 Documentation

- [Architecture Overview](ARCHITECTURE.md)
- [API Reference](API_REFERENCE.md)
- [SDK Documentation](SDK_GUIDE.md)

## 💬 Community

- **Discord**: [Join our community](https://discord.gg/openstrandai)
- **Twitter/X**: [@openstrandai](https://twitter.com/openstrandai)
- **Blog**: [openstrand.ai/blog](https://openstrand.ai/blog)

## 📄 License

OpenStrand is MIT licensed. See [LICENSE](LICENSE) for details.

## 🏢 Enterprise

For enterprise features including SSO, SLA support, and on-premise deployment:
- Email: enterprise@frame.dev
- Website: [openstrand.ai/enterprise](https://openstrand.ai/enterprise)

---

Built with ❤️ by [Framers](https://frame.dev)