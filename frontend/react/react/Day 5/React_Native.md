# React Native: Mobile Applications Explained (Basic to Advanced)

## **Introduction to React Native**

### **Concept (70%)**

React Native is a **cross-platform framework** for building **native mobile applications** using JavaScript and React. Unlike hybrid frameworks (like Cordova/PhoneGap), React Native renders **actual native UI components**, not web views, resulting in performance close to fully native applications.

**Core Philosophy**: "Learn once, write anywhere" - write React code that runs on both iOS and Android with minimal platform-specific adjustments.

**Key Differences from React Web:**
- Uses native components (`<View>` instead of `<div>`, `<Text>` instead of `<p>`)
- Different styling system (Flexbox-based, but not CSS)
- No DOM access
- Bridge architecture for JavaScript-to-Native communication
- Platform-specific APIs (camera, geolocation, push notifications)

## **1. Basic Concepts & Setup**

### **Concept (70%)**

**Architecture Overview:**
1. **JavaScript Thread**: Runs your React Native code and business logic
2. **Native Thread**: Handles UI rendering and native modules
3. **Bridge**: Serializes messages between JavaScript and Native threads

**Development Environment Options:**
- **Expo**: Quick start, managed workflow, no native code required
- **React Native CLI**: Full control, access to native code, more complex setup

**Core Components Every Developer Should Know:**
- `<View>`: Container component (similar to `div`)
- `<Text>`: For displaying text
- `<Image>`: For displaying images
- `<ScrollView>`: Scrollable container
- `<TextInput>`: User text input
- `<Button>` / `<TouchableOpacity>`: Interactive elements

### **Code (30%)**

```jsx
// BASIC: Hello World in React Native
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';

function HelloWorldApp() {
  return (
    <View style={styles.container}>
      <Text style={styles.text}>Hello, React Native!</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#f5fcff',
  },
  text: {
    fontSize: 20,
    fontWeight: 'bold',
    color: '#333',
  },
});

export default HelloWorldApp;

// BASIC: Simple Login Screen
import React, { useState } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  StyleSheet,
  KeyboardAvoidingView,
} from 'react-native';

function LoginScreen() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleLogin = () => {
    console.log('Logging in with:', { email, password });
  };

  return (
    <KeyboardAvoidingView style={styles.container} behavior="padding">
      <Text style={styles.title}>Welcome Back</Text>
      
      <TextInput
        style={styles.input}
        placeholder="Email"
        value={email}
        onChangeText={setEmail}
        autoCapitalize="none"
        keyboardType="email-address"
      />
      
      <TextInput
        style={styles.input}
        placeholder="Password"
        value={password}
        onChangeText={setPassword}
        secureTextEntry
      />
      
      <TouchableOpacity style={styles.button} onPress={handleLogin}>
        <Text style={styles.buttonText}>Login</Text>
      </TouchableOpacity>
    </KeyboardAvoidingView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    justifyContent: 'center',
    backgroundColor: '#fff',
  },
  title: {
    fontSize: 28,
    fontWeight: 'bold',
    marginBottom: 40,
    textAlign: 'center',
  },
  input: {
    height: 50,
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    paddingHorizontal: 15,
    marginBottom: 20,
    fontSize: 16,
  },
  button: {
    height: 50,
    backgroundColor: '#007AFF',
    borderRadius: 8,
    justifyContent: 'center',
    alignItems: 'center',
    marginTop: 10,
  },
  buttonText: {
    color: '#fff',
    fontSize: 18,
    fontWeight: '600',
  },
});
```

---

## **2. Intermediate Concepts: Navigation & State Management**

### **Concept (70%)**

**Navigation Solutions:**
1. **React Navigation** (Most popular): JavaScript-based, highly customizable
2. **React Native Navigation** (Wix): Native navigation, better performance
3. **Expo Router**: File-based routing (like Next.js)

**State Management Patterns:**
- Local State: `useState`, `useReducer`
- Context API: For app-wide state
- Redux/MobX: For complex applications
- React Query/SWR: For server state

