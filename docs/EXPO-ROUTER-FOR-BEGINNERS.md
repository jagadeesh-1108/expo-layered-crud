# Expo Router for Beginners: The Complete Guide to File-Based Navigation in React Native

*Learn how screens, folders and URLs fit together in a modern React Native app, with code you can copy and run.*

Every mobile app has the same question at its centre: **how does the user move from one screen to another?** Login to Home. Home to a product. A product to the cart. A tab bar at the bottom. A pop-up modal. A link in an email that opens straight to an order.

In React Native, this is called **navigation**, and for years it meant writing a lot of setup code by hand. Expo Router changes that. You create a file, and that file becomes a screen with its own URL.

This guide is written for students and beginners. By the end you will know:

- What Expo Router is and the problem it solves
- How it compares with React Navigation, and why most new Expo apps now use Expo Router
- How to set up a project and how folders turn into routes
- The core building blocks: layouts, stacks, tabs, dynamic routes, groups and navigation
- Real-world patterns: login-protected screens, modals, deep links, typed routes and API routes
- Common beginner mistakes, best practices, and the best links to keep learning

You only need basic React knowledge (components, props, state) to follow along.

## What is Expo Router?

**Expo Router is a file-based router for React Native and web apps.** Every file you put inside the `app` folder automatically becomes a screen, and the file's path becomes that screen's URL.

If you have used Next.js on the web, this will feel familiar. If you have not, here is the simplest way to think about it:

> **Your folder structure *is* your navigation.**

```
app/
  index.tsx        ->  /
  about.tsx        ->  /about
  settings.tsx     ->  /settings
```

Create `about.tsx`, and you can already navigate to `/about`. No registering screens, no navigator config, no linking map.

### Three ideas to remember

1. **Files are screens.** A file in `app/` that exports a React component is a route.
2. **Every screen has a URL.** Even on iOS and Android. That is what makes deep linking (opening a specific screen from a link) work automatically.
3. **It is built on React Navigation.** Expo Router does not replace React Navigation, it sits on top of it. The stacks, tabs and drawers you get are React Navigation navigators, with the same options and animations.

One codebase, one routing system, three platforms: Android, iOS and the web.

## Expo Router vs React Navigation: what's the difference?

**Short answer: React Navigation is the engine, Expo Router is the steering wheel built around it.** You are not choosing between two competing libraries so much as choosing how you want to *describe* your navigation.

### The same app, two ways

Here is a tiny app with a Home screen and a Profile screen.

**With React Navigation (classic dynamic API)**, you register every screen in code:

```tsx
// App.tsx
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import HomeScreen from './screens/HomeScreen';
import ProfileScreen from './screens/ProfileScreen';

const Stack = createNativeStackNavigator();

export default function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Profile" component={ProfileScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

// Somewhere in HomeScreen
navigation.navigate('Profile', { id: '42' });
```

**With Expo Router**, you just create files:

```
app/
  _layout.tsx         // says: "use a Stack"
  index.tsx           // Home  ->  /
  profile/[id].tsx    // Profile  ->  /profile/42
```

```tsx
// app/_layout.tsx
import { Stack } from 'expo-router';

export default function RootLayout() {
  return <Stack />;
}

// Somewhere in app/index.tsx
router.push('/profile/42');
```

No `NavigationContainer`, no screen registration, no name strings. The file tree does that work.

### Side-by-side comparison

**1. How routes are defined**

- React Navigation: in JavaScript, by registering screens inside navigators.
- Expo Router: by the files and folders inside `app/`.

**2. Deep linking (opening a screen from a URL)**

- React Navigation: you write a `linking` config that maps URLs to screen names (the newer static API can generate this for you).
- Expo Router: automatic. Every screen already has a URL.

**3. Web support**

- React Navigation: works on web, but URLs need configuration.
- Expo Router: real URLs out of the box, plus static rendering for SEO on web.

**4. TypeScript safety**

- React Navigation: you write a param list type for every navigator (or use the static API).
- Expo Router: typed routes are generated from your files, so a typo like `/porfile` becomes a compile error.

**5. Navigating**

