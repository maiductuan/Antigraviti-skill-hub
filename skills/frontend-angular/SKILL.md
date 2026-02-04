---
name: frontend-angular
description: Angular development with RxJS, NgRx state management, standalone components, and enterprise patterns
tags: [angular, rxjs, ngrx, frontend, typescript]
author: Antigravity Team
version: 1.0.0
---

# Angular Development Skill

Enterprise Angular development patterns.

## Standalone Components

```typescript
import { Component, inject, signal, computed } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule } from '@angular/router';

@Component({
  selector: 'app-user-card',
  standalone: true,
  imports: [CommonModule, RouterModule],
  template: `
    <div class="card">
      <h3>{{ user().name }}</h3>
      <p>{{ user().email }}</p>
      <button (click)="toggleDetails()">
        {{ showDetails() ? 'Hide' : 'Show' }} Details
      </button>
      @if (showDetails()) {
        <div class="details">
          <p>Role: {{ user().role }}</p>
        </div>
      }
    </div>
  `
})
export class UserCardComponent {
  user = signal({ name: 'John', email: 'john@example.com', role: 'Admin' });
  showDetails = signal(false);
  
  toggleDetails() {
    this.showDetails.update(v => !v);
  }
}
```

## Services & Dependency Injection

```typescript
import { Injectable, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable, catchError, retry, shareReplay } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class UserService {
  private http = inject(HttpClient);
  private baseUrl = '/api/users';
  
  private users$ = this.http.get<User[]>(this.baseUrl).pipe(
    retry(3),
    shareReplay(1)
  );

  getUsers(): Observable<User[]> {
    return this.users$;
  }

  getUser(id: string): Observable<User> {
    return this.http.get<User>(`${this.baseUrl}/${id}`);
  }

  createUser(user: CreateUserDto): Observable<User> {
    return this.http.post<User>(this.baseUrl, user);
  }
}
```

## NgRx Store

```typescript
// actions
export const loadUsers = createAction('[Users] Load');
export const loadUsersSuccess = createAction('[Users] Load Success', props<{ users: User[] }>());
export const loadUsersFailure = createAction('[Users] Load Failure', props<{ error: string }>());

// reducer
export const usersReducer = createReducer(
  initialState,
  on(loadUsers, state => ({ ...state, loading: true })),
  on(loadUsersSuccess, (state, { users }) => ({ ...state, users, loading: false })),
  on(loadUsersFailure, (state, { error }) => ({ ...state, error, loading: false }))
);

// effects
export const loadUsers$ = createEffect((
  actions$ = inject(Actions),
  userService = inject(UserService)
) => actions$.pipe(
  ofType(loadUsers),
  exhaustMap(() => userService.getUsers().pipe(
    map(users => loadUsersSuccess({ users })),
    catchError(error => of(loadUsersFailure({ error: error.message })))
  ))
), { functional: true });

// selectors
export const selectUsers = createSelector(selectUsersState, state => state.users);
export const selectLoading = createSelector(selectUsersState, state => state.loading);
```

## RxJS Patterns

```typescript
// Debounce search
searchControl.valueChanges.pipe(
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(query => this.searchService.search(query))
).subscribe(results => this.results = results);

// Combine multiple observables
combineLatest([users$, filters$]).pipe(
  map(([users, filters]) => users.filter(u => matchesFilters(u, filters)))
);

// Auto unsubscribe with takeUntilDestroyed
private destroyRef = inject(DestroyRef);

ngOnInit() {
  this.dataService.getData().pipe(
    takeUntilDestroyed(this.destroyRef)
  ).subscribe(data => this.data = data);
}
```

## Best Practices

1. **Use standalone components** (Angular 17+)
2. **Signals** for simple state
3. **NgRx** for complex state
4. **OnPush** change detection
5. **Lazy loading** for routes