**Platform-Specific Code:**
- Use `Platform.OS` to detect platform
- Platform-specific file extensions (`.ios.js`, `.android.js`)
- Different styling for different platforms

### **Code (30%)**

```jsx
// INTERMEDIATE: Tab Navigation with React Navigation
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { createStackNavigator } from '@react-navigation/stack';
import Ionicons from 'react-native-vector-icons/Ionicons';

// Screens
function HomeScreen() {
  return (
    <View style={styles.screen}>
      <Text>Home Screen</Text>
    </View>
  );
}

function ProfileScreen() {
  return (
    <View style={styles.screen}>
      <Text>Profile Screen</Text>
    </View>
  );
}

function SettingsScreen() {
  return (
    <View style={styles.screen}>
      <Text>Settings Screen</Text>
    </View>
  );
}

const Tab = createBottomTabNavigator();
const Stack = createStackNavigator();

function MainTabs() {
  return (
    <Tab.Navigator
      screenOptions={({ route }) => ({
        tabBarIcon: ({ focused, color, size }) => {
          let iconName;
          if (route.name === 'Home') {
            iconName = focused ? 'home' : 'home-outline';
          } else if (route.name === 'Profile') {
            iconName = focused ? 'person' : 'person-outline';
          } else if (route.name === 'Settings') {
            iconName = focused ? 'settings' : 'settings-outline';
          }
          return <Ionicons name={iconName} size={size} color={color} />;
        },
        tabBarActiveTintColor: '#007AFF',
        tabBarInactiveTintColor: 'gray',
      })}
    >
      <Tab.Screen name="Home" component={HomeScreen} />
      <Tab.Screen name="Profile" component={ProfileScreen} />
      <Tab.Screen name="Settings" component={SettingsScreen} />
    </Tab.Navigator>
  );
}

// Stack Navigator for nested navigation
function AppNavigator() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen 
          name="Main" 
          component={MainTabs} 
          options={{ headerShown: false }}
        />
        <Stack.Screen name="Details" component={DetailsScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

// INTERMEDIATE: State Management with Context
import React, { createContext, useContext, useReducer } from 'react';

const AuthContext = createContext();

function authReducer(state, action) {
  switch (action.type) {
    case 'SIGN_IN':
      return {
        ...state,
        isAuthenticated: true,
        user: action.payload.user,
        token: action.payload.token,
      };
    case 'SIGN_OUT':
      return {
        ...state,
        isAuthenticated: false,
        user: null,
        token: null,
      };
    default:
      return state;
  }
}

function AuthProvider({ children }) {
  const [state, dispatch] = useReducer(authReducer, {
    isAuthenticated: false,
    user: null,
    token: null,
  });

  const signIn = async (credentials) => {
    // API call
    const response = await fetch('/api/login', {
      method: 'POST',
      body: JSON.stringify(credentials),
    });
    const data = await response.json();
    
    dispatch({
      type: 'SIGN_IN',
      payload: { user: data.user, token: data.token },
    });
  };

  const signOut = () => {
    dispatch({ type: 'SIGN_OUT' });
  };

  return (
    <AuthContext.Provider value={{ state, signIn, signOut }}>
      {children}
    </AuthContext.Provider>
  );
}

// Usage in component
function ProtectedScreen() {
  const { state } = useContext(AuthContext);
  
  if (!state.isAuthenticated) {
    return (
      <View style={styles.container}>
        <Text>Please sign in</Text>
      </View>
    );
  }
  
  return (
    <View style={styles.container}>
      <Text>Welcome, {state.user.name}!</Text>
    </View>
  );
}
```

---

## **3. Advanced Concepts: Performance & Native Modules**

### **Concept (70%)**

**Performance Optimization Techniques:**
1. **Virtualized Lists**: `FlatList`, `SectionList` for long lists
2. **Memoization**: `React.memo`, `useMemo`, `useCallback`
3. **Image Optimization**: Caching, resizing, progressive loading
4. **Hermes Engine**: Facebook's optimized JavaScript engine for React Native
5. **Reduce Bridge Traffic**: Batch operations, avoid unnecessary re-renders

