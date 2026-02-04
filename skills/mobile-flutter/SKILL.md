---
name: mobile-flutter
description: Flutter development with Dart, state management patterns, and cross-platform mobile app architecture
tags: [flutter, dart, mobile, ios, android]
author: Antigravity Team
version: 1.0.0
---

# Flutter Development Skill

Cross-platform mobile development with Flutter.

## Basic Widgets

```dart
import 'package:flutter/material.dart';

class UserCard extends StatelessWidget {
  final String name;
  final String email;
  final VoidCallback? onTap;

  const UserCard({
    super.key,
    required this.name,
    required this.email,
    this.onTap,
  });

  @override
  Widget build(BuildContext context) {
    return Card(
      elevation: 2,
      margin: const EdgeInsets.all(8),
      child: InkWell(
        onTap: onTap,
        borderRadius: BorderRadius.circular(12),
        child: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Text(
                name,
                style: Theme.of(context).textTheme.titleMedium,
              ),
              const SizedBox(height: 4),
              Text(
                email,
                style: Theme.of(context).textTheme.bodySmall,
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

## State Management (Riverpod)

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';

// Simple state
final counterProvider = StateProvider<int>((ref) => 0);

// Async data
final usersProvider = FutureProvider<List<User>>((ref) async {
  final api = ref.read(apiProvider);
  return api.getUsers();
});

// State notifier for complex state
class CartNotifier extends StateNotifier<CartState> {
  CartNotifier() : super(const CartState());

  void addItem(Product product) {
    state = state.copyWith(
      items: [...state.items, CartItem(product: product, quantity: 1)],
    );
  }

  void removeItem(String productId) {
    state = state.copyWith(
      items: state.items.where((i) => i.product.id != productId).toList(),
    );
  }
}

final cartProvider = StateNotifierProvider<CartNotifier, CartState>(
  (ref) => CartNotifier(),
);

// Usage in widget
class CartScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final cart = ref.watch(cartProvider);
    
    return ListView.builder(
      itemCount: cart.items.length,
      itemBuilder: (context, index) => CartItemTile(cart.items[index]),
    );
  }
}
```

## Navigation (GoRouter)

```dart
final router = GoRouter(
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
      routes: [
        GoRoute(
          path: 'users/:id',
          builder: (context, state) => UserScreen(
            userId: state.pathParameters['id']!,
          ),
        ),
      ],
    ),
    GoRoute(
      path: '/login',
      builder: (context, state) => const LoginScreen(),
    ),
  ],
  redirect: (context, state) {
    final isLoggedIn = authService.isLoggedIn;
    final isLoginRoute = state.matchedLocation == '/login';
    
    if (!isLoggedIn && !isLoginRoute) return '/login';
    if (isLoggedIn && isLoginRoute) return '/';
    return null;
  },
);

// Navigation
context.go('/users/123');
context.push('/users/123');
context.pop();
```

## API Integration

```dart
import 'package:dio/dio.dart';

class ApiClient {
  final Dio _dio;

  ApiClient() : _dio = Dio(BaseOptions(
    baseUrl: 'https://api.example.com',
    connectTimeout: const Duration(seconds: 5),
    receiveTimeout: const Duration(seconds: 3),
  )) {
    _dio.interceptors.add(AuthInterceptor());
  }

  Future<List<User>> getUsers() async {
    final response = await _dio.get('/users');
    return (response.data as List)
        .map((json) => User.fromJson(json))
        .toList();
  }
}

class AuthInterceptor extends Interceptor {
  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    final token = storage.getToken();
    if (token != null) {
      options.headers['Authorization'] = 'Bearer $token';
    }
    handler.next(options);
  }
}
```

## Best Practices

1. **Use const** constructors for performance
2. **Riverpod** for state management
3. **GoRouter** for navigation
4. **Freezed** for immutable models
5. **Extract widgets** for reusability
