# DoubleUp - Quick Reference Checklist

## Before Going Live

### Infrastructure
- [ ] Supabase project configured
- [ ] Database migrations run
- [ ] Environment variables set up
- [ ] CDN configured for images
- [ ] Error logging configured (Sentry/LogRocket)
- [ ] Analytics installed (Mixpanel/Amplitude)

### Security
- [ ] CSP headers configured
- [ ] CORS properly set
- [ ] Rate limiting enabled
- [ ] Input validation on backend
- [ ] Authentication tests passed
- [ ] SSL certificate valid
- [ ] Secrets not exposed in code

### Performance
- [ ] Lighthouse score 90+
- [ ] Bundle size under 500KB
- [ ] First Contentful Paint < 2s
- [ ] Largest Contentful Paint < 2.5s
- [ ] Cumulative Layout Shift < 0.1
- [ ] Images optimized
- [ ] Caching headers set

### Content
- [ ] Copy proofread
- [ ] All links working
- [ ] 404 page working
- [ ] Terms of Service updated
- [ ] Privacy Policy updated
- [ ] Help/FAQ populated

### Testing
- [ ] Unit tests passing
- [ ] E2E tests passing
- [ ] Manual testing on iOS Safari
- [ ] Manual testing on Android Chrome
- [ ] Accessibility testing done
- [ ] Cross-browser testing done

### Monitoring
- [ ] Error tracking active
- [ ] Performance monitoring active
- [ ] User analytics tracking
- [ ] Crash reporting enabled
- [ ] Uptime monitoring configured

## Development Commands

```bash
# Setup
npm install
npm run dev

# Building
npm run build
npm run build:dev

# Testing
npm run test
npm run test:watch

# Linting
npm run lint

# Production preview
npm run preview
```

## Key Files

| File | Purpose |
|------|---------|
| `src/App.tsx` | Main app router |
| `src/contexts/AuthContext.tsx` | Authentication state |
| `src/lib/utils.ts` | Utility functions |
| `src/lib/errors.ts` | Error handling |
| `src/constants/app.ts` | App constants |
| `src/types/index.ts` | TypeScript definitions |
| `vite.config.ts` | Build config |
| `tailwind.config.ts` | Tailwind config |

## Component Library

### Buttons
```tsx
import { Button } from "@/components/ui/button";

<Button variant="primary" size="lg">Click</Button>
<Button variant="secondary" disabled>Disabled</Button>
<Button loading>Saving...</Button>
```

### Forms
```tsx
import { Input } from "@/components/ui/common";

<Input label="Email" error={error} placeholder="email@edu.com" />
```

### Cards
```tsx
import { Card } from "@/components/ui/common";

<Card hoverable>Content</Card>
```

## Common Patterns

### Error Handling
```tsx
import { handleError, showErrorToast } from "@/lib/errors";

try {
  await myAsyncFunction();
} catch (error) {
  const appError = handleError(error);
  showErrorToast(appError);
}
```

### Validation
```tsx
import { validateEmail, validatePassword } from "@/lib/errors";

const emailError = validateEmail(email);
const passwordError = validatePassword(password);
```

### Utilities
```tsx
import { formatDate, truncateText, validateEmail } from "@/lib/utils";

const date = formatDate(new Date());
const text = truncateText("Long text", 50);
```

### Custom Hooks
```tsx
import { useDebounce, useLocalStorage } from "@/hooks/useUtils";

const [value, setValue] = useLocalStorage("key", "default");
const debouncedValue = useDebounce(value, 500);
```

## Deployment Checklist

### Pre-Deployment
- [ ] All tests passing
- [ ] No console errors
- [ ] No TypeScript errors
- [ ] All environment variables set
- [ ] Database backups created
- [ ] Git tags created

### Deployment
- [ ] Run `npm run build`
- [ ] Verify build size
- [ ] Deploy to staging first
- [ ] Smoke test staging
- [ ] Deploy to production
- [ ] Verify production working
- [ ] Monitor error logs

### Post-Deployment
- [ ] Check Lighthouse scores
- [ ] Verify all features working
- [ ] Monitor error tracking
- [ ] Check user analytics
- [ ] Monitor performance metrics

## Troubleshooting

### High bundle size
```bash
npm run build -- --analyze
# Look for large dependencies
# Consider code splitting
```

### Slow performance
- Check Network tab in DevTools
- Profile with Lighthouse
- Check Core Web Vitals
- Optimize images
- Enable compression

### Build errors
```bash
# Clear cache
rm -rf node_modules package-lock.json
npm install

# Check for TypeScript errors
npx tsc --noEmit
```

## Resources

- [Supabase Docs](https://supabase.io/docs)
- [React Router Docs](https://reactrouter.com)
- [Tailwind CSS Docs](https://tailwindcss.com/docs)
- [Framer Motion Docs](https://www.framer.com/motion)
- [Vite Docs](https://vitejs.dev)

## Support

For issues or questions:
1. Check existing documentation
2. Review similar implementations in codebase
3. Check error logs and error boundaries
4. Reference type definitions in `src/types/index.ts`
5. Review utility functions in `src/lib/`

## Notes

- All components should include proper TypeScript types
- All interactive elements must be keyboard accessible
- All images must have alt text
- All buttons must have aria-labels or text content
- All forms must have proper validation
- All async operations should have loading states
- All errors should be caught and handled gracefully
