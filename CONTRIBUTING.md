# Contributing to Architecture-Driven-Scalable-System-Design

Thank you for your interest in contributing! This document provides guidelines and instructions for contributing to this project.

## Table of Contents
- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [How to Contribute](#how-to-contribute)
- [Development Workflow](#development-workflow)
- [Coding Standards](#coding-standards)
- [Commit Guidelines](#commit-guidelines)
- [Pull Request Process](#pull-request-process)

---

## Code of Conduct

### Our Pledge

We are committed to providing a welcoming and inclusive environment for all contributors, regardless of experience level, background, or identity.

### Expected Behavior

- Be respectful and considerate
- Use welcoming and inclusive language
- Accept constructive criticism gracefully
- Focus on what's best for the community
- Show empathy towards other community members

### Unacceptable Behavior

- Harassment, discrimination, or offensive comments
- Trolling or insulting/derogatory comments
- Publishing others' private information
- Unprofessional conduct

---

## Getting Started

### Prerequisites

Before contributing, ensure you have:
- Go 1.21+ installed
- MongoDB running locally or Atlas account
- Node.js 18+ (for frontend)
- Git installed and configured
- A GitHub account

### Fork and Clone

1. **Fork the repository** on GitHub
2. **Clone your fork**:
   ```bash
   git clone https://github.com/YOUR_USERNAME/Architecture-Driven-Scalable-System-Design.git
   cd Architecture-Driven-Scalable-System-Design
   ```
3. **Add upstream remote**:
   ```bash
   git remote add upstream https://github.com/Nsanjayboruds/Architecture-Driven-Scalable-System-Design.git
   ```

### Setup Development Environment

Follow the [Quick Start Guide](QUICK_START.md) to set up your local development environment.

---

## How to Contribute

### Reporting Bugs

If you find a bug, please create an issue with:
- **Clear title** describing the bug
- **Steps to reproduce** the issue
- **Expected behavior** vs actual behavior
- **Screenshots** (if applicable)
- **Environment details** (OS, Go version, browser)

**Example**:
```markdown
**Bug**: Authentication fails with valid credentials

**Steps to Reproduce**:
1. Navigate to login page
2. Enter valid email and password
3. Click "Login" button

**Expected**: User should be logged in
**Actual**: Error message "Invalid credentials"

**Environment**: Ubuntu 22.04, Go 1.21, Chrome 120
```

### Suggesting Enhancements

For feature requests or enhancements:
- **Check existing issues** to avoid duplicates
- **Describe the feature** clearly
- **Explain the use case** and benefits
- **Provide examples** or mockups if possible

### Documentation Improvements

Documentation contributions are highly valued! You can:
- Fix typos or clarify existing docs
- Add examples or tutorials
- Improve API documentation
- Translate documentation

---

## Development Workflow

### 1. Create a Branch

Always create a new branch for your work:

```bash
git checkout -b feature/your-feature-name
# or
git checkout -b fix/bug-description
# or
git checkout -b docs/documentation-update
```

**Branch Naming Convention**:
- `feature/` - New features
- `fix/` - Bug fixes
- `docs/` - Documentation changes
- `refactor/` - Code refactoring
- `test/` - Test additions/updates
- `chore/` - Maintenance tasks

### 2. Make Changes

- Write clean, readable code
- Follow the [Coding Standards](#coding-standards)
- Add tests for new features
- Update documentation as needed

### 3. Test Your Changes

**Backend (Go)**:
```bash
cd backend
go test ./...
go test -cover ./...
```

**Frontend**:
```bash
cd frontend
npm test
npm run lint
```

### 4. Commit Your Changes

Follow the [Commit Guidelines](#commit-guidelines):

```bash
git add .
git commit -m "feat: add user profile update feature"
```

### 5. Keep Your Branch Updated

```bash
git fetch upstream
git rebase upstream/main
```

### 6. Push to Your Fork

```bash
git push origin feature/your-feature-name
```

---

## Coding Standards

### Go Backend

**Follow Go conventions**:
- Use `gofmt` for formatting: `go fmt ./...`
- Use `goimports` for imports: `goimports -w .`
- Run linter: `golangci-lint run`

**Code Style**:
```go
// Good: Clear function names, proper error handling
func GetUserByID(ctx context.Context, userID string) (*models.User, error) {
    user, err := repository.FindUser(ctx, userID)
    if err != nil {
        return nil, fmt.Errorf("failed to get user: %w", err)
    }
    return user, nil
}

// Bad: Unclear names, ignored errors
func get(id string) *models.User {
    user, _ := repository.FindUser(context.Background(), id)
    return user
}
```

**Best Practices**:
- Use context for cancellation
- Return errors, don't panic
- Use interfaces for dependencies
- Write table-driven tests
- Add comments for exported functions

### Frontend (React)

**Code Style**:
- Use TypeScript for type safety
- Use functional components with hooks
- Follow React best practices
- Use Tailwind CSS for styling

**Example**:
```typescript
// Good: Typed props, clear naming
interface UserProfileProps {
  userId: string;
  onUpdate: (user: User) => void;
}

const UserProfile: React.FC<UserProfileProps> = ({ userId, onUpdate }) => {
  const [user, setUser] = useState<User | null>(null);
  
  // Implementation
};
```

### File Naming

- Go files: `snake_case.go` (e.g., `user_handler.go`)
- React components: `PascalCase.tsx` (e.g., `UserProfile.tsx`)
- Utilities: `camelCase.ts` (e.g., `formatDate.ts`)

---

## Commit Guidelines

We follow **Conventional Commits** specification.

### Commit Message Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types

- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, missing semi-colons)
- `refactor`: Code refactoring
- `perf`: Performance improvements
- `test`: Adding or updating tests
- `chore`: Maintenance tasks (dependencies, build)
- `ci`: CI/CD changes

### Examples

```bash
# Feature
feat(auth): add JWT token refresh functionality

# Bug fix
fix(api): handle null pointer in user profile endpoint

# Documentation
docs(readme): update installation instructions

# Breaking change
feat(api)!: change authentication response format

BREAKING CHANGE: The auth endpoint now returns { accessToken, refreshToken }
instead of { token }
```

### Rules

- Use imperative mood ("add" not "added" or "adds")
- Don't capitalize first letter
- No period at the end
- Keep subject line under 50 characters
- Wrap body at 72 characters
- Reference issues: `Closes #123` or `Fixes #456`

---

## Pull Request Process

### Before Submitting

✅ Checklist:
- [ ] Code follows project style guidelines
- [ ] All tests pass locally
- [ ] Added tests for new features
- [ ] Updated documentation
- [ ] Commit messages follow conventions
- [ ] Branch is up to date with `main`

### Creating a Pull Request

1. **Push your branch** to your fork
2. **Navigate to the original repository** on GitHub
3. **Click "New Pull Request"**
4. **Select your branch** to compare
5. **Fill out the PR template**:

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
How was this tested?

## Screenshots (if applicable)

## Checklist
- [ ] My code follows the project's style guidelines
- [ ] I have performed a self-review
- [ ] I have commented my code where necessary
- [ ] I have updated the documentation
- [ ] My changes generate no new warnings
- [ ] I have added tests
- [ ] All tests pass locally

## Related Issues
Closes #issue_number
```

### Review Process

1. **Automated checks** will run (tests, linting)
2. **Maintainers will review** your code
3. **Address feedback** by pushing new commits
4. Once approved, a **maintainer will merge** your PR

### After Your PR is Merged

1. **Delete your branch**:
   ```bash
   git branch -d feature/your-feature-name
   git push origin --delete feature/your-feature-name
   ```

2. **Update your local repository**:
   ```bash
   git checkout main
   git pull upstream main
   ```

---

## Development Tips

### Running Tests

```bash
# Backend - Run all tests
cd backend
go test ./...

# Backend - Run with coverage
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out

# Backend - Run specific package
go test ./internal/handlers

# Frontend - Run tests
cd frontend
npm test

# Frontend - Watch mode
npm test -- --watch
```

### Debugging

**Backend**:
```bash
# Use delve debugger
go install github.com/go-delve/delve/cmd/dlv@latest
dlv debug ./cmd/server/main.go
```

**Frontend**:
- Use browser DevTools
- React DevTools extension
- Console.log / debugger statements

### Hot Reload

**Backend** (using air):
```bash
# Install air
go install github.com/cosmtrek/air@latest

# Run with hot reload
air
```

**Frontend**:
```bash
npm run dev  # Vite has built-in HMR
```

---

## Questions or Need Help?

- **Email**: nishantborude555@gmail.com
- **GitHub Issues**: Open an issue for questions
- **Documentation**: Check [docs/](docs/) folder

---

## Recognition

Contributors will be recognized in:
- Project README
- Release notes
- GitHub contributors page

Thank you for contributing! 🎉

---

**Happy Coding!** 💻
