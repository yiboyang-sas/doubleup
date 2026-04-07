# DoubleUp - API & Integration Guide

## Database Schema Overview

### Tables

#### profiles
```sql
id: uuid (primary key)
user_id: uuid (foreign key to auth.users)
display_name: text
email: text
university: text
interested_in: enum('men', 'women', 'everyone')
bio: text
avatar_url: text
instagram: text
tiktok: text
interests: text[]
created_at: timestamp
updated_at: timestamp
```

#### duos
```sql
id: uuid (primary key)
user1_id: uuid (foreign key to profiles)
user2_id: uuid (foreign key to profiles)
prompt1: text
answer1: text
prompt2: text
answer2: text
interests: text[]
photo1_url: text
photo2_url: text
school: text
active: boolean
created_at: timestamp
updated_at: timestamp
```

#### matches
```sql
id: uuid (primary key)
user_id: uuid (foreign key to profiles)
liked_duo_id: uuid (foreign key to duos)
status: enum('pending', 'matched', 'rejected')
created_at: timestamp
```

#### chat_rooms
```sql
id: uuid (primary key)
name: text
type: enum('direct', 'group')
created_at: timestamp
```

#### chat_room_members
```sql
id: uuid (primary key)
room_id: uuid (foreign key)
user_id: uuid (foreign key)
joined_at: timestamp
```

#### messages
```sql
id: uuid (primary key)
room_id: uuid (foreign key)
sender_id: uuid (foreign key)
content: text
created_at: timestamp
updated_at: timestamp
```

#### duo_invites
```sql
id: uuid (primary key)
from_user_id: uuid (foreign key)
to_user_id: uuid (foreign key)
status: enum('pending', 'accepted', 'rejected')
created_at: timestamp
```

#### blocked_users
```sql
id: uuid (primary key)
user_id: uuid (foreign key)
blocked_user_id: uuid (foreign key)
created_at: timestamp
```

## API Endpoints

### Authentication
- `POST /auth/signup` - Sign up with email
- `POST /auth/login` - Login with email/password
- `POST /auth/logout` - Logout
- `POST /auth/refresh` - Refresh token
- `POST /auth/reset-password` - Reset password

### Profiles
- `GET /profiles/:id` - Get profile
- `PUT /profiles/:id` - Update profile
- `POST /profiles/:id/avatar` - Upload avatar
- `POST /profiles/:id/photos` - Upload photos

### Duos
- `GET /duos` - Get all active duos
- `GET /duos/:id` - Get duo by ID
- `POST /duos` - Create new duo
- `PUT /duos/:id` - Update duo
- `POST /duos/:id/photos` - Upload duo photos

### Discovery
- `GET /discovery/duos` - Get duos for swiping
- `POST /discovery/swipe` - Record swipe action
- `GET /discovery/matches` - Get matches

### Messaging
- `GET /chat-rooms` - Get user's chat rooms
- `GET /chat-rooms/:id/messages` - Get messages
- `POST /chat-rooms/:id/messages` - Send message
- `POST /chat-rooms` - Create new room
- `GET /chat-rooms/:id/members` - Get room members

### Invitations
- `GET /invites` - Get pending invites
- `POST /invites` - Send invite
- `PUT /invites/:id/accept` - Accept invite
- `PUT /invites/:id/reject` - Reject invite

### Users
- `GET /users/search` - Search users
- `POST /users/block` - Block user
- `GET /users/blocked` - Get blocked users
- `DELETE /users/block/:id` - Unblock user

## Real-time Subscriptions (Supabase)

### Messages
```typescript
supabase
  .channel(`room-${roomId}`)
  .on('postgres_changes', 
    { event: 'INSERT', schema: 'public', table: 'messages' },
    (payload) => { /* handle new message */ }
  )
  .subscribe();
```

### Notifications
```typescript
supabase
  .channel('notifications')
  .on('postgres_changes',
    { event: 'INSERT', schema: 'public', table: 'matches' },
    (payload) => { /* handle new match */ }
  )
  .subscribe();
```

### Presence (Online Status)
```typescript
const channel = supabase.channel('presence');

channel
  .on('presence', { event: 'sync' }, () => {
    const state = channel.presenceState();
  })
  .subscribe(async (status) => {
    if (status === 'SUBSCRIBED') {
      await channel.track({ user_id, online_at: new Date() });
    }
  });
```

## Image Storage

### Upload Endpoints
- Avatars: `/avatars/{user_id}/{filename}`
- Duo Photos: `/duos/{duo_id}/{slot}`
- Chat Media: `/chat/{room_id}/{filename}`