- React Navigation: `navigation.navigate('Profile', { id })`, using screen names.
- Expo Router: `router.push('/profile/42')` or `<Link href="/profile/42">`, using URLs.

**6. Code splitting and lazy loading**

- React Navigation: you set it up yourself.
- Expo Router: routes are evaluated lazily, and on web each route can be split into its own bundle.

**7. Backend endpoints**

- React Navigation: not part of the library.
- Expo Router: API routes (`+api.ts` files) let you write server endpoints in the same project.

**8. Flexibility**

- React Navigation: full control. You can build navigators at runtime, any way you like.
- Expo Router: a convention you follow. Very flexible, but the structure lives in the file system.

### So why choose Expo Router?

- **Less boilerplate.** Beginners spend time on screens, not on wiring.
- **Deep links for free.** Push notifications, emails, QR codes and shared links can open the exact screen.
- **One mental model for mobile and web.** URLs work the same everywhere.
- **Easier to read a new codebase.** Open the `app/` folder and you can see every screen in the app.
- **Type-safe navigation** with almost no extra code.
- **It is the default.** New projects from `create-expo-app` start with Expo Router, and the Expo docs and tutorials are written for it.
- **You keep React Navigation's power.** Stack, Tabs and Drawer options are the same React Navigation options underneath.

### When React Navigation alone still makes sense

- You maintain a large existing app already built on React Navigation, and a migration is not worth the cost right now.
- You need navigators that are fully generated at runtime (for example, screens decided by a server config).
- Your team prefers explicit configuration over file conventions.

For a brand-new Expo project, especially as a learner, **start with Expo Router.**

## Getting started: your first Expo Router project

The fastest way is to let Expo create the project for you. The default template already includes Expo Router.

```bash
npx create-expo-app@latest my-app
cd my-app
npx expo start
```

Scan the QR code with the **Expo Go** app on your phone, or press `i` (iOS simulator), `a` (Android emulator) or `w` (web).

Want to start with an empty `app` folder and build everything yourself? Run `npm run reset-project` inside the new project. It moves the example code aside so you get a clean start.

### Two settings that make it work

If you open `package.json`, you will see the entry point points to Expo Router:

```json
{
  "main": "expo-router/entry"
}
```

And `app.json` has a `scheme`, which is the prefix used for deep links (for example `myapp://profile/42`):

```json
{
  "expo": {
    "scheme": "myapp"
  }
}
```

