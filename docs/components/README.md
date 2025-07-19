# Component Documentation

This document provides comprehensive documentation for all public components in RepJC1.

## Table of Contents

- [Overview](#overview)
- [UI Components](#ui-components)
- [Form Components](#form-components)
- [Layout Components](#layout-components)
- [Data Display Components](#data-display-components)
- [Navigation Components](#navigation-components)
- [Utility Components](#utility-components)
- [Component Development Guidelines](#component-development-guidelines)

## Overview

All components in RepJC1 follow consistent patterns and conventions:

- **Props**: All components accept typed props with clear interfaces
- **Styling**: Components use CSS modules or styled-components
- **Accessibility**: All components follow WCAG 2.1 guidelines
- **Testing**: Each component includes comprehensive tests

## UI Components

### Button Component

A versatile button component with multiple variants and states.

#### Props

```typescript
interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'tertiary' | 'danger';
  size?: 'small' | 'medium' | 'large';
  disabled?: boolean;
  loading?: boolean;
  icon?: React.ReactNode;
  iconPosition?: 'left' | 'right';
  onClick?: (event: React.MouseEvent<HTMLButtonElement>) => void;
  children: React.ReactNode;
  className?: string;
  type?: 'button' | 'submit' | 'reset';
}
```

#### Usage Examples

```jsx
import { Button } from '@/components/ui/Button';

// Basic button
<Button onClick={() => console.log('Clicked!')}>
  Click me
</Button>

// Primary button with icon
<Button variant="primary" icon={<SaveIcon />}>
  Save Changes
</Button>

// Loading state
<Button loading disabled>
  Submitting...
</Button>

// Danger variant
<Button variant="danger" onClick={handleDelete}>
  Delete Item
</Button>

// Different sizes
<Button size="small">Small</Button>
<Button size="medium">Medium</Button>
<Button size="large">Large</Button>
```

#### Styling

```css
.button {
  padding: 8px 16px;
  border-radius: 4px;
  border: none;
  cursor: pointer;
  transition: all 0.2s ease;
}

.primary {
  background-color: #007bff;
  color: white;
}

.secondary {
  background-color: #6c757d;
  color: white;
}

.danger {
  background-color: #dc3545;
  color: white;
}
```

### Input Component

A flexible input component with validation and multiple types.

#### Props

```typescript
interface InputProps {
  type?: 'text' | 'email' | 'password' | 'number' | 'tel' | 'url';
  placeholder?: string;
  value?: string;
  defaultValue?: string;
  disabled?: boolean;
  required?: boolean;
  label?: string;
  error?: string;
  helperText?: string;
  icon?: React.ReactNode;
  iconPosition?: 'left' | 'right';
  onChange?: (event: React.ChangeEvent<HTMLInputElement>) => void;
  onBlur?: (event: React.FocusEvent<HTMLInputElement>) => void;
  onFocus?: (event: React.FocusEvent<HTMLInputElement>) => void;
  className?: string;
  autoComplete?: string;
}
```

#### Usage Examples

```jsx
import { Input } from '@/components/ui/Input';

// Basic input
<Input 
  placeholder="Enter your name"
  onChange={(e) => setName(e.target.value)}
/>

// Input with label and validation
<Input
  label="Email Address"
  type="email"
  required
  error={emailError}
  value={email}
  onChange={(e) => setEmail(e.target.value)}
/>

// Input with icon
<Input
  icon={<SearchIcon />}
  placeholder="Search..."
  onChange={handleSearch}
/>

// Password input
<Input
  type="password"
  label="Password"
  helperText="Must be at least 8 characters"
  value={password}
  onChange={(e) => setPassword(e.target.value)}
/>
```

### Modal Component

A flexible modal component for displaying content in an overlay.

#### Props

```typescript
interface ModalProps {
  isOpen: boolean;
  onClose: () => void;
  title?: string;
  size?: 'small' | 'medium' | 'large' | 'fullscreen';
  closeOnBackdropClick?: boolean;
  closeOnEscapeKey?: boolean;
  showCloseButton?: boolean;
  children: React.ReactNode;
  className?: string;
  backdropClassName?: string;
}
```

#### Usage Examples

```jsx
import { Modal } from '@/components/ui/Modal';

// Basic modal
<Modal
  isOpen={isModalOpen}
  onClose={() => setIsModalOpen(false)}
  title="Confirm Action"
>
  <p>Are you sure you want to continue?</p>
  <div className="modal-actions">
    <Button onClick={() => setIsModalOpen(false)}>Cancel</Button>
    <Button variant="primary" onClick={handleConfirm}>Confirm</Button>
  </div>
</Modal>

// Large modal with custom content
<Modal
  isOpen={isLargeModalOpen}
  onClose={() => setIsLargeModalOpen(false)}
  size="large"
  title="User Details"
>
  <UserForm user={selectedUser} onSave={handleSave} />
</Modal>

// Fullscreen modal
<Modal
  isOpen={isFullscreenOpen}
  onClose={() => setIsFullscreenOpen(false)}
  size="fullscreen"
>
  <ImageEditor image={selectedImage} />
</Modal>
```

## Form Components

### Form Component

A comprehensive form component with validation and submission handling.

#### Props

```typescript
interface FormProps {
  onSubmit: (data: any) => void | Promise<void>;
  validationSchema?: any; // Yup schema
  initialValues?: Record<string, any>;
  enableReinitialize?: boolean;
  children: React.ReactNode;
  className?: string;
}
```

#### Usage Examples

```jsx
import { Form, Input, Button } from '@/components';
import * as Yup from 'yup';

const validationSchema = Yup.object({
  name: Yup.string().required('Name is required'),
  email: Yup.string().email('Invalid email').required('Email is required'),
  age: Yup.number().min(18, 'Must be at least 18').required('Age is required')
});

<Form
  onSubmit={handleSubmit}
  validationSchema={validationSchema}
  initialValues={{ name: '', email: '', age: '' }}
>
  <Input name="name" label="Full Name" />
  <Input name="email" label="Email Address" type="email" />
  <Input name="age" label="Age" type="number" />
  <Button type="submit">Submit</Button>
</Form>
```

### Select Component

A dropdown select component with search and multi-select capabilities.

#### Props

```typescript
interface SelectProps {
  options: Option[];
  value?: string | string[];
  defaultValue?: string | string[];
  placeholder?: string;
  multiple?: boolean;
  searchable?: boolean;
  disabled?: boolean;
  loading?: boolean;
  label?: string;
  error?: string;
  onChange?: (value: string | string[]) => void;
  onSearch?: (query: string) => void;
  className?: string;
}

interface Option {
  value: string;
  label: string;
  disabled?: boolean;
}
```

#### Usage Examples

```jsx
import { Select } from '@/components/ui/Select';

const options = [
  { value: 'us', label: 'United States' },
  { value: 'ca', label: 'Canada' },
  { value: 'mx', label: 'Mexico' }
];

// Basic select
<Select
  options={options}
  placeholder="Select a country"
  onChange={(value) => setCountry(value)}
/>

// Multi-select with search
<Select
  options={skillOptions}
  multiple
  searchable
  placeholder="Select skills"
  value={selectedSkills}
  onChange={setSelectedSkills}
/>

// Loading state
<Select
  options={[]}
  loading
  placeholder="Loading options..."
/>
```

## Layout Components

### Grid Component

A flexible grid system for responsive layouts.

#### Props

```typescript
interface GridProps {
  container?: boolean;
  item?: boolean;
  spacing?: number;
  direction?: 'row' | 'column';
  justify?: 'flex-start' | 'center' | 'flex-end' | 'space-between' | 'space-around';
  align?: 'flex-start' | 'center' | 'flex-end' | 'stretch';
  xs?: number | 'auto';
  sm?: number | 'auto';
  md?: number | 'auto';
  lg?: number | 'auto';
  xl?: number | 'auto';
  children: React.ReactNode;
  className?: string;
}
```

#### Usage Examples

```jsx
import { Grid } from '@/components/layout/Grid';

// Grid container with items
<Grid container spacing={2}>
  <Grid item xs={12} md={6}>
    <Card>Content 1</Card>
  </Grid>
  <Grid item xs={12} md={6}>
    <Card>Content 2</Card>
  </Grid>
  <Grid item xs={12}>
    <Card>Full width content</Card>
  </Grid>
</Grid>

// Centered content
<Grid container justify="center" align="center" style={{ minHeight: '100vh' }}>
  <Grid item xs={12} md={4}>
    <LoginForm />
  </Grid>
</Grid>
```

### Container Component

A container component for consistent page layouts.

#### Props

```typescript
interface ContainerProps {
  maxWidth?: 'sm' | 'md' | 'lg' | 'xl' | false;
  fluid?: boolean;
  children: React.ReactNode;
  className?: string;
}
```

#### Usage Examples

```jsx
import { Container } from '@/components/layout/Container';

// Standard container
<Container maxWidth="lg">
  <h1>Page Title</h1>
  <p>Page content...</p>
</Container>

// Fluid container
<Container fluid>
  <FullWidthComponent />
</Container>
```

## Data Display Components

### Table Component

A comprehensive table component with sorting, filtering, and pagination.

#### Props

```typescript
interface TableProps {
  data: any[];
  columns: Column[];
  loading?: boolean;
  pagination?: PaginationConfig;
  sorting?: SortingConfig;
  filtering?: FilteringConfig;
  selection?: SelectionConfig;
  onRowClick?: (row: any) => void;
  className?: string;
}

interface Column {
  key: string;
  title: string;
  render?: (value: any, row: any) => React.ReactNode;
  sortable?: boolean;
  filterable?: boolean;
  width?: string | number;
}
```

#### Usage Examples

```jsx
import { Table } from '@/components/ui/Table';

const columns = [
  { key: 'name', title: 'Name', sortable: true },
  { key: 'email', title: 'Email', filterable: true },
  { 
    key: 'status', 
    title: 'Status',
    render: (status) => <Badge variant={status}>{status}</Badge>
  },
  {
    key: 'actions',
    title: 'Actions',
    render: (_, row) => (
      <Button size="small" onClick={() => editUser(row.id)}>
        Edit
      </Button>
    )
  }
];

<Table
  data={users}
  columns={columns}
  loading={isLoading}
  pagination={{
    page: currentPage,
    pageSize: 10,
    total: totalUsers,
    onChange: setCurrentPage
  }}
  sorting={{
    field: sortField,
    direction: sortDirection,
    onChange: handleSort
  }}
  onRowClick={(row) => navigate(`/users/${row.id}`)}
/>
```

### Card Component

A flexible card component for displaying content.

#### Props

```typescript
interface CardProps {
  title?: string;
  subtitle?: string;
  image?: string;
  actions?: React.ReactNode;
  padding?: 'none' | 'small' | 'medium' | 'large';
  elevation?: number;
  clickable?: boolean;
  onClick?: () => void;
  children: React.ReactNode;
  className?: string;
}
```

#### Usage Examples

```jsx
import { Card } from '@/components/ui/Card';

// Basic card
<Card title="Card Title" subtitle="Card subtitle">
  <p>Card content goes here.</p>
</Card>

// Card with image and actions
<Card
  title="Product Name"
  image="/product-image.jpg"
  actions={
    <div>
      <Button size="small">View</Button>
      <Button size="small" variant="primary">Buy</Button>
    </div>
  }
>
  <p>Product description...</p>
</Card>

// Clickable card
<Card
  title="Clickable Card"
  clickable
  onClick={() => navigate('/details')}
>
  <p>Click anywhere on this card to navigate.</p>
</Card>
```

## Navigation Components

### Navbar Component

A responsive navigation component.

#### Props

```typescript
interface NavbarProps {
  brand?: React.ReactNode;
  items?: NavItem[];
  actions?: React.ReactNode;
  sticky?: boolean;
  transparent?: boolean;
  className?: string;
}

interface NavItem {
  label: string;
  href?: string;
  onClick?: () => void;
  active?: boolean;
  disabled?: boolean;
  children?: NavItem[];
}
```

#### Usage Examples

```jsx
import { Navbar } from '@/components/navigation/Navbar';

const navItems = [
  { label: 'Home', href: '/' },
  { label: 'Products', href: '/products' },
  { 
    label: 'Services', 
    children: [
      { label: 'Consulting', href: '/services/consulting' },
      { label: 'Support', href: '/services/support' }
    ]
  },
  { label: 'Contact', href: '/contact' }
];

<Navbar
  brand={<Logo />}
  items={navItems}
  actions={
    <div>
      <Button variant="secondary">Login</Button>
      <Button variant="primary">Sign Up</Button>
    </div>
  }
  sticky
/>
```

### Breadcrumb Component

A breadcrumb navigation component.

#### Props

```typescript
interface BreadcrumbProps {
  items: BreadcrumbItem[];
  separator?: React.ReactNode;
  className?: string;
}

interface BreadcrumbItem {
  label: string;
  href?: string;
  onClick?: () => void;
  active?: boolean;
}
```

#### Usage Examples

```jsx
import { Breadcrumb } from '@/components/navigation/Breadcrumb';

const breadcrumbItems = [
  { label: 'Home', href: '/' },
  { label: 'Products', href: '/products' },
  { label: 'Laptops', href: '/products/laptops' },
  { label: 'MacBook Pro', active: true }
];

<Breadcrumb items={breadcrumbItems} />
```

## Utility Components

### Loading Component

A loading indicator component with multiple variants.

#### Props

```typescript
interface LoadingProps {
  variant?: 'spinner' | 'dots' | 'bars' | 'pulse';
  size?: 'small' | 'medium' | 'large';
  color?: string;
  overlay?: boolean;
  text?: string;
  className?: string;
}
```

#### Usage Examples

```jsx
import { Loading } from '@/components/ui/Loading';

// Basic spinner
<Loading />

// Loading with text
<Loading text="Loading data..." />

// Overlay loading
<Loading overlay variant="pulse" text="Please wait..." />

// Different variants
<Loading variant="dots" size="large" />
<Loading variant="bars" color="#007bff" />
```

### ErrorBoundary Component

An error boundary component for handling React errors.

#### Props

```typescript
interface ErrorBoundaryProps {
  fallback?: React.ComponentType<{ error: Error; resetError: () => void }>;
  onError?: (error: Error, errorInfo: React.ErrorInfo) => void;
  children: React.ReactNode;
}
```

#### Usage Examples

```jsx
import { ErrorBoundary } from '@/components/utility/ErrorBoundary';

// Basic error boundary
<ErrorBoundary>
  <MyComponent />
</ErrorBoundary>

// Custom fallback component
<ErrorBoundary
  fallback={({ error, resetError }) => (
    <div>
      <h2>Something went wrong</h2>
      <p>{error.message}</p>
      <Button onClick={resetError}>Try again</Button>
    </div>
  )}
  onError={(error, errorInfo) => {
    console.error('Error caught by boundary:', error, errorInfo);
  }}
>
  <MyComponent />
</ErrorBoundary>
```

## Component Development Guidelines

### Creating New Components

1. **File Structure**
```
components/
  ui/
    Button/
      Button.tsx
      Button.module.css
      Button.test.tsx
      index.ts
```

2. **Component Template**
```tsx
import React from 'react';
import styles from './ComponentName.module.css';

interface ComponentNameProps {
  // Define props here
}

export const ComponentName: React.FC<ComponentNameProps> = ({
  // Destructure props
}) => {
  return (
    <div className={styles.container}>
      {/* Component JSX */}
    </div>
  );
};

export default ComponentName;
```

3. **Testing Template**
```tsx
import { render, screen } from '@testing-library/react';
import { ComponentName } from './ComponentName';

describe('ComponentName', () => {
  it('renders correctly', () => {
    render(<ComponentName />);
    expect(screen.getByRole('...')).toBeInTheDocument();
  });
});
```

### Best Practices

1. **Props Design**
   - Use TypeScript interfaces for props
   - Provide sensible defaults
   - Keep prop APIs minimal and focused

2. **Accessibility**
   - Include proper ARIA attributes
   - Ensure keyboard navigation
   - Maintain color contrast ratios

3. **Performance**
   - Use React.memo for pure components
   - Implement proper key props for lists
   - Avoid unnecessary re-renders

4. **Styling**
   - Use CSS modules or styled-components
   - Follow consistent naming conventions
   - Support theming and customization

5. **Documentation**
   - Include comprehensive prop documentation
   - Provide usage examples
   - Document accessibility features