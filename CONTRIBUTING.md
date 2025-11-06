# Contributing to OpenStrand

Thank you for your interest in contributing to OpenStrand! We welcome contributions from the community.

## Getting Started

1. **Fork the repository**
   ```bash
   git clone https://github.com/YOUR_USERNAME/openstrand-monorepo.git
   cd openstrand-monorepo
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Start development**
   ```bash
   ./start-local.sh
   ```

## Development Workflow

1. Create a feature branch
   ```bash
   git checkout -b feature/amazing-feature
   ```

2. Make your changes following our conventions

3. Test your changes
   ```bash
   npm test
   npm run type-check
   npm run lint
   ```

4. Commit using conventional commits
   ```bash
   git commit -m "feat: add amazing feature"
   ```

5. Push and create a Pull Request

## Commit Message Convention

We use conventional commits:

- `feat:` New feature
- `fix:` Bug fix
- `docs:` Documentation changes
- `chore:` Maintenance tasks
- `refactor:` Code refactoring
- `test:` Test additions/changes
- `perf:` Performance improvements

## Code Style

- TypeScript with strict mode enabled
- Prettier for formatting (auto-applied)
- ESLint for linting
- Follow existing patterns in the codebase

## Testing

- Write tests for new features
- Ensure existing tests pass
- Aim for good coverage of critical paths

## Pull Request Process

1. Update documentation if needed
2. Add tests for new functionality
3. Ensure CI passes
4. Request review from maintainers
5. Address feedback promptly

## Areas We Need Help

- 📱 Mobile app improvements (Capacitor)
- 🌐 Internationalization (i18n)
- 📚 Documentation and tutorials
- 🧪 Test coverage
- 🎨 UI/UX improvements
- 🔌 Plugin system
- 🚀 Performance optimizations

## Questions?

- Open an issue for bugs/features
- Join our [Discord](https://discord.gg/openstrand) for discussions
- Email: team@frame.dev for security issues

## License

By contributing, you agree that your contributions will be licensed under the MIT License.