**Native Modules & Bridges:**
- Extend React Native with native functionality
- Write platform-specific code in Java/Kotlin (Android) or Objective-C/Swift (iOS)
- Expose native APIs to JavaScript

**Advanced Architectures:**
- **Clean Architecture**: Separation of concerns
- **Feature-based Organization**: Scalable folder structure
- **Monorepos**: Shared code between mobile and web
- **Continuous Integration/Deployment**: Automated testing and deployment

### **Code (30%)**

```jsx
// ADVANCED: Performance - Virtualized List with Optimizations
import React, { useCallback, useState, useEffect } from 'react';
import {
  FlatList,
  View,
  Text,
  Image,
  StyleSheet,
  ActivityIndicator,
  RefreshControl,
} from 'react-native';

const PAGE_SIZE = 20;

function OptimizedProductList() {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(false);
  const [refreshing, setRefreshing] = useState(false);
  const [page, setPage] = useState(1);
  const [hasMore, setHasMore] = useState(true);

  // Memoized item renderer
  const renderItem = useCallback(({ item }) => (
    <ProductItem item={item} />
  ), []);

  // Memoized key extractor
  const keyExtractor = useCallback((item) => item.id.toString(), []);

  const loadProducts = async (pageNum = 1, isRefresh = false) => {
    if (loading && !isRefresh) return;
    
    setLoading(true);
    try {
      const response = await fetch(
        `https://api.example.com/products?page=${pageNum}&limit=${PAGE_SIZE}`
      );
      const newProducts = await response.json();
      
      if (isRefresh) {
        setProducts(newProducts);
      } else {
        setProducts(prev => [...prev, ...newProducts]);
      }
      
      setHasMore(newProducts.length === PAGE_SIZE);
      setPage(pageNum);
    } catch (error) {
      console.error('Failed to load products:', error);
    } finally {
      setLoading(false);
      setRefreshing(false);
    }
  };

  const handleRefresh = () => {
    setRefreshing(true);
    loadProducts(1, true);
  };

  const handleLoadMore = () => {
    if (!loading && hasMore) {
      loadProducts(page + 1);
    }
  };

  useEffect(() => {
    loadProducts();
  }, []);

  const renderFooter = () => {
    if (!loading) return null;
    return (
      <View style={styles.footer}>
        <ActivityIndicator size="small" />
      </View>
    );
  };

  return (
    <FlatList
      data={products}
      renderItem={renderItem}
      keyExtractor={keyExtractor}
      contentContainerStyle={styles.list}
      onEndReached={handleLoadMore}
      onEndReachedThreshold={0.5}
      ListFooterComponent={renderFooter}
      refreshControl={
        <RefreshControl
          refreshing={refreshing}
          onRefresh={handleRefresh}
          colors={['#007AFF']}
        />
      }
      initialNumToRender={10}
      maxToRenderPerBatch={10}
      windowSize={21}
      removeClippedSubviews={true}
    />
  );
}

// Memoized component for list items
const ProductItem = React.memo(({ item }) => (
  <View style={styles.item}>
    <Image
      source={{ uri: item.image }}
      style={styles.image}
      resizeMode="cover"
    />
    <View style={styles.details}>
      <Text style={styles.title} numberOfLines={2}>
        {item.title}
      </Text>
      <Text style={styles.price}>${item.price.toFixed(2)}</Text>
    </View>
  </View>
));

// ADVANCED: Creating a Custom Native Module (iOS - Swift)
/*
// 1. Create a Swift file: CustomModule.swift
import Foundation

@objc(CustomModule)
class CustomModule: NSObject {
  
  @objc
  func getDeviceInfo(_ resolve: @escaping RCTPromiseResolveBlock,
                     rejecter reject: @escaping RCTPromiseRejectBlock) {
    let deviceInfo = [
      "model": UIDevice.current.model,
      "systemName": UIDevice.current.systemName,
      "systemVersion": UIDevice.current.systemVersion,
      "batteryLevel": UIDevice.current.batteryLevel,
    ]
    resolve(deviceInfo)
  }
  
  @objc
  static func requiresMainQueueSetup() -> Bool {
    return true
  }
}
*/

