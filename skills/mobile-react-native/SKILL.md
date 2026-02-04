---
name: mobile-react-native
description: React Native development with Expo, navigation, native modules, and cross-platform patterns
tags: [react-native, expo, mobile, ios, android]
author: Antigravity Team
version: 1.0.0
---

# React Native Development Skill

Build cross-platform mobile apps.

## Expo Setup

```bash
npx create-expo-app@latest my-app
cd my-app
npx expo start
```

## Navigation

```javascript
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';

const Stack = createNativeStackNavigator();
const Tab = createBottomTabNavigator();

function HomeStack() {
  return (
    <Stack.Navigator>
      <Stack.Screen name="Home" component={HomeScreen} />
      <Stack.Screen name="Details" component={DetailsScreen} />
    </Stack.Navigator>
  );
}

export default function App() {
  return (
    <NavigationContainer>
      <Tab.Navigator>
        <Tab.Screen 
          name="HomeTab" 
          component={HomeStack}
          options={{ tabBarIcon: ({color}) => <HomeIcon color={color} /> }}
        />
        <Tab.Screen name="Profile" component={ProfileScreen} />
      </Tab.Navigator>
    </NavigationContainer>
  );
}
```

## Styling

```javascript
import { StyleSheet, View, Text, useWindowDimensions } from 'react-native';

function Card({ title, children }) {
  const { width } = useWindowDimensions();
  const isTablet = width > 768;
  
  return (
    <View style={[styles.card, isTablet && styles.cardTablet]}>
      <Text style={styles.title}>{title}</Text>
      {children}
    </View>
  );
}

const styles = StyleSheet.create({
  card: {
    backgroundColor: '#fff',
    borderRadius: 12,
    padding: 16,
    marginVertical: 8,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 8,
    elevation: 3
  },
  cardTablet: {
    maxWidth: 600,
    alignSelf: 'center'
  },
  title: {
    fontSize: 18,
    fontWeight: '600',
    marginBottom: 8
  }
});
```

## State Management

```javascript
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import AsyncStorage from '@react-native-async-storage/async-storage';

const useStore = create(
  persist(
    (set) => ({
      user: null,
      setUser: (user) => set({ user }),
      logout: () => set({ user: null })
    }),
    {
      name: 'user-storage',
      storage: createJSONStorage(() => AsyncStorage)
    }
  )
);
```

## API Calls

```javascript
import { useQuery, useMutation } from '@tanstack/react-query';

function UserList() {
  const { data, isLoading, error, refetch } = useQuery({
    queryKey: ['users'],
    queryFn: () => fetch('/api/users').then(r => r.json())
  });

  if (isLoading) return <ActivityIndicator />;
  
  return (
    <FlatList
      data={data}
      keyExtractor={item => item.id}
      renderItem={({ item }) => <UserCard user={item} />}
      onRefresh={refetch}
      refreshing={isLoading}
    />
  );
}
```

## Native Modules (Expo)

```javascript
import * as Location from 'expo-location';
import * as ImagePicker from 'expo-image-picker';
import * as Notifications from 'expo-notifications';

async function getLocation() {
  const { status } = await Location.requestForegroundPermissionsAsync();
  if (status !== 'granted') return null;
  
  return Location.getCurrentPositionAsync({});
}

async function pickImage() {
  const { status } = await ImagePicker.requestMediaLibraryPermissionsAsync();
  if (status !== 'granted') return null;
  
  return ImagePicker.launchImageLibraryAsync({
    mediaTypes: ImagePicker.MediaTypeOptions.Images,
    allowsEditing: true,
    quality: 0.8
  });
}
```

## Best Practices

1. **Use FlatList** for long lists (not ScrollView)
2. **Memoize** expensive components
3. **Handle offline** states
4. **Test on real devices**
5. **Use Hermes** for performance
