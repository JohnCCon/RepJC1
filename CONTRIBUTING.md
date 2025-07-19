# Contributing to RepJC1

Thank you for your interest in contributing to RepJC1! This document provides guidelines and information for contributors.

## Table of Contents

- [Getting Started](#getting-started)
- [Development Setup](#development-setup)
- [Contributing Process](#contributing-process)
- [Code Standards](#code-standards)
- [Testing Guidelines](#testing-guidelines)
- [Documentation Guidelines](#documentation-guidelines)
- [Pull Request Process](#pull-request-process)
- [Issue Reporting](#issue-reporting)

## Getting Started

### Prerequisites

- Node.js (version 14.0 or higher)
- npm or yarn package manager
- Git

### Development Setup

1. **Fork the repository**
   ```bash
   # Click the "Fork" button on GitHub, then clone your fork
   git clone https://github.com/your-username/RepJC1.git
   cd RepJC1
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Create a branch for your work**
   ```bash
   git checkout -b feature/your-feature-name
   # or
   git checkout -b fix/issue-description
   ```

4. **Start the development server**
   ```bash
   npm start
   ```

## Contributing Process

### 1. Choose an Issue

- Look for issues labeled `good first issue` for beginner-friendly tasks
- Check issues labeled `help wanted` for areas where contributions are needed
- Feel free to create new issues for bugs or feature requests

### 2. Discuss Before Large Changes

For significant changes or new features:
- Open an issue first to discuss the approach
- Wait for maintainer feedback before starting work
- This helps ensure your contribution aligns with project goals

### 3. Make Your Changes

- Follow our [code standards](#code-standards)
- Write tests for new functionality
- Update documentation as needed
- Keep commits small and focused

### 4. Submit a Pull Request

- Follow our [pull request process](#pull-request-process)
- Be responsive to feedback
- Update your PR based on review comments

## Code Standards

### TypeScript

We use TypeScript for type safety and better developer experience.

```typescript
// ✅ Good: Proper typing
interface UserData {
  id: number;
  name: string;
  email: string;
  isActive: boolean;
}

function createUser(userData: UserData): Promise<User> {
  return api.post('/users', userData);
}

// ❌ Bad: No typing
function createUser(userData: any): any {
  return api.post('/users', userData);
}
```

### Component Structure

```tsx
// ✅ Good: Proper component structure
import React from 'react';
import styles from './ComponentName.module.css';

interface ComponentNameProps {
  title: string;
  onAction?: () => void;
  children?: React.ReactNode;
}

/**
 * ComponentName - Brief description of what this component does
 * 
 * @param title - The title to display
 * @param onAction - Optional callback for action events
 * @param children - Optional child components
 */
export const ComponentName: React.FC<ComponentNameProps> = ({
  title,
  onAction,
  children
}) => {
  return (
    <div className={styles.container}>
      <h2 className={styles.title}>{title}</h2>
      {children}
      {onAction && (
        <button onClick={onAction} className={styles.actionButton}>
          Action
        </button>
      )}
    </div>
  );
};

export default ComponentName;
```

### Function Documentation

Use JSDoc comments for all public functions:

```typescript
/**
 * Formats a number as currency with locale support
 * 
 * @param amount - The amount to format
 * @param currency - Currency code (default: 'USD')
 * @param locale - Locale string (default: 'en-US')
 * @returns Formatted currency string
 * 
 * @example
 * ```typescript
 * formatCurrency(1234.56); // "$1,234.56"
 * formatCurrency(1234.56, 'EUR', 'de-DE'); // "1.234,56 €"
 * ```
 */
export function formatCurrency(
  amount: number,
  currency: string = 'USD',
  locale: string = 'en-US'
): string {
  return new Intl.NumberFormat(locale, {
    style: 'currency',
    currency
  }).format(amount);
}
```

### Naming Conventions

- **Files**: Use PascalCase for components (`UserCard.tsx`), camelCase for utilities (`formatDate.ts`)
- **Components**: PascalCase (`UserCard`, `DataTable`)
- **Functions**: camelCase (`formatCurrency`, `validateEmail`)
- **Variables**: camelCase (`userName`, `isActive`)
- **Constants**: SCREAMING_SNAKE_CASE (`API_BASE_URL`, `MAX_RETRY_ATTEMPTS`)
- **Types/Interfaces**: PascalCase (`User`, `ApiResponse`)

### CSS Standards

Use CSS Modules for component styling:

```css
/* ComponentName.module.css */
.container {
  padding: 1rem;
  border-radius: 8px;
  background-color: var(--background-color);
}

.title {
  font-size: 1.5rem;
  font-weight: 600;
  margin-bottom: 1rem;
  color: var(--text-primary);
}

.actionButton {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  background-color: var(--primary-color);
  color: white;
  cursor: pointer;
  transition: background-color 0.2s ease;
}

.actionButton:hover {
  background-color: var(--primary-color-dark);
}
```

## Testing Guidelines

### Unit Tests

Write unit tests for all functions and components:

```typescript
// formatCurrency.test.ts
import { formatCurrency } from './formatCurrency';

describe('formatCurrency', () => {
  it('formats USD currency correctly', () => {
    expect(formatCurrency(1234.56)).toBe('$1,234.56');
  });

  it('formats EUR currency correctly', () => {
    expect(formatCurrency(1234.56, 'EUR', 'de-DE')).toBe('1.234,56 €');
  });

  it('handles zero amount', () => {
    expect(formatCurrency(0)).toBe('$0.00');
  });

  it('handles negative amounts', () => {
    expect(formatCurrency(-1234.56)).toBe('-$1,234.56');
  });
});
```

### Component Tests

```tsx
// UserCard.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { UserCard } from './UserCard';

const mockUser = {
  id: 1,
  name: 'John Doe',
  email: 'john@example.com',
  isActive: true
};

describe('UserCard', () => {
  it('renders user information correctly', () => {
    render(<UserCard user={mockUser} />);
    
    expect(screen.getByText('John Doe')).toBeInTheDocument();
    expect(screen.getByText('john@example.com')).toBeInTheDocument();
  });

  it('calls onEdit when edit button is clicked', () => {
    const onEdit = jest.fn();
    render(<UserCard user={mockUser} onEdit={onEdit} />);
    
    fireEvent.click(screen.getByText('Edit'));
    expect(onEdit).toHaveBeenCalledWith(mockUser);
  });

  it('shows active status correctly', () => {
    render(<UserCard user={mockUser} />);
    
    expect(screen.getByText('Active')).toBeInTheDocument();
    expect(screen.getByText('Active')).toHaveClass('status-active');
  });
});
```

### Test Coverage

- Aim for at least 80% test coverage
- Focus on testing public APIs and critical business logic
- Include edge cases and error scenarios
- Use meaningful test descriptions

### Running Tests

```bash
# Run all tests
npm test

# Run tests in watch mode
npm test -- --watch

# Run tests with coverage
npm test -- --coverage

# Run specific test file
npm test -- UserCard.test.tsx
```

## Documentation Guidelines

### API Documentation

Document all public APIs with comprehensive examples:

```typescript
/**
 * Creates a new user in the system
 * 
 * @param userData - User information
 * @param options - Additional options for user creation
 * @returns Promise that resolves to the created user
 * 
 * @throws {ValidationError} When user data is invalid
 * @throws {DuplicateEmailError} When email already exists
 * 
 * @example
 * ```typescript
 * const user = await createUser({
 *   name: 'John Doe',
 *   email: 'john@example.com'
 * });
 * console.log('Created user:', user.id);
 * ```
 * 
 * @example
 * ```typescript
 * // With options
 * const user = await createUser(
 *   { name: 'John Doe', email: 'john@example.com' },
 *   { sendWelcomeEmail: true }
 * );
 * ```
 */
```

### Component Documentation

Document component props and usage:

```tsx
/**
 * UserCard - Displays user information in a card format
 * 
 * @component
 * @example
 * ```tsx
 * <UserCard
 *   user={user}
 *   onEdit={handleEdit}
 *   onDelete={handleDelete}
 *   showActions={true}
 * />
 * ```
 */
export const UserCard: React.FC<UserCardProps> = ({ ... }) => {
  // Component implementation
};
```

### Updating Documentation

When making changes:
- Update relevant documentation files
- Add examples for new features
- Update JSDoc comments for changed APIs
- Run documentation generation to check for errors

## Pull Request Process

### Before Submitting

1. **Ensure code quality**
   ```bash
   npm run lint
   npm run type-check
   npm test
   npm run build
   ```

2. **Update documentation**
   - Update relevant README sections
   - Add or update API documentation
   - Include examples for new features

3. **Write clear commit messages**
   ```bash
   # Good commit messages
   feat: add user management API
   fix: resolve memory leak in data processing
   docs: update component documentation
   test: add tests for validation functions
   
   # Bad commit messages
   Update code
   Fix bug
   Changes
   ```

### Pull Request Template

Use this template for your pull requests:

```markdown
## Description
Brief description of what this PR does.

## Type of Change
- [ ] Bug fix (non-breaking change which fixes an issue)
- [ ] New feature (non-breaking change which adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to not work as expected)
- [ ] Documentation update

## Testing
- [ ] Tests pass locally
- [ ] Added tests for new functionality
- [ ] Updated documentation

## Screenshots (if applicable)
Add screenshots to help explain your changes.

## Checklist
- [ ] My code follows the style guidelines
- [ ] I have performed a self-review of my code
- [ ] I have commented my code, particularly in hard-to-understand areas
- [ ] I have made corresponding changes to the documentation
- [ ] My changes generate no new warnings
- [ ] I have added tests that prove my fix is effective or that my feature works
- [ ] New and existing unit tests pass locally with my changes
```

### Review Process

1. **Automated checks** must pass (linting, tests, build)
2. **Code review** by at least one maintainer
3. **Documentation review** for user-facing changes
4. **Testing** in development environment
5. **Approval** and merge by maintainer

## Issue Reporting

### Bug Reports

Use the bug report template:

```markdown
## Bug Description
A clear and concise description of what the bug is.

## Steps to Reproduce
1. Go to '...'
2. Click on '....'
3. Scroll down to '....'
4. See error

## Expected Behavior
A clear and concise description of what you expected to happen.

## Actual Behavior
A clear and concise description of what actually happened.

## Screenshots
If applicable, add screenshots to help explain your problem.

## Environment
- OS: [e.g. macOS, Windows, Linux]
- Browser [e.g. chrome, safari]
- Version [e.g. 22]

## Additional Context
Add any other context about the problem here.
```

### Feature Requests

Use the feature request template:

```markdown
## Feature Description
A clear and concise description of what you want to happen.

## Use Case
Describe the use case or problem this feature would solve.

## Proposed Solution
A clear and concise description of what you want to happen.

## Alternatives Considered
A clear and concise description of any alternative solutions or features you've considered.

## Additional Context
Add any other context or screenshots about the feature request here.
```

## Code of Conduct

### Our Standards

- Use welcoming and inclusive language
- Be respectful of differing viewpoints and experiences
- Gracefully accept constructive criticism
- Focus on what is best for the community
- Show empathy towards other community members

### Unacceptable Behavior

- Harassment of any kind
- Discriminatory language or behavior
- Personal attacks or insults
- Publishing others' private information
- Other conduct which could reasonably be considered inappropriate

### Enforcement

Instances of abusive, harassing, or otherwise unacceptable behavior may be reported to the project maintainers. All complaints will be reviewed and investigated promptly and fairly.

## Getting Help

- **Documentation**: Check the [docs](docs/) directory
- **Issues**: Search existing issues before creating new ones
- **Discussions**: Use GitHub Discussions for questions
- **Email**: Contact maintainers directly for security issues

## Recognition

Contributors will be recognized in the following ways:
- Listed in CONTRIBUTORS.md
- Mentioned in release notes for significant contributions
- Invited to join the core team for exceptional contributors

Thank you for contributing to RepJC1!