// ADVANCED: JavaScript side using the native module
import { NativeModules } from 'react-native';

const { CustomModule } = NativeModules;

async function useNativeModule() {
  try {
    const deviceInfo = await CustomModule.getDeviceInfo();
    console.log('Device Info:', deviceInfo);
    return deviceInfo;
  } catch (error) {
    console.error('Failed to get device info:', error);
  }
}

// ADVANCED: Platform-specific components
import { Platform, View, Text, StyleSheet } from 'react-native';

function PlatformSpecificComponent() {
  return (
    <View style={styles.container}>
      {Platform.OS === 'ios' ? (
        <View style={styles.iosContainer}>
          <Text style={styles.iosText}>iOS Specific Design</Text>
        </View>
      ) : (
        <View style={styles.androidContainer}>
          <Text style={styles.androidText}>Android Specific Design</Text>
        </View>
      )}
    </View>
  );
}

// Or use platform-specific files:
// MyComponent.ios.js
// MyComponent.android.js
```

---

## **4. Advanced: Animations & Gestures**

### **Concept (70%)**

**Animation Libraries:**
1. **Animated API**: Built-in, declarative animations
2. **React Native Reanimated 2**: More powerful, runs on UI thread
3. **Lottie**: Vector animations from After Effects

**Gesture Handling:**
- **PanResponder**: Built-in gesture system
- **React Native Gesture Handler**: Better performance, native-driven gestures
- **React Native Reanimated Gestures**: Combined with animations

**Advanced UI Patterns:**
- Shared element transitions
- Parallax effects
- Interactive gestures
- Physics-based animations

### **Code (30%)**

```jsx
// ADVANCED: Reanimated 2 Animation
import React, { useEffect } from 'react';
import { View, StyleSheet, Dimensions } from 'react-native';
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withTiming,
  withSpring,
  withRepeat,
  Easing,
} from 'react-native-reanimated';

const { width } = Dimensions.get('window');

function AdvancedAnimationScreen() {
  const translateX = useSharedValue(0);
  const scale = useSharedValue(1);
  const rotation = useSharedValue(0);

  useEffect(() => {
    // Bouncing animation
    translateX.value = withRepeat(
      withTiming(width - 100, {
        duration: 2000,
        easing: Easing.inOut(Easing.ease),
      }),
      -1, // Infinite repeat
      true // Reverse
    );

    // Scaling animation
    scale.value = withRepeat(
      withSpring(1.5, { damping: 2, stiffness: 100 }),
      -1,
      true
    );

    // Rotation animation
    rotation.value = withRepeat(
      withTiming(360, { duration: 3000 }),
      -1,
      false
    );
  }, []);

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [
      { translateX: translateX.value },
      { scale: scale.value },
      { rotate: `${rotation.value}deg` },
    ],
  }));

  return (
    <View style={styles.container}>
      <Animated.View style={[styles.box, animatedStyle]} />
    </View>
  );
}

// ADVANCED: Gesture Handler with Reanimated
import {
  GestureDetector,
  Gesture,
} from 'react-native-gesture-handler';

