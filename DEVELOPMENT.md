# DoubleUp - Development Guide

## Project Structure

```
src/
├── components/
│   ├── ui/              # Shadcn UI components
│   ├── common.tsx       # Reusable custom components
│   ├── AppLayout.tsx
│   ├── BottomNav.tsx
│   ├── ErrorBoundary.tsx
│   └── ...
├── contexts/            # React contexts
│   └── AuthContext.tsx
├── hooks/               # Custom React hooks
│   ├── useUtils.ts      # Utility hooks (debounce, localStorage, etc)
│   └── ...
├── pages/               # Page components
├── lib/
│   └── utils.ts         # Utility functions
├── constants/
│   └── app.ts           # App-wide constants
├── integrations/
│   └── supabase/        # Supabase integration
└── data/                # Mock data
```

## Key Components

### Button Component
Custom button component with multiple variants and sizes:
```tsx
<Button variant="primary" size="lg" loading={isLoading}>
  Click me
</Button>
```

### Input Component
Input with built-in label and error handling:
```tsx
<Input label="Email" error={emailError} placeholder="your@email.com" />
```

### Utility Functions

#### Date Formatting
- `formatDate(date)` - Returns formatted date string
- `formatTime(date)` - Returns formatted time string
- `getRelativeTime(date)` - Returns relative time (e.g., "2h ago")

#### Validation
- `validateEmail(email)` - Validates email format
- `validatePassword(password)` - Checks minimum length
- `truncateText(text, maxLength)` - Truncates with ellipsis

## Accessibility Features

All interactive elements include:
- `aria-label` - Screen reader labels
- `focus-visible` - Keyboard focus indicators
- `aria-pressed` / `aria-selected` - State indicators
- Proper semantic HTML
- Keyboard navigation support

## Performance Optimizations

1. **Code Splitting**: Separate chunks for vendor, UI, and Supabase
2. **Tree Shaking**: Unused code is automatically removed
3. **Lazy Loading**: Components load on demand
4. **Image Optimization**: Images served from CDN

## Environment Variables

Create a `.env.local` file:
```
VITE_SUPABASE_URL=your_url
VITE_SUPABASE_ANON_KEY=your_key
```

## Development

```bash
npm install
npm run dev      # Start development server
npm run build    # Build for production
npm run lint     # Run ESLint
npm test         # Run tests
```

## Browser Support

- Chrome/Edge 90+
- Firefox 88+
- Safari 14+
- Mobile browsers (iOS Safari, Chrome Android)

## Testing

Unit tests use Vitest. Add tests in `src/test/`:
```typescript
import { describe, it, expect } from "vitest";
import { validateEmail } from "@/lib/utils";

describe("validateEmail", () => {
  it("validates correct email format", () => {
    expect(validateEmail("test@example.com")).toBe(true);
    expect(validateEmail("invalid-email")).toBe(false);
  });
});
```

## Deployment

### Production Checklist
- [ ] All environment variables set
- [ ] Error logging configured
- [ ] Analytics enabled
- [ ] Security headers configured
- [ ] CSP policy set
- [ ] Rate limiting enabled
- [ ] CORS configured properly

### Build Command
```bash
npm run build
```

### Deploy to Vercel
```bash
vercel deploy
```

## Security Best Practices

1. Never commit `.env.local`
2. Use environment variables for sensitive data
3. Always validate user input
4. Sanitize output
5. Use HTTPS only
6. Implement CORS properly
7. Use CSP headers
8. Regular dependency updates

## Common Issues & Fixes

### Build size too large
- Use dynamic imports
- Remove unused dependencies
- Enable compression in build

### Slow initial load
- Implement code splitting
- Lazy load routes
- Optimize images

### Memory leaks
- Clean up subscriptions in useEffect
- Remove event listeners
- Clear timers

## Git Workflow

```bash
git checkout -b feature/my-feature
git commit -m "feat: add new feature"
git push origin feature/my-feature
# Create PR and merge
```

## Contributing

1. Follow code style
2. Add tests for new features
3. Update documentation
4. Commit with clear messages
5. Keep PRs focused
