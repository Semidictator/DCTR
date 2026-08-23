# Contributing to The Dictator's Republic ($DCTR)

Thank you for your interest in contributing to DCTR! We're building a satirical yet functional decentralized intelligence network, and community contributions are essential to our mission.

---

## Code of Conduct

This project adheres to a Code of Conduct aimed at fostering a welcoming and inclusive community. By participating, you agree to uphold these standards. See `CODE_OF_CONDUCT.md` for details.

---

## How to Contribute

### 1. **Reporting Bugs**
- Use the **Bug Report** issue template.
- Include steps to reproduce, expected behavior, and actual behavior.
- Provide environment details (OS, Node.js version, etc.).

### 2. **Suggesting Features**
- Use the **Feature Request** issue template.
- Explain the problem this feature solves.
- Provide mockups or examples if applicable.

### 3. **Submitting Pull Requests**

#### Prerequisites
- Fork the repository.
- Clone your fork locally.
- Install dependencies: `npm install`.
- Create a new branch: `git checkout -b feature/your-feature-name`.

#### Development Workflow
1. Write code following TypeScript best practices.
2. Ensure your code adheres to the linting rules: `npm run lint`.
3. Write or update tests as needed: `npm run test`.
4. Commit messages should be clear and descriptive:
   - ✅ `feat: add rescue command with DexScreener integration`
   - ❌ `fix stuff`

#### Pull Request Guidelines
- Reference related issues in your PR description.
- Provide a clear description of changes and why they're needed.
- Ensure all tests pass: `npm run test`.
- Request review from maintainers.

---

## Development Setup

### Prerequisites
- **Node.js** >= 18.0.0
- **npm** >= 9.0.0
- **Telegram Bot Token** (from BotFather)
- **Solana RPC URL** (Devnet or Mainnet)

### Installation
```bash
git clone https://github.com/Semidictator/DCTR.git
cd DCTR
npm install
```

### Configuration
1. Copy `.env.example` to `.env`:
   ```bash
   cp .env.example .env
   ```
2. Fill in your credentials:
   - `TELEGRAM_BOT_TOKEN`: Get from [@BotFather](https://t.me/BotFather) on Telegram.
   - `SOLANA_RPC_URL`: Use Devnet for testing or Mainnet for production.

### Running the Bot Locally
```bash
npm run dev
```

The bot should log: `Bot started. Listening for updates...`

---

## Project Structure

```
DCTR/
├── src/
│   ├── bot.ts                 # Main bot initialization & command handlers
│   ├── services/
│   │   ├── dexscreener.ts     # DexScreener API integration
│   │   ├── solana.ts          # Solana blockchain utilities
│   │   └── rank.ts            # User rank management
│   ├── types/
│   │   └── index.ts           # TypeScript interfaces & types
│   ├── utils/
│   │   ├── logger.ts          # Logging utility
│   │   └── constants.ts       # App constants & emojis
│   └── index.ts               # Entry point
├── .env.example               # Environment variables template
├── package.json               # Dependencies & scripts
├── tsconfig.json              # TypeScript configuration
├── eslintrc.json              # ESLint rules
└── README.md                  # Project overview
```

---

## Coding Standards

### TypeScript
- Use strict mode: `"strict": true` in `tsconfig.json`.
- Type all function parameters and returns.
- Use interfaces for complex objects.

### Naming Conventions
- **Functions**: `camelCase` (e.g., `fetchTokenMetrics()`)
- **Classes**: `PascalCase` (e.g., `DexScreenerService`)
- **Constants**: `UPPER_SNAKE_CASE` (e.g., `BOT_START_MESSAGE`)
- **Files**: `kebab-case` for components, `camelCase` for utilities.

### Error Handling
- Always wrap async operations in try-catch.
- Log errors with context: `logger.error('Failed to fetch token stats', { contractAddress, error })`.
- Return meaningful error messages to users.

---

## Testing

We use **Jest** for unit and integration tests.

```bash
# Run all tests
npm run test

# Run tests in watch mode
npm run test:watch

# Generate coverage report
npm run test:coverage
```

### Writing Tests
```typescript
describe('DexScreenerService', () => {
  it('should fetch token metrics successfully', async () => {
    const metrics = await dexscreener.fetchTokenMetrics('contractAddress');
    expect(metrics).toHaveProperty('price');
  });
});
```

---

## Commit Conventions

We follow [Conventional Commits](https://www.conventionalcommits.org/):

- `feat:` A new feature
- `fix:` A bug fix
- `docs:` Documentation changes
- `style:` Code style changes (formatting, missing semicolons, etc.)
- `refactor:` Code refactoring without feature changes
- `perf:` Performance improvements
- `test:` Adding or updating tests
- `chore:` Dependency updates, configuration changes

Example:
```
feat(bot): add /rescue command with inline voting buttons

- Fetch live token stats from DexScreener API
- Display metrics in formatted Telegram card
- Add HOLD, AVERAGE, EXIT voting buttons

Closes #42
```

---

## Release Process

1. Update version in `package.json`.
2. Update `CHANGELOG.md` with changes.
3. Create a git tag: `git tag -a v1.0.0 -m "Release v1.0.0"`.
4. Push to GitHub: `git push origin main --tags`.
5. GitHub Actions will automatically build and deploy.

---

## Community & Support

- **Discussions**: [GitHub Discussions](https://github.com/Semidictator/DCTR/discussions)
- **Issues**: [Report bugs or request features](https://github.com/Semidictator/DCTR/issues)
- **Telegram**: [@DCTR_Community](https://t.me/DCTR_Community) (coming soon)
- **Discord**: [DCTR Server](https://discord.gg/dctr) (coming soon)

---

## Questions?

Feel free to open an issue with the label `question` or start a discussion. We're here to help!

---

**Thank you for contributing to the revolution! 🚀**