Adding Expo Router to an older project? Follow the manual steps in the [official installation guide](https://docs.expo.dev/router/installation/).

### The file naming rules (the most important part)

Learn these and you understand 80% of Expo Router.

```
app/
  _layout.tsx            Root layout: wraps every screen
  index.tsx              /
  about.tsx              /about
  +not-found.tsx         Shown for any URL that doesn't exist

  products/
    index.tsx            /products
    [id].tsx             /products/1, /products/abc ...

  (tabs)/                Group: NOT part of the URL
    _layout.tsx          Tab bar layout
    home.tsx             /home
    profile.tsx          /profile

  docs/
    [...slug].tsx        /docs/a, /docs/a/b/c ...

  api/
    hello+api.ts         Server endpoint: GET /api/hello
```

- **`index.tsx`** is the default screen of a folder, like `index.html` on a website.
- **`_layout.tsx`** wraps every screen in its folder. This is where you choose Stack, Tabs or Drawer.
- **`[id].tsx`** (square brackets) is a **dynamic route**. The part of the URL in that position becomes a parameter called `id`.
- **`[...slug].tsx`** is a **catch-all route**. It matches any number of URL segments.
- **`(group)`** (round brackets) organises files or shares a layout **without** adding anything to the URL. `app/(tabs)/home.tsx` is just `/home`.
- **`+not-found.tsx`** is your 404 screen.
- **`+api.ts`** files are server-side API routes, not screens.

Anything that is not a screen (components, hooks, utilities) should live **outside** the `app` folder, for example in `components/` or `src/`. Otherwise Expo Router will try to treat it as a route.

## Core concepts, with code

### 1. Layouts: the frame around your screens

A `_layout.tsx` file decides **how** the screens in its folder are shown. The root layout `app/_layout.tsx` is also the place for things every screen needs: fonts, theme providers, auth context, a query client.

```tsx
// app/_layout.tsx
import { Stack } from 'expo-router';

export default function RootLayout() {
  return (
    <Stack>
      <Stack.Screen name="index" options={{ title: 'Home' }} />
      <Stack.Screen name="about" options={{ title: 'About us' }} />
    </Stack>
  );
}
```

You get four kinds of layout:

- **`Stack`**: screens pile on top of each other, with a back button and slide animation. The most common choice.
- **`Tabs`**: a bottom tab bar.
- **`Drawer`**: a side menu (install `@react-navigation/drawer` and its peer packages first, see the [Drawer docs](https://docs.expo.dev/router/advanced/drawer/)).
- **`Slot`**: renders the current child with no navigation UI at all. Useful for custom designs.

Tip: you do not have to list every screen in a `Stack`. Screens are added automatically; you only list them when you want to set options.

### 2. Stack navigation

Think of a stack like plates: each new screen goes on top, and **Back** removes the top plate.

You can also set the header title from **inside** the screen:

```tsx
// app/about.tsx
import { Stack } from 'expo-router';
import { Text, View } from 'react-native';

export default function About() {
  return (
    <View style={{ flex: 1, alignItems: 'center', justifyContent: 'center' }}>
      <Stack.Screen options={{ title: 'About' }} />
      <Text>About this app</Text>
    </View>
  );
}
```

### 3. Tabs

Put tab screens in a group with their own layout:

```
app/
  _layout.tsx
  (tabs)/
    _layout.tsx
    index.tsx        // Home tab  ->  /
    settings.tsx     // Settings tab  ->  /settings
```

```tsx
// app/(tabs)/_layout.tsx
import { Tabs } from 'expo-router';
import Ionicons from '@expo/vector-icons/Ionicons';

export default function TabLayout() {
  return (
    <Tabs screenOptions={{ tabBarActiveTintColor: '#2563eb' }}>
      <Tabs.Screen
        name="index"
        options={{
          title: 'Home',
          tabBarIcon: ({ color, size }) => <Ionicons name="home" color={color} size={size} />,
        }}
      />
      <Tabs.Screen
        name="settings"
        options={{
          title: 'Settings',
          tabBarIcon: ({ color, size }) => <Ionicons name="settings" color={color} size={size} />,
        }}
      />
    </Tabs>
  );
}
```

And in the root layout, show the whole tab group as one screen of the stack:

```tsx
// app/_layout.tsx
import { Stack } from 'expo-router';

export default function RootLayout() {
  return (
    <Stack>
      <Stack.Screen name="(tabs)" options={{ headerShown: false }} />
    </Stack>
  );
}
```

This "Stack on the outside, Tabs on the inside" pattern is how most real apps are built. A details screen pushed on the outer stack covers the tab bar, just like Instagram or WhatsApp.

Newer Expo SDKs also offer **native tabs** (`expo-router/unstable-native-tabs`), which use the platform's real tab bar. It is still marked unstable, so stick with `Tabs` while learning.

### 4. Moving between screens

There are two ways to navigate.

**Declarative, with `<Link>`** (like an `<a>` tag on the web):

```tsx
import { Link } from 'expo-router';
import { Pressable, Text } from 'react-native';

<Link href="/about">Go to About</Link>

// Make any component a link with asChild
<Link href="/settings" asChild>
  <Pressable>
    <Text>Open settings</Text>
  </Pressable>
</Link>
```

**Imperative, with `router`** (inside a function, for example after a form submits):

```tsx
import { router } from 'expo-router';

router.push('/products/42');   // open a new screen on top
router.replace('/home');       // swap the current screen (no going back)
router.back();                 // go back one screen
router.navigate('/settings');  // go to a screen, reusing it if it is already open
router.dismissAll();           // close every screen in the current stack
```

You can also get the same object with the `useRouter()` hook inside a component.

When should you use which?

- Use **`push`** for normal "go forward" navigation.
- Use **`replace`** after login or onboarding, so the user cannot press Back to return to the login screen.
- Use **`back`** for custom back buttons.

### 5. Dynamic routes and URL parameters

A product list cannot have one file per product. Instead you write one file with square brackets:

```tsx
// app/products/[id].tsx
import { useLocalSearchParams } from 'expo-router';
import { Text, View } from 'react-native';

export default function ProductDetails() {
  const { id } = useLocalSearchParams<{ id: string }>();

  return (
    <View>
      <Text>Showing product {id}</Text>
    </View>
  );
}
```

Navigate to it with a string or with an object (the object form avoids string mistakes):

```tsx
<Link href="/products/42">Product 42</Link>

<Link href={{ pathname: '/products/[id]', params: { id: '42' } }}>
  Product 42
</Link>

// Extra values become query parameters: /products/42?color=red
router.push({ pathname: '/products/[id]', params: { id: '42', color: 'red' } });
```

Two hooks read parameters:

- **`useLocalSearchParams()`**: params for **this** screen only. Use this almost always.
- **`useGlobalSearchParams()`**: params of whatever URL is currently focused, even from other screens. It re-renders more often, so use it only when you really need it.

Remember that params from a URL are always **strings**. Convert them yourself, for example `Number(id)`.

### 6. Groups: organise without changing URLs

Round-bracket folders are perfect for separating parts of your app that need different layouts:

```
app/
  (auth)/
    _layout.tsx     // simple stack, no tabs
    login.tsx       // /login
    register.tsx    // /register
  (app)/
    _layout.tsx     // tabs
    index.tsx       // /
    orders.tsx      // /orders
```

The URLs stay clean (`/login`, not `/(auth)/login`), but each group gets its own layout. This is the foundation for the login flow in the next section.

## Real-world patterns you will use in every app

### 1. Login-protected screens (Protected Routes)

Almost every app has screens only logged-in users can see. Expo Router (SDK 53 and newer) has a built-in way to do this: **`Stack.Protected`**. When the `guard` is `false`, those screens cannot be opened, and the user is sent to the first screen they are allowed to see.

```tsx
// app/_layout.tsx
import { Stack } from 'expo-router';
import { AuthProvider, useAuth } from '@/context/auth';

export default function RootLayout() {
  return (
    <AuthProvider>
      <RootNavigator />
    </AuthProvider>
  );
}

function RootNavigator() {
  const { isLoggedIn } = useAuth();

  return (
    <Stack screenOptions={{ headerShown: false }}>
      <Stack.Protected guard={isLoggedIn}>
        <Stack.Screen name="(app)" />
      </Stack.Protected>

      <Stack.Protected guard={!isLoggedIn}>
        <Stack.Screen name="(auth)" />
      </Stack.Protected>
    </Stack>
  );
}
```

When the user logs in, `isLoggedIn` becomes `true`, the `(app)` group unlocks and the login screens lock. Logging out does the reverse. No manual redirects needed.

On older SDKs you will see the same idea done with the `<Redirect href="/login" />` component inside a layout. It still works, and you will meet it in older tutorials.

### 2. Modals

A modal is a screen that slides up over the current one: a filter sheet, a "create post" form, a confirmation.

```
app/
  _layout.tsx
  index.tsx
  create-post.tsx
```

```tsx
// app/_layout.tsx
<Stack>
  <Stack.Screen name="index" />
  <Stack.Screen name="create-post" options={{ presentation: 'modal' }} />
</Stack>
```

Open it like any other screen with `router.push('/create-post')`, and close it with `router.back()`.

### 3. Deep linking: open a screen from a link

Because every screen already has a URL, these links work with no extra code:

- `myapp://products/42` opens the product screen in your installed app
- `https://yourdomain.com/products/42` opens the same screen on the web (and in the app too, once you set up universal links / app links)

Test a deep link on a simulator while `npx expo start` is running:

```bash
npx uri-scheme open "myapp://products/42" --ios
npx uri-scheme open "myapp://products/42" --android
```

This is what makes push notifications, marketing emails, QR codes and "share this product" buttons land on the right screen.

### 4. Typed routes: catch broken links before they ship

Turn on typed routes in `app.json` (if your template has not already):

```json
{
  "expo": {
    "experiments": {
      "typedRoutes": true
    }
  }
}
```

Run `npx expo start` once, and Expo generates types from your `app` folder. Now this is a TypeScript error:

```tsx
<Link href="/prodcuts/42">Oops</Link>   // Error: this route does not exist
```

Rename a file, and every broken link in the project lights up red. For beginners, this alone saves hours.

### 5. A custom 404 screen

```tsx
// app/+not-found.tsx
import { Link, Stack } from 'expo-router';
import { Text, View } from 'react-native';

export default function NotFound() {
  return (
    <View style={{ flex: 1, alignItems: 'center', justifyContent: 'center' }}>
      <Stack.Screen options={{ title: 'Oops!' }} />
      <Text>This screen doesn't exist.</Text>
      <Link href="/">Go to home</Link>
    </View>
  );
}
```

### 6. API routes: a backend inside your app project

Files ending in `+api.ts` run on a server, not on the phone. They are handy for hiding API keys or small backend logic.

```ts
// app/api/hello+api.ts
export function GET(request: Request) {
  return Response.json({ message: 'Hello from the server' });
}
```

Call it from your app with `fetch('/api/hello')`. To use API routes in production, set `web.output` to `"server"` in `app.json` and deploy the server (for example with EAS Hosting). The [API routes guide](https://docs.expo.dev/router/reference/api-routes/) explains the details.

### 7. Handy hooks

- **`usePathname()`**: the current path, e.g. `/products/42`. Great for highlighting the active menu item.
- **`useSegments()`**: the path split into pieces, e.g. `['(app)', 'products', '[id]']`.
- **`useFocusEffect()`**: run code every time a screen comes into focus, for example to refresh data.
- **`useNavigation()`**: access the underlying React Navigation object when you need something advanced.

### Putting it together: a real app structure

Here is how a small shopping app might look:

```
app/
  _layout.tsx                 // Root Stack + AuthProvider + Stack.Protected
  +not-found.tsx

  (auth)/
    _layout.tsx               // Stack
    login.tsx                 // /login
    register.tsx              // /register

  (app)/
    _layout.tsx               // Stack
    (tabs)/
      _layout.tsx             // Tabs: Home, Cart, Profile
      index.tsx               // /
      cart.tsx                // /cart
      profile.tsx             // /profile
    products/[id].tsx         // /products/42 (covers the tab bar)
    orders/[orderId].tsx      // /orders/1001
    checkout.tsx              // /checkout (presentation: 'modal')

components/                   // reusable UI, NOT routes
hooks/
services/
```

Read the tree from top to bottom and you can explain the whole app's navigation in one minute. That readability is the real superpower of Expo Router.

## Common beginner mistakes (and how to fix them)

1. **Putting components inside `app/`.** A `Button.tsx` in `app/` becomes a route called `/Button`. Keep reusable code in `components/`.
2. **Forgetting the default export.** Every route file must `export default` a component, or the screen will not load.
3. **Confusing `[id]` and `(group)`.** Square brackets = a value in the URL. Round brackets = folder for organising, invisible in the URL.
4. **Two files for the same URL.** `app/home.tsx` and `app/(tabs)/home.tsx` both mean `/home`. Expo Router cannot decide which one to show.
5. **Using `push` after login.** The user can press Back and land on the login screen again. Use `replace`, or better, `Stack.Protected`.
6. **Treating params as numbers.** URL params are strings. Write `Number(id)` before doing maths.
7. **Reaching for `useGlobalSearchParams` by default.** It re-renders on every URL change. Use `useLocalSearchParams` unless you have a reason not to.
8. **Putting secrets in app code.** Anything in your app bundle can be read by users. Keep secret keys on a server, for example in an API route.

## Best practices

- **Keep `app/` thin.** Route files should mostly import a screen component and render it. Put the real UI and logic in feature folders.
- **Name files the way you want URLs to look:** lowercase with dashes, like `order-history.tsx`.
- **Use groups to separate flows:** `(auth)`, `(app)`, `(tabs)`, `(onboarding)`.
- **Turn on typed routes** from day one.
- **Prefer the object form of `href`** (`{ pathname, params }`) for dynamic routes.
- **Do global setup in the root `_layout.tsx`:** fonts, theme, auth, data providers.
- **Test your deep links early,** not the week before launch.

## Quick cheat sheet

```
index.tsx            default screen of a folder
_layout.tsx          navigator for a folder (Stack / Tabs / Drawer / Slot)
[id].tsx             dynamic segment          useLocalSearchParams()
[...slug].tsx        catch-all segment
(group)/             organise, not in the URL
+not-found.tsx       404 screen
name+api.ts          server API route

<Link href="/x" />   navigate in JSX
router.push()        go forward
router.replace()     swap, no back
router.back()        go back
Stack.Protected      guard screens behind login
```

## Best resources to keep learning

**Official Expo docs (start here)**

- [Introduction to Expo Router](https://docs.expo.dev/router/introduction/): what it is and why it exists
- [Install Expo Router](https://docs.expo.dev/router/installation/): new and existing projects
- [Core concepts of file-based routing](https://docs.expo.dev/router/basics/core-concepts/): the six rules behind everything
- [Router notation](https://docs.expo.dev/router/basics/notation/): `[]`, `()`, `+` and `_` explained
- [Navigating between pages](https://docs.expo.dev/router/basics/navigation/): `Link`, `router`, params
- [Layouts](https://docs.expo.dev/router/basics/layout/) and [Common navigation patterns](https://docs.expo.dev/router/basics/common-navigation-patterns/)
- [Stack](https://docs.expo.dev/router/advanced/stack/), [Tabs](https://docs.expo.dev/router/advanced/tabs/), [Drawer](https://docs.expo.dev/router/advanced/drawer/), [Modals](https://docs.expo.dev/router/advanced/modals/) and [Nesting navigators](https://docs.expo.dev/router/advanced/nesting-navigators/)
- [Native tabs](https://docs.expo.dev/router/advanced/native-tabs/): the newer platform-native tab bar
- [URL parameters](https://docs.expo.dev/router/reference/url-parameters/) and [Hooks reference](https://docs.expo.dev/router/reference/hooks/)
- [Protected routes](https://docs.expo.dev/router/advanced/protected/) and [Authentication](https://docs.expo.dev/router/advanced/authentication/)
- [Typed routes](https://docs.expo.dev/router/reference/typed-routes/)
- [Linking into your app (deep links)](https://docs.expo.dev/linking/into-your-app/)
- [Redirects](https://docs.expo.dev/router/reference/redirects/) and [Error handling](https://docs.expo.dev/router/error-handling/)
- [API routes](https://docs.expo.dev/router/reference/api-routes/), [Static rendering](https://docs.expo.dev/router/reference/static-rendering/) and [EAS Hosting](https://docs.expo.dev/eas/hosting/introduction/)
- [Migrating from React Navigation](https://docs.expo.dev/router/migrate/from-react-navigation/): side-by-side mapping of old and new code

**Hands-on**

- [Expo's official tutorial](https://docs.expo.dev/tutorial/introduction/): build a small app step by step, using Expo Router
- [Expo Router source code on GitHub](https://github.com/expo/expo/tree/main/packages/expo-router)
- [Expo blog](https://expo.dev/blog): release notes and new router features with each SDK

**React Navigation (to understand what runs underneath)**

- [React Navigation: Getting started](https://reactnavigation.org/docs/getting-started)
- [React Navigation: Static configuration](https://reactnavigation.org/docs/static-configuration)

## Conclusion

Expo Router turns navigation from a configuration problem into a folder problem. Create a file, get a screen. Get a screen, get a URL. Get a URL, get deep links, web support and type safety for free.

Underneath, it is still React Navigation, so nothing you learn is wasted. You just write far less code to get there.

**Your homework:** create a new project with `npx create-expo-app@latest`, run `npm run reset-project`, and build a three-screen app: a Home tab, a Profile tab, and a `products/[id]` details screen opened from Home. If you can do that, you understand the core of Expo Router.

Happy building!

*If this guide helped you, give it a clap and follow for more beginner-friendly React Native and Expo articles.*
