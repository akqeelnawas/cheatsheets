
# 📱 React Native Comprehensive Cheat Sheet

## 🔧 Project Setup

```bash
npx react-native init MyApp
cd MyApp
npx react-native run-android  # Android
npx react-native run-ios      # iOS
```

## 🧱 File Structure

```
MyApp/
├── android/
├── ios/
├── src/
│   ├── components/
│   ├── screens/
│   ├── navigation/
│   └── utils/
├── App.js
└── package.json
```

## 🧠 State Management

- **React State**: `useState`, `useReducer`
- **Context API**: `createContext`, `useContext`
- **Redux Toolkit**: State container
- **Recoil**, **Zustand**, **Jotai**: Lightweight alternatives

## 🔄 Navigation

Install React Navigation:
```bash
npm install @react-navigation/native
npm install react-native-screens react-native-safe-area-context
npm install @react-navigation/native-stack
```

Usage:
```js
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';

const Stack = createNativeStackNavigator();

<NavigationContainer>
  <Stack.Navigator>
    <Stack.Screen name="Home" component={HomeScreen} />
  </Stack.Navigator>
</NavigationContainer>
```

## 🎨 Styling

- **Stylesheet API**
```js
import { StyleSheet } from 'react-native';
const styles = StyleSheet.create({ button: { padding: 10 } });
```

- **Tailwind RN**, **styled-components**, **emotion-native**

## ⚙️ Components

```js
import { View, Text, Button, TextInput, ScrollView } from 'react-native';
```

## 📦 Async Storage

```bash
npm install @react-native-async-storage/async-storage
```

```js
import AsyncStorage from '@react-native-async-storage/async-storage';

await AsyncStorage.setItem('@key', 'value');
const value = await AsyncStorage.getItem('@key');
```

## 📡 Networking

```js
fetch('https://api.example.com/data')
  .then(res => res.json())
  .then(data => console.log(data));
```

Or use Axios:
```bash
npm install axios
```

## 🧪 Testing

- **Jest** (unit tests)
- **React Native Testing Library** (UI tests)

```bash
npm install --save-dev @testing-library/react-native jest
```

## 📲 Native Modules

Use [React Native Modules](https://reactnative.dev/docs/native-modules-intro) for bridging custom native code.

## 🧰 Helpful Libraries

- **Formik** + **Yup**: Forms and validation
- **React Query / SWR**: Server-state management
- **Lottie**: Animations
- **react-native-device-info**
- **react-native-firebase**

## 🔍 Debugging Tools

- React DevTools
- Flipper
- Reactotron

## 🧩 Effects & Lifecycles

```js
useEffect(() => {
  // Component did mount
  return () => {
    // Component will unmount
  };
}, []);
```

## 🔒 Permissions

```bash
npm install react-native-permissions
```

```js
import { request, PERMISSIONS } from 'react-native-permissions';
request(PERMISSIONS.ANDROID.CAMERA);
```

## 📦 Deployment

### Android
```bash
cd android
./gradlew assembleRelease
```

### iOS
```bash
cd ios
xcodebuild -scheme MyApp -configuration Release
```

---

🧠 **Tip**: Modularize your app, manage environments (`react-native-config`), and use CI/CD (Fastlane, GitHub Actions) for scalability.

