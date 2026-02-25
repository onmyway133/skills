# React Native

## Overview

Mobile development best practices covering performance, layout, animations, and platform-specific patterns from Vercel's react-native-guidelines skill.

## Performance

### FlashList for Large Lists

```tsx
import { FlashList } from "@shopify/flash-list";

function ProductList({ products }) {
  return (
    <FlashList
      data={products}
      renderItem={({ item }) => <ProductCard product={item} />}
      estimatedItemSize={120}
      keyExtractor={(item) => item.id}
    />
  );
}

// Avoid FlatList for large datasets
// FlashList is significantly faster
```

### Memoization

```tsx
// Memoize expensive components
const ProductCard = memo(function ProductCard({ product }) {
  return (
    <View style={styles.card}>
      <Image source={{ uri: product.image }} style={styles.image} />
      <Text>{product.name}</Text>
      <Text>{product.price}</Text>
    </View>
  );
});

// Memoize callbacks
const handlePress = useCallback((id: string) => {
  navigation.navigate('Product', { id });
}, [navigation]);
```

### Image Optimization

```tsx
import FastImage from 'react-native-fast-image';

// Use FastImage for better caching
<FastImage
  source={{
    uri: imageUrl,
    priority: FastImage.priority.high,
    cache: FastImage.cacheControl.immutable,
  }}
  style={styles.image}
  resizeMode={FastImage.resizeMode.cover}
/>
```

## Layout

### Flexbox Defaults

```tsx
// React Native uses flexbox by default
// flexDirection: 'column' (unlike web's 'row')
const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  row: {
    flexDirection: 'row',
    gap: 12,
  },
});
```

### Safe Areas

```tsx
import { SafeAreaView, useSafeAreaInsets } from 'react-native-safe-area-context';

// Wrap entire screen
function Screen({ children }) {
  return (
    <SafeAreaView style={styles.container} edges={['top', 'bottom']}>
      {children}
    </SafeAreaView>
  );
}

// Or use hook for fine control
function Header() {
  const insets = useSafeAreaInsets();

  return (
    <View style={[styles.header, { paddingTop: insets.top }]}>
      <Text>Header</Text>
    </View>
  );
}
```

### Keyboard Avoiding

```tsx
import { KeyboardAvoidingView, Platform } from 'react-native';

function FormScreen() {
  return (
    <KeyboardAvoidingView
      behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
      style={styles.container}
    >
      <TextInput placeholder="Email" />
      <TextInput placeholder="Password" secureTextEntry />
      <Button title="Submit" onPress={handleSubmit} />
    </KeyboardAvoidingView>
  );
}
```

## Animations

### Reanimated

```tsx
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withSpring,
} from 'react-native-reanimated';

function AnimatedCard() {
  const scale = useSharedValue(1);

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ scale: scale.value }],
  }));

  const handlePressIn = () => {
    scale.value = withSpring(0.95);
  };

  const handlePressOut = () => {
    scale.value = withSpring(1);
  };

  return (
    <Pressable onPressIn={handlePressIn} onPressOut={handlePressOut}>
      <Animated.View style={[styles.card, animatedStyle]}>
        <Text>Press me</Text>
      </Animated.View>
    </Pressable>
  );
}
```

### Gesture Handler

```tsx
import { Gesture, GestureDetector } from 'react-native-gesture-handler';

function SwipeableCard() {
  const translateX = useSharedValue(0);

  const pan = Gesture.Pan()
    .onUpdate((e) => {
      translateX.value = e.translationX;
    })
    .onEnd(() => {
      translateX.value = withSpring(0);
    });

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ translateX: translateX.value }],
  }));

  return (
    <GestureDetector gesture={pan}>
      <Animated.View style={[styles.card, animatedStyle]}>
        <Text>Swipe me</Text>
      </Animated.View>
    </GestureDetector>
  );
}
```

## Platform-Specific Code

### Platform.select

```tsx
import { Platform, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  container: {
    ...Platform.select({
      ios: {
        shadowColor: '#000',
        shadowOffset: { width: 0, height: 2 },
        shadowOpacity: 0.25,
        shadowRadius: 4,
      },
      android: {
        elevation: 4,
      },
    }),
  },
});
```

### Platform-Specific Files

```
// File structure
components/
  Button.ios.tsx
  Button.android.tsx
  Button.tsx  // fallback

// Import automatically selects correct file
import { Button } from './components/Button';
```

### Feature Detection

```tsx
import { Platform } from 'react-native';

function HapticButton({ onPress, children }) {
  const handlePress = () => {
    if (Platform.OS === 'ios') {
      // iOS-specific haptic feedback
      Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Medium);
    }
    onPress();
  };

  return <Pressable onPress={handlePress}>{children}</Pressable>;
}
```

## State Management

### Zustand (Recommended)

```tsx
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import AsyncStorage from '@react-native-async-storage/async-storage';

const useStore = create(
  persist(
    (set) => ({
      user: null,
      setUser: (user) => set({ user }),
      logout: () => set({ user: null }),
    }),
    {
      name: 'app-storage',
      storage: createJSONStorage(() => AsyncStorage),
    }
  )
);
```

## External Reference

For comprehensive guidance, see [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills).