function GestureAnimationScreen() {
  const translateX = useSharedValue(0);
  const translateY = useSharedValue(0);
  const scale = useSharedValue(1);

  const panGesture = Gesture.Pan()
    .onUpdate((event) => {
      translateX.value = event.translationX;
      translateY.value = event.translationY;
    })
    .onEnd(() => {
      translateX.value = withSpring(0);
      translateY.value = withSpring(0);
    });

  const pinchGesture = Gesture.Pinch()
    .onUpdate((event) => {
      scale.value = event.scale;
    })
    .onEnd(() => {
      scale.value = withSpring(1);
    });

  const composedGesture = Gesture.Simultaneous(panGesture, pinchGesture);

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [
      { translateX: translateX.value },
      { translateY: translateY.value },
      { scale: scale.value },
    ],
  }));

  return (
    <View style={styles.container}>
      <GestureDetector gesture={composedGesture}>
        <Animated.Image
          source={{ uri: 'https://example.com/image.jpg' }}
          style={[styles.image, animatedStyle]}
        />
      </GestureDetector>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  box: {
    width: 100,
    height: 100,
    backgroundColor: '#007AFF',
    borderRadius: 20,
  },
  image: {
    width: 200,
    height: 200,
    borderRadius: 10,
  },
});
```

---

## **5. Production-Ready Application Architecture**

### **Concept (70%)**

**Scalable Folder Structure:**
```
src/
├── api/           # API calls and configurations
├── assets/        # Images, fonts, videos
├── components/    # Reusable UI components
│   ├── common/    # Button, Input, etc.
│   └── features/  # Feature-specific components
├── constants/     # Colors, dimensions, strings
├── contexts/      # React Context providers
├── hooks/         # Custom hooks
├── navigation/    # Navigation configuration
├── screens/       # Screen components
├── services/      # Business logic, API clients
├── store/         # State management (Redux)
├── themes/        # Light/dark mode, styling
└── utils/         # Helper functions
```

**Essential Production Features:**
1. **Error Boundary & Crash Reporting**: Sentry, Crashlytics
2. **Analytics**: Mixpanel, Google Analytics
3. **Deep Linking**: Handle URLs and app links
4. **Push Notifications**: Firebase Cloud Messaging
5. **Offline Support**: AsyncStorage, WatermelonDB, Realm
6. **Biometric Authentication**: Face ID, Touch ID, fingerprint
7. **Code Push**: Over-the-air updates (Microsoft App Center)
8. **Security**: Keychain, encryption, SSL pinning

**Testing Strategy:**
- Unit Tests: Jest
- Component Tests: React Native Testing Library
- E2E Tests: Detox, Maestro
- Snapshot Tests: Component snapshots

### **Code (30%)**

```jsx
// ADVANCED: Production App with Multiple Features
import React, { useEffect } from 'react';
import { Provider as PaperProvider } from 'react-native-paper';
import { NavigationContainer } from '@react-navigation/native';
import { Provider as StoreProvider } from 'react-redux';
import { PersistGate } from 'redux-persist/integration/react';
import { SafeAreaProvider } from 'react-native-safe-area-context';
import { GestureHandlerRootView } from 'react-native-gesture-handler';
import { QueryClient, QueryClientProvider } from 'react-query';
import * as Sentry from '@sentry/react-native';

// Import app components
import { store, persistor } from './src/store';
import { theme } from './src/themes';
import AppNavigator from './src/navigation/AppNavigator';
import { AuthProvider } from './src/contexts/AuthContext';
import { NotificationProvider } from './src/contexts/NotificationContext';
import { OfflineProvider } from './src/contexts/OfflineContext';

// Initialize Sentry for error tracking
Sentry.init({
  dsn: 'YOUR_SENTRY_DSN',
  enableAutoSessionTracking: true,
});

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 minutes
      retry: 3,
    },
  },
});

function AppContent() {
  // Handle app state changes
  useEffect(() => {
    // Initialize analytics
    // Setup deep linking
    // Check for updates
  }, []);

  return (
    <OfflineProvider>
      <AuthProvider>
        <NotificationProvider>
          <QueryClientProvider client={queryClient}>
            <AppNavigator />
          </QueryClientProvider>
        </NotificationProvider>
      </AuthProvider>
    </OfflineProvider>
  );
}

function App() {
  return (
    <Sentry.TouchEventBoundary>
      <GestureHandlerRootView style={{ flex: 1 }}>
        <SafeAreaProvider>
          <StoreProvider store={store}>
            <PersistGate loading={null} persistor={persistor}>
              <PaperProvider theme={theme}>
                <NavigationContainer
                  linking={{
                    prefixes: ['myapp://', 'https://myapp.com'],
                    config: {
                      screens: {
                        Home: 'home',
                        Profile: 'profile/:id',
                        Settings: 'settings',
                      },
                    },
                  }}
                  fallback={<SplashScreen />}
                >
                  <AppContent />
                </NavigationContainer>
              </PaperProvider>
            </PersistGate>
          </StoreProvider>
        </SafeAreaProvider>
      </GestureHandlerRootView>
    </Sentry.TouchEventBoundary>
  );
}