### Storage Policies
- Public read access
- Authenticated user write access
- User can only write to own files

### Image Optimization
- Max size: 5MB
- Formats: JPG, PNG, WebP
- Auto-resize: 1024x1024 max
- Auto-compress: 80% quality

## Error Handling

### Error Codes

#### Authentication
- `AUTH_INVALID_EMAIL` - Invalid email format
- `AUTH_WEAK_PASSWORD` - Password too weak
- `AUTH_USER_NOT_FOUND` - User doesn't exist
- `AUTH_INVALID_CREDENTIALS` - Wrong password

#### Validation
- `VALIDATION_REQUIRED` - Required field missing
- `VALIDATION_INVALID_FORMAT` - Invalid format
- `VALIDATION_TOO_SHORT` - Too short
- `VALIDATION_TOO_LONG` - Too long

#### Files
- `FILE_UNSUPPORTED_TYPE` - Wrong file type
- `FILE_TOO_LARGE` - File > 5MB
- `FILE_UPLOAD_FAILED` - Upload error

#### Database
- `DATABASE_ERROR` - General error
- `DATABASE_CONFLICT` - Duplicate record

### Error Response Format
```json
{
  "success": false,
  "error": {
    "code": "AUTH_INVALID_EMAIL",
    "message": "Invalid email format",
    "details": {}
  }
}
```

## Rate Limiting

- Login attempts: 5 per minute
- API calls: 1000 per hour per user
- Upload: 10 per minute
- Message send: 100 per minute

## Authentication Flow

1. User enters email/password
2. POST `/auth/signup` or `/auth/login`
3. Supabase returns `access_token` and `refresh_token`
4. Store tokens in secure storage
5. Include `Authorization: Bearer {token}` in requests
6. Refresh token expires in 7 days
7. Automatically refresh using refresh token

## WebSocket Events

### Message Events
- `message:new` - New message arrived
- `message:updated` - Message edited
- `message:deleted` - Message deleted

### Match Events
- `match:new` - New match found
- `match:accepted` - Match accepted
- `match:rejected` - Match rejected

### User Events
- `user:online` - User came online
- `user:offline` - User went offline
- `user:blocked` - User blocked

### Notification Events
- `notification:invite` - Duo invite received
- `notification:match` - Match notification
- `notification:message` - New message

## Rate Limiting Strategy

```typescript
import { useDebounce } from '@/hooks/useUtils';

// Debounce search to prevent excessive API calls
const debouncedQuery = useDebounce(searchQuery, 500);

useEffect(() => {
  // Only call API after user stops typing
  searchUsers(debouncedQuery);
}, [debouncedQuery]);
```

## Caching Strategy

```typescript
import { useLocalStorage } from '@/hooks/useUtils';

// Cache user data locally
const [cachedProfile, setCachedProfile] = useLocalStorage('profile', null);

// Use cache initially, then fetch fresh data
useEffect(() => {
  if (!cachedProfile) {
    fetchProfile().then(setCachedProfile);
  }
}, []);
```

## Best Practices

### For API Calls
1. Always include error handling
2. Set loading states
3. Use debouncing for search
4. Cache when appropriate
5. Validate user input
6. Log errors (not in production)

### For Real-time
1. Subscribe in useEffect
2. Unsubscribe on cleanup
3. Handle connection loss
4. Retry with backoff
5. Verify user permissions

### For Security
1. Never expose secrets in code
2. Use HTTPS only
3. Validate all input
4. Sanitize output
5. Check user permissions
6. Use Row-Level Security

### For Performance
1. Paginate large datasets
2. Use indexes on common queries
3. Cache frequently accessed data
4. Use CDN for images
5. Compress API responses
6. Optimize database queries

## Testing API Integration

```typescript
// Mock API responses for testing
vi.mock('@/integrations/supabase/client', () => ({
  supabase: {
    auth: {
      signIn: vi.fn(() => Promise.resolve({ user: mockUser })),
    },
    from: vi.fn(() => ({
      select: vi.fn(() => ({
        eq: vi.fn(() => Promise.resolve({ data: [] })),
      })),
    }),
  },
}));
```

## Monitoring & Analytics

### Track Events
- User signup
- User login
- Swipe action
- Match found
- Message sent
- Profile updated

### Track Errors
- Authentication failures
- API errors
- Network errors
- Upload failures

### Track Performance
- Page load time
- API response time
- Image load time
- First interaction
