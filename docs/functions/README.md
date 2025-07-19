# Function Reference

This document provides comprehensive documentation for all public functions in RepJC1.

## Table of Contents

- [Overview](#overview)
- [Utility Functions](#utility-functions)
- [Data Processing Functions](#data-processing-functions)
- [Authentication Functions](#authentication-functions)
- [Validation Functions](#validation-functions)
- [API Client Functions](#api-client-functions)
- [State Management Functions](#state-management-functions)
- [Date and Time Functions](#date-and-time-functions)
- [String Manipulation Functions](#string-manipulation-functions)
- [Array and Object Functions](#array-and-object-functions)
- [File Handling Functions](#file-handling-functions)

## Overview

All functions in RepJC1 follow consistent patterns:

- **TypeScript**: All functions are fully typed with comprehensive interfaces
- **Error Handling**: Functions include proper error handling and validation
- **Testing**: Each function includes unit tests and examples
- **Documentation**: JSDoc comments with parameter and return type documentation

## Utility Functions

### `debounce`

Creates a debounced function that delays invoking the provided function until after wait milliseconds have elapsed since the last time the debounced function was invoked.

```typescript
function debounce<T extends (...args: any[]) => any>(
  func: T,
  wait: number,
  immediate?: boolean
): (...args: Parameters<T>) => void
```

#### Parameters

- `func` (Function): The function to debounce
- `wait` (number): The number of milliseconds to delay
- `immediate` (boolean, optional): If true, trigger the function on the leading edge instead of trailing

#### Returns

- (Function): Returns the new debounced function

#### Example Usage

```typescript
import { debounce } from '@/utils/debounce';

// Debounce search input
const debouncedSearch = debounce((query: string) => {
  console.log('Searching for:', query);
  // Perform search
}, 300);

// Usage in React component
const handleInputChange = (event: React.ChangeEvent<HTMLInputElement>) => {
  debouncedSearch(event.target.value);
};

// Immediate execution example
const debouncedSubmit = debounce(handleSubmit, 1000, true);
```

### `throttle`

Creates a throttled function that only invokes the provided function at most once per specified interval.

```typescript
function throttle<T extends (...args: any[]) => any>(
  func: T,
  limit: number
): (...args: Parameters<T>) => void
```

#### Parameters

- `func` (Function): The function to throttle
- `limit` (number): The number of milliseconds to throttle invocations to

#### Returns

- (Function): Returns the new throttled function

#### Example Usage

```typescript
import { throttle } from '@/utils/throttle';

// Throttle scroll handler
const throttledScrollHandler = throttle(() => {
  console.log('Scroll position:', window.scrollY);
}, 100);

window.addEventListener('scroll', throttledScrollHandler);

// Throttle API calls
const throttledApiCall = throttle(async (data: any) => {
  return await api.post('/endpoint', data);
}, 1000);
```

### `generateId`

Generates a unique identifier string.

```typescript
function generateId(length?: number, prefix?: string): string
```

#### Parameters

- `length` (number, optional): Length of the generated ID (default: 8)
- `prefix` (string, optional): Prefix to add to the generated ID

#### Returns

- (string): A unique identifier string

#### Example Usage

```typescript
import { generateId } from '@/utils/generateId';

// Basic usage
const id = generateId(); // e.g., "a1b2c3d4"

// Custom length
const longId = generateId(16); // e.g., "a1b2c3d4e5f6g7h8"

// With prefix
const userId = generateId(8, 'user_'); // e.g., "user_a1b2c3d4"
```

## Data Processing Functions

### `formatCurrency`

Formats a number as currency with locale support.

```typescript
function formatCurrency(
  amount: number,
  currency?: string,
  locale?: string
): string
```

#### Parameters

- `amount` (number): The amount to format
- `currency` (string, optional): Currency code (default: 'USD')
- `locale` (string, optional): Locale string (default: 'en-US')

#### Returns

- (string): Formatted currency string

#### Example Usage

```typescript
import { formatCurrency } from '@/utils/formatCurrency';

// Basic usage
formatCurrency(1234.56); // "$1,234.56"

// Different currency
formatCurrency(1234.56, 'EUR', 'de-DE'); // "1.234,56 €"

// Japanese Yen
formatCurrency(1234.56, 'JPY', 'ja-JP'); // "¥1,235"
```

### `parseCSV`

Parses CSV data into an array of objects.

```typescript
function parseCSV<T = Record<string, string>>(
  csvData: string,
  options?: {
    delimiter?: string;
    headers?: string[];
    skipEmptyLines?: boolean;
  }
): T[]
```

#### Parameters

- `csvData` (string): The CSV data to parse
- `options` (object, optional): Parsing options
  - `delimiter` (string): Field delimiter (default: ',')
  - `headers` (string[]): Custom headers array
  - `skipEmptyLines` (boolean): Skip empty lines (default: true)

#### Returns

- (T[]): Array of parsed objects

#### Example Usage

```typescript
import { parseCSV } from '@/utils/parseCSV';

const csvData = `
name,email,age
John Doe,john@example.com,30
Jane Smith,jane@example.com,25
`;

// Basic parsing
const users = parseCSV(csvData);
// Result: [
//   { name: 'John Doe', email: 'john@example.com', age: '30' },
//   { name: 'Jane Smith', email: 'jane@example.com', age: '25' }
// ]

// Custom delimiter
const tsvData = "name\temail\tage\nJohn\tjohn@example.com\t30";
const tsvUsers = parseCSV(tsvData, { delimiter: '\t' });

// Custom headers
const dataWithoutHeaders = "John,john@example.com,30\nJane,jane@example.com,25";
const usersWithCustomHeaders = parseCSV(dataWithoutHeaders, {
  headers: ['name', 'email', 'age']
});
```

### `transformData`

Transforms data using a mapping function with support for nested properties.

```typescript
function transformData<T, U>(
  data: T[],
  transformer: (item: T) => U
): U[]

function transformData<T, U>(
  data: T,
  transformer: (item: T) => U
): U
```

#### Parameters

- `data` (T | T[]): Data to transform
- `transformer` (Function): Transformation function

#### Returns

- (U | U[]): Transformed data

#### Example Usage

```typescript
import { transformData } from '@/utils/transformData';

interface User {
  id: number;
  firstName: string;
  lastName: string;
  email: string;
}

interface DisplayUser {
  id: number;
  fullName: string;
  email: string;
}

const users: User[] = [
  { id: 1, firstName: 'John', lastName: 'Doe', email: 'john@example.com' },
  { id: 2, firstName: 'Jane', lastName: 'Smith', email: 'jane@example.com' }
];

// Transform array
const displayUsers = transformData(users, (user) => ({
  id: user.id,
  fullName: `${user.firstName} ${user.lastName}`,
  email: user.email
}));

// Transform single object
const singleUser = transformData(users[0], (user) => ({
  id: user.id,
  fullName: `${user.firstName} ${user.lastName}`,
  email: user.email
}));
```

## Authentication Functions

### `generateToken`

Generates a JWT token with specified payload and options.

```typescript
function generateToken(
  payload: Record<string, any>,
  secret: string,
  options?: {
    expiresIn?: string | number;
    algorithm?: string;
  }
): Promise<string>
```

#### Parameters

- `payload` (object): Token payload
- `secret` (string): Secret key for signing
- `options` (object, optional): Token options
  - `expiresIn` (string | number): Token expiration
  - `algorithm` (string): Signing algorithm (default: 'HS256')

#### Returns

- (Promise<string>): JWT token

#### Example Usage

```typescript
import { generateToken } from '@/auth/generateToken';

// Basic usage
const token = await generateToken(
  { userId: 123, role: 'user' },
  process.env.JWT_SECRET
);

// With expiration
const tokenWithExpiry = await generateToken(
  { userId: 123, role: 'admin' },
  process.env.JWT_SECRET,
  { expiresIn: '24h' }
);

// Custom algorithm
const rsaToken = await generateToken(
  { userId: 123 },
  privateKey,
  { algorithm: 'RS256', expiresIn: '1h' }
);
```

### `verifyToken`

Verifies and decodes a JWT token.

```typescript
function verifyToken(
  token: string,
  secret: string,
  options?: {
    algorithms?: string[];
  }
): Promise<any>
```

#### Parameters

- `token` (string): JWT token to verify
- `secret` (string): Secret key for verification
- `options` (object, optional): Verification options

#### Returns

- (Promise<any>): Decoded token payload

#### Example Usage

```typescript
import { verifyToken } from '@/auth/verifyToken';

try {
  const payload = await verifyToken(token, process.env.JWT_SECRET);
  console.log('User ID:', payload.userId);
  console.log('Role:', payload.role);
} catch (error) {
  console.error('Token verification failed:', error.message);
}
```

### `hashPassword`

Hashes a password using bcrypt with salt rounds.

```typescript
function hashPassword(password: string, saltRounds?: number): Promise<string>
```

#### Parameters

- `password` (string): Password to hash
- `saltRounds` (number, optional): Number of salt rounds (default: 12)

#### Returns

- (Promise<string>): Hashed password

#### Example Usage

```typescript
import { hashPassword } from '@/auth/hashPassword';

// Hash password
const hashedPassword = await hashPassword('userPassword123');

// Custom salt rounds
const strongHash = await hashPassword('userPassword123', 14);
```

### `comparePassword`

Compares a plain text password with a hashed password.

```typescript
function comparePassword(password: string, hash: string): Promise<boolean>
```

#### Parameters

- `password` (string): Plain text password
- `hash` (string): Hashed password to compare against

#### Returns

- (Promise<boolean>): True if passwords match

#### Example Usage

```typescript
import { comparePassword } from '@/auth/comparePassword';

const isValid = await comparePassword('userPassword123', hashedPassword);

if (isValid) {
  console.log('Password is correct');
} else {
  console.log('Invalid password');
}
```

## Validation Functions

### `validateEmail`

Validates an email address format.

```typescript
function validateEmail(email: string): boolean
```

#### Parameters

- `email` (string): Email address to validate

#### Returns

- (boolean): True if email is valid

#### Example Usage

```typescript
import { validateEmail } from '@/utils/validation';

validateEmail('user@example.com'); // true
validateEmail('invalid-email'); // false
validateEmail('user@'); // false
```

### `validatePassword`

Validates password strength based on configurable criteria.

```typescript
function validatePassword(
  password: string,
  options?: {
    minLength?: number;
    requireUppercase?: boolean;
    requireLowercase?: boolean;
    requireNumbers?: boolean;
    requireSpecialChars?: boolean;
    forbiddenChars?: string[];
  }
): {
  isValid: boolean;
  errors: string[];
}
```

#### Parameters

- `password` (string): Password to validate
- `options` (object, optional): Validation criteria

#### Returns

- (object): Validation result with errors

#### Example Usage

```typescript
import { validatePassword } from '@/utils/validation';

// Basic validation
const result = validatePassword('Password123!');
console.log(result.isValid); // true
console.log(result.errors); // []

// Custom criteria
const strictResult = validatePassword('password', {
  minLength: 12,
  requireUppercase: true,
  requireNumbers: true,
  requireSpecialChars: true
});
console.log(strictResult.errors);
// ['Password must be at least 12 characters', 'Password must contain uppercase letters', ...]
```

### `validatePhoneNumber`

Validates phone number format with country code support.

```typescript
function validatePhoneNumber(
  phoneNumber: string,
  countryCode?: string
): boolean
```

#### Parameters

- `phoneNumber` (string): Phone number to validate
- `countryCode` (string, optional): Country code (e.g., 'US', 'GB')

#### Returns

- (boolean): True if phone number is valid

#### Example Usage

```typescript
import { validatePhoneNumber } from '@/utils/validation';

validatePhoneNumber('+1-555-123-4567'); // true
validatePhoneNumber('555-123-4567', 'US'); // true
validatePhoneNumber('07911 123456', 'GB'); // true
```

## API Client Functions

### `createApiClient`

Creates a configured API client with interceptors and error handling.

```typescript
function createApiClient(
  baseURL: string,
  options?: {
    timeout?: number;
    headers?: Record<string, string>;
    interceptors?: {
      request?: (config: any) => any;
      response?: (response: any) => any;
      error?: (error: any) => any;
    };
  }
): ApiClient
```

#### Parameters

- `baseURL` (string): Base URL for API requests
- `options` (object, optional): Client configuration options

#### Returns

- (ApiClient): Configured API client instance

#### Example Usage

```typescript
import { createApiClient } from '@/api/createApiClient';

// Basic client
const apiClient = createApiClient('https://api.example.com');

// Client with auth interceptor
const authApiClient = createApiClient('https://api.example.com', {
  headers: {
    'Content-Type': 'application/json'
  },
  interceptors: {
    request: (config) => {
      const token = localStorage.getItem('authToken');
      if (token) {
        config.headers.Authorization = `Bearer ${token}`;
      }
      return config;
    },
    error: (error) => {
      if (error.response?.status === 401) {
        // Handle unauthorized
        redirectToLogin();
      }
      throw error;
    }
  }
});

// Usage
const users = await authApiClient.get('/users');
const newUser = await authApiClient.post('/users', userData);
```

### `makeRequest`

Makes HTTP requests with retry logic and error handling.

```typescript
function makeRequest<T = any>(
  url: string,
  options?: {
    method?: 'GET' | 'POST' | 'PUT' | 'DELETE' | 'PATCH';
    data?: any;
    headers?: Record<string, string>;
    timeout?: number;
    retries?: number;
    retryDelay?: number;
  }
): Promise<T>
```

#### Parameters

- `url` (string): Request URL
- `options` (object, optional): Request options

#### Returns

- (Promise<T>): Response data

#### Example Usage

```typescript
import { makeRequest } from '@/api/makeRequest';

// GET request
const users = await makeRequest<User[]>('/api/users');

// POST request with data
const newUser = await makeRequest<User>('/api/users', {
  method: 'POST',
  data: { name: 'John Doe', email: 'john@example.com' },
  headers: { 'Content-Type': 'application/json' }
});

// Request with retry logic
const dataWithRetry = await makeRequest('/api/unstable-endpoint', {
  retries: 3,
  retryDelay: 1000
});
```

## State Management Functions

### `createStore`

Creates a simple state store with subscription capabilities.

```typescript
function createStore<T>(
  initialState: T,
  options?: {
    middleware?: Middleware<T>[];
  }
): Store<T>
```

#### Parameters

- `initialState` (T): Initial state value
- `options` (object, optional): Store configuration

#### Returns

- (Store<T>): State store instance

#### Example Usage

```typescript
import { createStore } from '@/state/createStore';

interface AppState {
  user: User | null;
  theme: 'light' | 'dark';
  loading: boolean;
}

const initialState: AppState = {
  user: null,
  theme: 'light',
  loading: false
};

// Create store
const store = createStore(initialState);

// Subscribe to changes
const unsubscribe = store.subscribe((state) => {
  console.log('State changed:', state);
});

// Update state
store.setState((prevState) => ({
  ...prevState,
  user: { id: 1, name: 'John Doe' }
}));

// Get current state
const currentState = store.getState();
```

### `useLocalStorage`

Custom hook for managing localStorage with JSON serialization.

```typescript
function useLocalStorage<T>(
  key: string,
  defaultValue: T
): [T, (value: T | ((prev: T) => T)) => void, () => void]
```

#### Parameters

- `key` (string): localStorage key
- `defaultValue` (T): Default value if key doesn't exist

#### Returns

- (tuple): [value, setValue, removeValue]

#### Example Usage

```typescript
import { useLocalStorage } from '@/hooks/useLocalStorage';

function UserPreferences() {
  const [theme, setTheme, removeTheme] = useLocalStorage('theme', 'light');
  const [settings, setSettings] = useLocalStorage('userSettings', {
    notifications: true,
    autoSave: false
  });

  const toggleTheme = () => {
    setTheme(theme === 'light' ? 'dark' : 'light');
  };

  const updateSettings = (newSettings: Partial<typeof settings>) => {
    setSettings(prev => ({ ...prev, ...newSettings }));
  };

  return (
    <div>
      <button onClick={toggleTheme}>
        Current theme: {theme}
      </button>
      <button onClick={() => updateSettings({ notifications: !settings.notifications })}>
        Notifications: {settings.notifications ? 'On' : 'Off'}
      </button>
    </div>
  );
}
```

## Date and Time Functions

### `formatDate`

Formats dates with locale and timezone support.

```typescript
function formatDate(
  date: Date | string | number,
  format?: string,
  locale?: string,
  timezone?: string
): string
```

#### Parameters

- `date` (Date | string | number): Date to format
- `format` (string, optional): Format pattern (default: 'YYYY-MM-DD')
- `locale` (string, optional): Locale string (default: 'en-US')
- `timezone` (string, optional): Timezone identifier

#### Returns

- (string): Formatted date string

#### Example Usage

```typescript
import { formatDate } from '@/utils/formatDate';

const date = new Date('2023-12-25T10:30:00Z');

// Basic formatting
formatDate(date); // "2023-12-25"
formatDate(date, 'MM/DD/YYYY'); // "12/25/2023"
formatDate(date, 'MMMM Do, YYYY'); // "December 25th, 2023"

// Locale formatting
formatDate(date, 'MMMM Do, YYYY', 'es-ES'); // "diciembre 25º, 2023"
formatDate(date, 'DD/MM/YYYY', 'en-GB'); // "25/12/2023"

// Timezone formatting
formatDate(date, 'YYYY-MM-DD HH:mm', 'en-US', 'America/New_York');
```

### `addDays`

Adds or subtracts days from a date.

```typescript
function addDays(date: Date, days: number): Date
```

#### Parameters

- `date` (Date): Base date
- `days` (number): Number of days to add (negative to subtract)

#### Returns

- (Date): New date instance

#### Example Usage

```typescript
import { addDays } from '@/utils/dateUtils';

const today = new Date();
const tomorrow = addDays(today, 1);
const lastWeek = addDays(today, -7);
const nextMonth = addDays(today, 30);
```

### `isDateInRange`

Checks if a date falls within a specified range.

```typescript
function isDateInRange(
  date: Date,
  startDate: Date,
  endDate: Date,
  inclusive?: boolean
): boolean
```

#### Parameters

- `date` (Date): Date to check
- `startDate` (Date): Range start date
- `endDate` (Date): Range end date
- `inclusive` (boolean, optional): Include range boundaries (default: true)

#### Returns

- (boolean): True if date is in range

#### Example Usage

```typescript
import { isDateInRange } from '@/utils/dateUtils';

const checkDate = new Date('2023-06-15');
const startDate = new Date('2023-06-01');
const endDate = new Date('2023-06-30');

isDateInRange(checkDate, startDate, endDate); // true

// Check if date is in current month
const currentMonth = new Date();
const monthStart = new Date(currentMonth.getFullYear(), currentMonth.getMonth(), 1);
const monthEnd = new Date(currentMonth.getFullYear(), currentMonth.getMonth() + 1, 0);

isDateInRange(new Date(), monthStart, monthEnd); // true
```

## String Manipulation Functions

### `slugify`

Converts a string to a URL-friendly slug.

```typescript
function slugify(
  text: string,
  options?: {
    separator?: string;
    lowercase?: boolean;
    strict?: boolean;
  }
): string
```

#### Parameters

- `text` (string): Text to slugify
- `options` (object, optional): Slugify options
  - `separator` (string): Character to use as separator (default: '-')
  - `lowercase` (boolean): Convert to lowercase (default: true)
  - `strict` (boolean): Remove non-alphanumeric characters (default: false)

#### Returns

- (string): Slugified string

#### Example Usage

```typescript
import { slugify } from '@/utils/stringUtils';

slugify('Hello World!'); // "hello-world"
slugify('Product Title & Description', { separator: '_' }); // "product_title_description"
slugify('Special Characters: @#$%', { strict: true }); // "special-characters"
```

### `truncate`

Truncates text to a specified length with ellipsis.

```typescript
function truncate(
  text: string,
  length: number,
  options?: {
    ellipsis?: string;
    wordBoundary?: boolean;
  }
): string
```

#### Parameters

- `text` (string): Text to truncate
- `length` (number): Maximum length
- `options` (object, optional): Truncation options

#### Returns

- (string): Truncated string

#### Example Usage

```typescript
import { truncate } from '@/utils/stringUtils';

const longText = "This is a very long piece of text that needs to be truncated";

truncate(longText, 20); // "This is a very long..."
truncate(longText, 20, { ellipsis: ' [more]' }); // "This is a very long [more]"
truncate(longText, 20, { wordBoundary: true }); // "This is a very..."
```

### `capitalize`

Capitalizes text with various options.

```typescript
function capitalize(
  text: string,
  mode?: 'first' | 'words' | 'sentences'
): string
```

#### Parameters

- `text` (string): Text to capitalize
- `mode` (string, optional): Capitalization mode (default: 'first')
  - `'first'`: Capitalize first letter only
  - `'words'`: Capitalize first letter of each word
  - `'sentences'`: Capitalize first letter of each sentence

#### Returns

- (string): Capitalized string

#### Example Usage

```typescript
import { capitalize } from '@/utils/stringUtils';

capitalize('hello world'); // "Hello world"
capitalize('hello world', 'words'); // "Hello World"
capitalize('hello. how are you?', 'sentences'); // "Hello. How are you?"
```

## Array and Object Functions

### `groupBy`

Groups array elements by a specified key or function.

```typescript
function groupBy<T, K extends keyof any>(
  array: T[],
  key: K | ((item: T) => K)
): Record<K, T[]>
```

#### Parameters

- `array` (T[]): Array to group
- `key` (K | Function): Grouping key or function

#### Returns

- (Record<K, T[]>): Grouped object

#### Example Usage

```typescript
import { groupBy } from '@/utils/arrayUtils';

const users = [
  { name: 'John', department: 'Engineering' },
  { name: 'Jane', department: 'Marketing' },
  { name: 'Bob', department: 'Engineering' }
];

// Group by property
const byDepartment = groupBy(users, 'department');
// Result: {
//   Engineering: [{ name: 'John', department: 'Engineering' }, { name: 'Bob', department: 'Engineering' }],
//   Marketing: [{ name: 'Jane', department: 'Marketing' }]
// }

// Group by function
const byNameLength = groupBy(users, user => user.name.length);
// Result: {
//   3: [{ name: 'Bob', department: 'Engineering' }],
//   4: [{ name: 'John', department: 'Engineering' }, { name: 'Jane', department: 'Marketing' }]
// }
```

### `sortBy`

Sorts array by one or more properties.

```typescript
function sortBy<T>(
  array: T[],
  key: keyof T | ((item: T) => any),
  direction?: 'asc' | 'desc'
): T[]
```

#### Parameters

- `array` (T[]): Array to sort
- `key` (keyof T | Function): Sort key or function
- `direction` (string, optional): Sort direction (default: 'asc')

#### Returns

- (T[]): Sorted array (new instance)

#### Example Usage

```typescript
import { sortBy } from '@/utils/arrayUtils';

const products = [
  { name: 'Laptop', price: 999, category: 'Electronics' },
  { name: 'Book', price: 20, category: 'Books' },
  { name: 'Phone', price: 699, category: 'Electronics' }
];

// Sort by price (ascending)
const byPrice = sortBy(products, 'price');

// Sort by price (descending)
const byPriceDesc = sortBy(products, 'price', 'desc');

// Sort by custom function
const byNameLength = sortBy(products, product => product.name.length);
```

### `deepClone`

Creates a deep copy of an object or array.

```typescript
function deepClone<T>(obj: T): T
```

#### Parameters

- `obj` (T): Object to clone

#### Returns

- (T): Deep cloned object

#### Example Usage

```typescript
import { deepClone } from '@/utils/objectUtils';

const original = {
  user: {
    name: 'John',
    preferences: {
      theme: 'dark',
      notifications: true
    }
  },
  items: [1, 2, 3]
};

const cloned = deepClone(original);
cloned.user.name = 'Jane'; // Original remains unchanged
cloned.items.push(4); // Original array remains unchanged
```

### `mergeDeep`

Deeply merges multiple objects.

```typescript
function mergeDeep<T extends Record<string, any>>(...objects: Partial<T>[]): T
```

#### Parameters

- `objects` (...Partial<T>[]): Objects to merge

#### Returns

- (T): Merged object

#### Example Usage

```typescript
import { mergeDeep } from '@/utils/objectUtils';

const defaultConfig = {
  api: {
    timeout: 5000,
    retries: 3
  },
  ui: {
    theme: 'light',
    animations: true
  }
};

const userConfig = {
  api: {
    timeout: 10000
  },
  ui: {
    theme: 'dark'
  }
};

const finalConfig = mergeDeep(defaultConfig, userConfig);
// Result: {
//   api: { timeout: 10000, retries: 3 },
//   ui: { theme: 'dark', animations: true }
// }
```

## File Handling Functions

### `downloadFile`

Downloads a file from a URL or blob with progress tracking.

```typescript
function downloadFile(
  source: string | Blob,
  filename: string,
  options?: {
    onProgress?: (progress: number) => void;
    signal?: AbortSignal;
  }
): Promise<void>
```

#### Parameters

- `source` (string | Blob): File URL or blob
- `filename` (string): Download filename
- `options` (object, optional): Download options

#### Returns

- (Promise<void>): Download completion promise

#### Example Usage

```typescript
import { downloadFile } from '@/utils/fileUtils';

// Download from URL
await downloadFile('/api/files/report.pdf', 'monthly-report.pdf', {
  onProgress: (progress) => {
    console.log(`Download progress: ${progress}%`);
  }
});

// Download blob
const csvData = new Blob([csvContent], { type: 'text/csv' });
await downloadFile(csvData, 'export.csv');

// With abort controller
const controller = new AbortController();
downloadFile('/large-file.zip', 'file.zip', {
  signal: controller.signal
});

// Cancel download after 10 seconds
setTimeout(() => controller.abort(), 10000);
```

### `readFileAsText`

Reads a file as text with encoding support.

```typescript
function readFileAsText(
  file: File,
  encoding?: string
): Promise<string>
```

#### Parameters

- `file` (File): File to read
- `encoding` (string, optional): Text encoding (default: 'UTF-8')

#### Returns

- (Promise<string>): File content as string

#### Example Usage

```typescript
import { readFileAsText } from '@/utils/fileUtils';

// File input handler
const handleFileSelect = async (event: React.ChangeEvent<HTMLInputElement>) => {
  const file = event.target.files?.[0];
  if (file) {
    try {
      const content = await readFileAsText(file);
      console.log('File content:', content);
    } catch (error) {
      console.error('Failed to read file:', error);
    }
  }
};

// Read with specific encoding
const csvContent = await readFileAsText(csvFile, 'UTF-8');
```

### `validateFileType`

Validates file type against allowed types.

```typescript
function validateFileType(
  file: File,
  allowedTypes: string[],
  options?: {
    checkMimeType?: boolean;
    checkExtension?: boolean;
  }
): boolean
```

#### Parameters

- `file` (File): File to validate
- `allowedTypes` (string[]): Array of allowed types
- `options` (object, optional): Validation options

#### Returns

- (boolean): True if file type is valid

#### Example Usage

```typescript
import { validateFileType } from '@/utils/fileUtils';

const imageTypes = ['image/jpeg', 'image/png', 'image/gif'];
const documentTypes = ['.pdf', '.doc', '.docx'];

// Validate image file
const isValidImage = validateFileType(imageFile, imageTypes);

// Validate document by extension
const isValidDocument = validateFileType(docFile, documentTypes, {
  checkExtension: true,
  checkMimeType: false
});

// File upload handler
const handleFileUpload = (file: File) => {
  if (!validateFileType(file, ['image/jpeg', 'image/png'])) {
    alert('Please select a JPEG or PNG image');
    return;
  }
  
  // Process valid file
  uploadFile(file);
};
```