// ADVANCED: Custom Hook for Biometric Authentication
import { Platform } from 'react-native';
import * as LocalAuthentication from 'expo-local-authentication';
import * as Keychain from 'react-native-keychain';

function useBiometricAuth() {
  const [isAvailable, setIsAvailable] = useState(false);
  const [biometryType, setBiometryType] = useState(null);

  useEffect(() => {
    checkBiometricAvailability();
  }, []);

  const checkBiometricAvailability = async () => {
    const compatible = await LocalAuthentication.hasHardwareAsync();
    const enrolled = await LocalAuthentication.isEnrolledAsync();
    const types = await LocalAuthentication.supportedAuthenticationTypesAsync();
    
    setIsAvailable(compatible && enrolled);
    
    if (types.includes(LocalAuthentication.AuthenticationType.FACIAL_RECOGNITION)) {
      setBiometryType('Face ID');
    } else if (types.includes(LocalAuthentication.AuthenticationType.FINGERPRINT)) {
      setBiometryType('Touch ID');
    }
  };

  const authenticate = async () => {
    try {
      const result = await LocalAuthentication.authenticateAsync({
        promptMessage: 'Authenticate to continue',
        fallbackLabel: 'Use passcode',
        disableDeviceFallback: false,
      });
      
      return result.success;
    } catch (error) {
      console.error('Biometric authentication failed:', error);
      return false;
    }
  };

  const storeCredentials = async (username, password) => {
    await Keychain.setInternetCredentials(
      'myapp',
      username,
      password,
      {
        accessible: Keychain.ACCESSIBLE.WHEN_UNLOCKED_THIS_DEVICE_ONLY,
        authenticationType: Keychain.AUTHENTICATION_TYPE.BIOMETRICS,
      }
    );
  };

  const getCredentials = async () => {
    const credentials = await Keychain.getInternetCredentials('myapp');
    return credentials;
  };

  return {
    isAvailable,
    biometryType,
    authenticate,
    storeCredentials,
    getCredentials,
  };
}
```

---

## **Learning Path & Best Practices**

### **Progression Path:**

1. **Beginner (1-2 months)**
   - Learn React Native basics
   - Build simple apps (Todo, Weather, Calculator)
   - Understand core components and styling

2. **Intermediate (3-6 months)**
   - Master navigation patterns
   - Implement state management
   - Integrate REST APIs
   - Add authentication

3. **Advanced (6+ months)**
   - Native module development
   - Performance optimization
   - Complex animations
   - Architecture patterns
   - Publishing to stores

### **Best Practices:**

1. **Code Organization**: Follow consistent folder structure
2. **Component Design**: Create reusable, composable components
3. **Performance**: Optimize images, virtualize lists, avoid unnecessary re-renders
4. **Testing**: Write tests for critical paths
5. **Security**: Store sensitive data securely, validate inputs
6. **Accessibility**: Add accessibility labels, support screen readers
7. **Internationalization**: Prepare for multiple languages
8. **Theming**: Support light/dark modes

### **When to Choose React Native:**

✅ **Choose React Native When:**
- Need to deploy on both iOS and Android
- Team knows JavaScript/React
- Don't need platform-specific advanced features
- Want faster development cycle

❌ **Consider Native When:**
- Need maximum performance (games, heavy animations)
- Require platform-specific advanced features
- App heavily relies on device hardware
- Have separate iOS and Android teams

### **Resources to Continue Learning:**

1. **Official Documentation**: reactnative.dev
2. **Expo**: expo.dev (great for beginners)
3. **React Native Community**: GitHub repositories, Discord
4. **Advanced Courses**: Udemy, Pluralsight, Frontend Masters
5. **Open Source Apps**: Study code from popular apps

React Native continues to evolve with improvements like the New Architecture (Fabric, TurboModules, JSI) making it even more performant. As you progress, you'll find that many concepts from React web translate well, but the mobile-specific considerations require dedicated learning and practice.