# DoubleUp - Advanced Utilities & Hooks Documentation

## Overview

This document covers all the advanced utilities, hooks, and modules added to DoubleUp for improved performance, maintainability, and developer experience.

---

## 📚 Table of Contents

1. [Advanced Hooks](#advanced-hooks)
2. [Performance Utilities](#performance-utilities)
3. [API & Networking](#api--networking)
4. [Analytics](#analytics)
5. [Validation](#validation)
6. [Usage Examples](#usage-examples)

---

## Advanced Hooks

### useOptimistic

Enables optimistic UI updates - updates the UI immediately while an async operation is in progress.

```tsx
import { useOptimistic } from '@/hooks/useOptimistic';

function MyComponent() {
  const { data, isPending, error, updateOptimistic } = useOptimistic(
    initialValue,
    async (newValue) => {
      const response = await api.updateValue(newValue);
      return response;
    }
  );

  const handleUpdate = async () => {
    try {
      await updateOptimistic(newValue);
    } catch (error) {
      console.error('Update failed:', error);
    }
  };

  return (
    <div>
      <p>Current: {data}</p>
      {isPending && <p>Updating...</p>}
      {error && <p>Error: {error.message}</p>}
      <button onClick={handleUpdate}>Update</button>
    </div>
  );
}
```

### useLoadMore

Handles pagination and infinite scrolling.

```tsx
import { useLoadMore } from '@/hooks/useOptimistic';

function ListComponent() {
  const { items, isLoading, hasMore, loadMore } = useLoadMore(
    [],
    async (offset, limit) => {
      const response = await api.getItems(offset, limit);
      return response.items;
    },
    10 // page size
  );

  return (
    <div>
      <ul>
        {items.map(item => <li key={item.id}>{item.name}</li>)}
      </ul>
      {hasMore && (
        <button onClick={loadMore} disabled={isLoading}>
          {isLoading ? 'Loading...' : 'Load More'}
        </button>
      )}
    </div>
  );
}
```

### useSearch

Debounced search hook for real-time search.

```tsx
import { useSearch } from '@/hooks/useOptimistic';

function SearchComponent() {
  const { query, results, isLoading, handleSearch } = useSearch(
    async (query) => {
      const response = await api.search(query);
      return response.results;
    },
    300 // debounce ms
  );

  return (
    <div>
      <input 
        value={query} 
        onChange={(e) => handleSearch(e.target.value)}
        placeholder="Search..."
      />
      {isLoading && <p>Searching...</p>}
      <ul>
        {results.map(result => (
          <li key={result.id}>{result.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

### useForm & useMultiStepForm

Advanced form handling with validation.

```tsx
import { useForm, useMultiStepForm } from '@/hooks/useForm';
import { Rules, validator } from '@/lib/validation';

function LoginForm() {
  const { values, errors, handleSubmit, getFieldProps } = useForm(
    { email: '', password: '' },
    async (values) => {
      await api.login(values);
    },
    (values) => {
      const schema = {
        email: Rules.email(),
        password: [
          Rules.required(),
          Rules.minLength(8)
        ]
      };
      return validator.validate(values, schema);
    }
  );

  return (
    <form onSubmit={handleSubmit}>
      <input {...getFieldProps('email')} placeholder="Email" />
      {errors.email && <span>{errors.email}</span>}
      
      <input {...getFieldProps('password')} type="password" placeholder="Password" />
      {errors.password && <span>{errors.password}</span>}
      
      <button type="submit">Login</button>
    </form>
  );
}

// Multi-step form
function SignUpForm() {
  const { currentStep, currentFields, isLastStep, progress, nextStep, prevStep } = useMultiStepForm(
    [
      ['email', 'password'], // Step 1
      ['name', 'university'], // Step 2
      ['interests', 'bio'] // Step 3
    ],
    { email: '', password: '', name: '', university: '', interests: [], bio: '' }
  );

  return (
    <div>
      <div>Progress: {progress}%</div>
      {/* Render fields based on currentFields */}
      <button onClick={prevStep} disabled={currentStep === 0}>Previous</button>
      <button onClick={nextStep} disabled={isLastStep}>Next</button>
    </div>
  );
}
```

---

## Performance Utilities

### Memoization

```tsx
import { memoize } from '@/lib/performance';

const expensiveCalculation = memoize((a: number, b: number) => {
  // This will only run once for the same inputs
  return a + b;
});

console.log(expensiveCalculation(1, 2)); // Runs
console.log(expensiveCalculation(1, 2)); // Uses cache
```

### Debounce & Throttle

```tsx
import { debounceWithImmediate, throttle } from '@/lib/performance';

const debouncedSearch = debounceWithImmediate((query: string) => {
  api.search(query);
}, 500);

const throttledScroll = throttle(() => {
  console.log('Scroll event');
}, 1000);

window.addEventListener('scroll', throttledScroll);
```

### Retry with Backoff

```tsx
import { retryWithBackoff } from '@/lib/performance';

const data = await retryWithBackoff(
  () => api.fetchData(),
  3, // max retries
  1000 // base delay ms
);
```

### Performance Measurement

```tsx
import { measurePerformance } from '@/lib/performance';

const { result, duration } = measurePerformance('myOperation', () => {
  return expensiveFunction();
});

console.log(`Operation took ${duration}ms`);
```

---

## API & Networking

### ApiClient

Advanced HTTP client with interceptors and retries.

```tsx
import { apiClient } from '@/lib/api';

// Configure client
apiClient.use('request', (config) => {
  // Add auth token
  config.headers = {
    ...config.headers,
    'Authorization': `Bearer ${token}`
  };
  return config;
});

// Make requests
const user = await apiClient.get<User>('/users/me');
const created = await apiClient.post<Post>('/posts', { title: 'Hello' });
```

### Query Cache

```tsx
import { queryCache } from '@/lib/api';

// Cache for 5 minutes
queryCache.set('userList', users, 5 * 60 * 1000);

// Retrieve
const cached = queryCache.get('userList');

// Check existence
if (queryCache.has('userList')) {
  console.log('Cache is fresh');
}
```

### Network Status

```tsx
import { networkStatus } from '@/lib/api';

// Check current status
if (networkStatus.isOnline()) {
  console.log('Online');
}

// Listen for changes
const unsubscribe = networkStatus.onChange((isOnline) => {
  console.log(isOnline ? 'Connected' : 'Disconnected');
});
```

---

## Analytics

### Tracking Events

```tsx
import { analytics } from '@/lib/analytics';

// Track custom event
analytics.track('user_signup', 'onboarding', 'email');

// Track conversion
analytics.trackConversion('premium_upgrade', 9.99);

// Track performance
analytics.trackPerformance('page_load', 1234);

// Track errors
analytics.trackError(error, { context: 'payment' });
```

### Using Hooks

```tsx
import { useAnalyticsPageView, useAnalyticsEvent } from '@/lib/analytics';

function MyPage() {
  useAnalyticsPageView('my-page');
  
  const trackClick = useAnalyticsEvent('button_click', 'interaction');
  
  return (
    <button onClick={() => trackClick('submit')}>
      Submit
    </button>
  );
}
```

---

## Validation

### Schema Validation

```tsx
import { Rules, validator, validateData } from '@/lib/validation';

const schema = {
  email: Rules.email(),
  password: [
    Rules.required(),
    Rules.minLength(8),
    Rules.pattern(/[A-Z]/, 'Must contain uppercase letter')
  ],
  age: [
    Rules.required(),
    Rules.minValue(18),
    Rules.maxValue(120)
  ],
  website: Rules.url(),
  phone: Rules.phone(),
};

const result = validateData(formData, schema);
if (result.success) {
  console.log('Valid data:', result.data);
} else {
  console.log('Errors:', result.errors);
}
```

### Available Rules

- `required()` - Field is required
- `email()` - Valid email format
- `minLength(length)` - Minimum string length
- `maxLength(length)` - Maximum string length
- `pattern(regex)` - Matches regex pattern
- `match(other)` - Matches other value
- `custom(fn)` - Custom validation function
- `minValue(num)` - Minimum numeric value
- `maxValue(num)` - Maximum numeric value
- `url()` - Valid URL
- `phone()` - Valid phone number
- `creditCard()` - Valid credit card (Luhn check)

### Input Sanitization

```tsx
import { sanitizeInput } from '@/lib/validation';

const cleanInput = sanitizeInput(userInput);
// Prevents XSS attacks
```

---

## Usage Examples

### Complete Login Page with New Utilities

```tsx
import { useForm } from '@/hooks/useForm';
import { Rules, validator } from '@/lib/validation';
import { analytics } from '@/lib/analytics';

export function LoginPage() {
  const { values, errors, isSubmitting, handleSubmit, getFieldProps } = useForm(
    { email: '', password: '' },
    async (values) => {
      try {
        await apiClient.post('/auth/login', values);
        analytics.track('user_login', 'authentication', 'email');
      } catch (error) {
        analytics.trackError(error as Error, { context: 'login' });
        throw error;
      }
    },
    (values) => {
      return validator.validate(values, {
        email: Rules.email(),
        password: [
          Rules.required(),
          Rules.minLength(8)
        ]
      });
    }
  );

  return (
    <form onSubmit={handleSubmit}>
      <input {...getFieldProps('email')} type="email" />
      {errors.email && <p className="error">{errors.email}</p>}
      
      <input {...getFieldProps('password')} type="password" />
      {errors.password && <p className="error">{errors.password}</p>}
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
}
```

### Infinite Scroll with Search

```tsx
import { useSearch } from '@/hooks/useOptimistic';
import { useLoadMore } from '@/hooks/useOptimistic';

export function UsersList() {
  const search = useSearch(api.searchUsers);
  const list = useLoadMore([], api.loadUsers);

  return (
    <div>
      <input 
        value={search.query}
        onChange={(e) => search.handleSearch(e.target.value)}
        placeholder="Search users..."
      />
      
      <ul>
        {(search.query ? search.results : list.items).map(user => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
      
      {!search.query && list.hasMore && (
        <button onClick={list.loadMore} disabled={list.isLoading}>
          Load More
        </button>
      )}
    </div>
  );
}
```

---

## Performance Tips

1. **Use memoize** for expensive calculations
2. **Use debounce** for search and resize events
3. **Use throttle** for scroll events
4. **Cache API responses** with QueryCache
5. **Use useOptimistic** for better UX on mutations
6. **Track performance** with analytics
7. **Validate early** to catch errors quickly

---

## Migration Guide

If you have existing code:

### Before
```tsx
// Old validation approach
if (!email || !email.includes('@')) {
  setError('Invalid email');
}
```

### After
```tsx
// New validation approach
import { Rules, validator } from '@/lib/validation';

const error = validator.validateField(email, Rules.email());
```

---

## Summary

These advanced utilities provide:

✅ **Better Performance** - Memoization, debouncing, lazy loading
✅ **Improved DX** - Advanced form handling, validation
✅ **Network Resilience** - Retry logic, offline support
✅ **Analytics** - Track user behavior and performance
✅ **Code Reusability** - Centralized utilities and hooks

All utilities are fully typed with TypeScript for maximum safety and IDE support.
