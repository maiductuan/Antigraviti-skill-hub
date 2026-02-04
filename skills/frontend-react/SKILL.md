---
name: frontend-react
description: React and Next.js development patterns including component architecture, hooks, state management with Redux/Zustand, and performance optimization
tags: [react, nextjs, frontend, hooks, redux, zustand]
author: Antigravity Team
version: 1.0.0
---

# React & Next.js Development Skill

Expert guidance for building modern React applications with best practices.

## Component Architecture

### Functional Components with Hooks
```jsx
import { useState, useEffect, useCallback, useMemo } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchUser(userId).then(data => {
      setUser(data);
      setLoading(false);
    });
  }, [userId]);

  const fullName = useMemo(() => 
    user ? `${user.firstName} ${user.lastName}` : '',
    [user]
  );

  const handleUpdate = useCallback(async (updates) => {
    await updateUser(userId, updates);
  }, [userId]);

  if (loading) return <Spinner />;
  return <ProfileCard user={user} onUpdate={handleUpdate} />;
}
```

### Custom Hooks Pattern
```jsx
function useAsync(asyncFn, deps = []) {
  const [state, setState] = useState({
    data: null,
    loading: true,
    error: null
  });

  useEffect(() => {
    setState(s => ({ ...s, loading: true }));
    asyncFn()
      .then(data => setState({ data, loading: false, error: null }))
      .catch(error => setState({ data: null, loading: false, error }));
  }, deps);

  return state;
}
```

## State Management

### Zustand (Recommended for most cases)
```jsx
import { create } from 'zustand';

const useStore = create((set, get) => ({
  items: [],
  addItem: (item) => set(state => ({ 
    items: [...state.items, item] 
  })),
  removeItem: (id) => set(state => ({
    items: state.items.filter(i => i.id !== id)
  })),
  getTotal: () => get().items.reduce((sum, i) => sum + i.price, 0)
}));
```

### React Query for Server State
```jsx
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

function Users() {
  const queryClient = useQueryClient();
  
  const { data, isLoading } = useQuery({
    queryKey: ['users'],
    queryFn: fetchUsers,
    staleTime: 5 * 60 * 1000
  });

  const mutation = useMutation({
    mutationFn: createUser,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    }
  });
}
```

## Next.js App Router

### Server Components (Default)
```jsx
// app/users/page.jsx - Server Component
async function UsersPage() {
  const users = await db.users.findMany();
  return <UserList users={users} />;
}
```

### Client Components
```jsx
'use client';
// Only use when you need interactivity
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

### Server Actions
```jsx
// app/actions.js
'use server';

export async function createPost(formData) {
  const title = formData.get('title');
  await db.posts.create({ data: { title } });
  revalidatePath('/posts');
}
```

## Performance Optimization

1. **Memoization**: Use `useMemo` and `useCallback` wisely
2. **Code Splitting**: Dynamic imports with `next/dynamic`
3. **Image Optimization**: Use `next/image` with proper sizing
4. **Virtualization**: Use `react-window` for long lists

## File Structure
```
src/
├── app/                 # Next.js App Router
├── components/
│   ├── ui/             # Reusable UI components
│   └── features/       # Feature-specific components
├── hooks/              # Custom hooks
├── lib/                # Utilities and configs
├── stores/             # Zustand stores
└── types/              # TypeScript types
```
