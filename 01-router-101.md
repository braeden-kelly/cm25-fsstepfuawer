
# Module 01: Router 101!

### Goal
Run the app and start exploring file-based routing by adding a few routes of your own.

### Concepts
- Adding tab routes in Expo Router
- Adding dynamic stack routes
- How layout files can customize the display and behavior of routes

### Tasks
- Get the app running on your local machine, displaying on your phone, simulator, or web browser
- Add the "visit" tab and customize the icon by customizing the layout file
- Add the `works/[workId]` dynamic route so you can navigate to individual works of art
- In a very slight nod to responsive design, customize the `works/[workId]` route to use a transparent modal, centering the artwork in the middle of the screen on larger screens.

### Helpful links
- TBD

# Exercises

## Exercise 0: Get this thing running
For the full experience, make sure you have the app running on both your phone and your browser. However, you can still do the entire workshop if you can only get one of these working.

1. Run `npm install`
2. Run `npx expo start`
3. Scan the QR Code on Expo Go on your phone, or press `i` to open in the iOS simulator.
4. Press `w` to run in your web browser.

Note that the app isn't entirely responsive yet (you're be working on that!). So, it might be easiest to shrink the app down to mobile size by opening Chrome Devtools.

## Exercise 1: Routing and layout basics
We're adding a new screen with info about visiting the museum... but where? Let's try a few places, and learn some Expo Router basics along the way.

_This short exercise is helpful if you're pretty new to Expo Router, but doesn't factor into the final app, so if you're already familiar with stack layouts and route groups, you can skip to Exercise 2._

1. Create a **visit.tsx** file in the **(app)** folder. Copy our [**visit.tsx** starter here](/files/01/visit.tsx) for its contents. `(app)` is a _route group_, representing the "logged-in" experience of the app (this implies a "logged out" experience; we'll get back to that).
2. In your web browser, try navigating to it by using the URL bar, by entering: `http://localhost:8081/visit`. Notice how the route group folder doesn't factor in the URL. These are just for organizing routes and defining their behavior without affecting the URL.
3. Try adding a `Link` to the "visit" page. The very top of the **index** page in **app/(app)/(tabs)/index.tsx** will work:
```tsx
<Link href="/visit">Visit</Link>
```
4. Tap on the link in the browser or the mobile app. It should take you to the "visit" page, and you should be able to go back to the main tabs. This is because the **\_layout.tsx** inside the **(app)** folder groups the routes at this level into a _stack navigator_.
5. Get rid of that `Link` - we're going to put it somewhere else

### Not found routes.
There's a special route at the top of your **app** folder (which is the folder that always contains your Expo Router routes): **not-found.tsx**
This route will render if your app navigates to a route that doesn't exist.
6. Try typing a non-existent route into the browser, e.g., `http://localhost:8081/slartibartfast`. Where does it go?

## Exercise 2: Add the Visit page to the tab navigator.
We've decided it would be best to have information for visitors as a separate tab.

1. Create a **visit.tsx** file in the **(app)/(tabs)** folder. Copy our [**visit.tsx** starter here](/files/01/visit.tsx) for its contents. (or, if you already created **visit.tsx** in Exercise 1, just drag it into **(tabs)**)
2. Your app should live-reload to show a new tab. Check out **(app)/(tabs)/_layout.tsx**. This layout defines a tab navigator, so it's saying any routes at this level should be arranged as tabs.
3. The Visit tab looks pretty basic right now. Not the right icon or text. We can information in **_layout.tsx** to change the behavior and appearance of the tab. Add this between the exhibits and profile tab:
```tsx
<Tabs.Screen
  name="visit"
  options={{
    title: "Visit",
    tabBarIcon: ({ color }) => (
      <TabBarIcon type="MaterialIcons" name="map" color={color} />
    ),
  }}
/>
```
🏃**Try it.** Now the tab should look pretty good.

### Nested routes
Notice that not everything in that tabs route is just a single file. There's the **exhibits** folder, with multiple routes. This is a stack nested inside a tab.
Let's experiment with some different scenarios here. **Refresh the browser / shake and refresh your app before each one to reset navigation history**:
a. Exhibits tab -> click on an exhibit -> go back (pretty normal stack-in-tabs)
b. Exhibits tab -> Home tab -> click on an exhibit name above an artwork -> go back (normal-ish, going back to the index of Exhibits instead of Home is interesting)
c. Home tab -> click on an exhibit -> go back (oh wait, you can't, that's bad!)
d. (browser-only) Home tab -> click on an exhibit -> reload page (can't go anywhere! real bad)

4. Fix c) and d) by adding the following to **(app)/exhibits/_layout.tsx**:
```tsx
export const unstable_settings = {
  // Ensure any route can link back to `/`
  initialRouteName: "index",
};
```

This says that, when inside the **exhibits** route, the index should be the first route. The ensures that, even if you go directly to a exhibit, the exhibits list will be underneath it.

<!--
Errata: 
- c) is not actually fixed, not sure why .. actually, it's due to lazy=false not existing on headless tabs, but withAnchor might fix this.
    Anyway, this will work at this stage of the workshop as long as we set lazy: true ;-)
    I think I'll leave that in there for them, we don't want to deal with this complication
    But something I should ask Mark about
- Also, there's param junk when going back from d). Maybe just an alpha thing?
- Facing this issue right now: https://github.com/expo/expo/issues/31747
 -->


## Exercise 3: Navigating to arbitrary things with dynamic routes
Dynamic routes let you put a variable into your URL, that you can then query on the destination screen. They are defined with square brackets, e.g., `[id]`.

There's already a dynamic route in the app: `/exhibits/[exhibitName]`. When you navigate to that route, the URL will contain the actual exhibit, e.g., `/exhibits/Textiles`

Notice how the artworks don't go anywhere yet when you click on them. Let's fix that.

1. Create the file **app/(app)/works/[workId].tsx** (add the extra folder, of course). Use the [**[workId].tsx** starter here](/files/01/[workId].tsx).
2. In **Artwork.tsx** (shared by both screens), wrap the artwork in a `Link`, navigating to `works/[workId]`:
```diff
renderItem={({ item }) => (
+  <Link asChild href={`/works/${artwork.id}`}>
    <Pressable>
    // ...
+    </Pressable>
  </Link>
```

🏃**Try it.** You should be able to navigate to artwork now. Try a few.

Oh wait, it's going to the same work every time. That's because we didn't read the route parameter inside the **[workId].tsx** screen itself.

3. Add this at the top of **[workId].tsx**, replacing the hardcoded `workId`:

```tsx
 const { workId } = useLocalSearchParams<{
    workId: string;
  }>();
```

(import `useLocalSearchParams` from `expo-router`)

🏃**Try it.** You should now be able to navigate to the correct artwork from both the Home and Exhibits tabs.

### About the elephant in the header...
By now, you're probably quite annoyed by that header with route garbage at the top of your screen. You could set `title` in screen options to at least make it something nice, but you'd still have double headers! We've clearly implemented our own header in **[workId].tsx**, so let's get rid of React Navigation's.

4. In **app/(app)/_layout.tsx**, add a reference to `works/[workId]`, so you can hide the header with screen options:

```tsx
<Stack.Screen
  name="works/[id]/index"
  options={{
    headerShown: false,
  }}
/>
```

## Exercise 4: A little bit of responsiveness
Head over to your browser, and make the window wide. Now open an artwork. Real "blown up mobile app" vibes, right? Let's use some navigator props and creative styling to make this a centered modal on web, floating above the underlying content.

1. In **app/(app)/layout.tsx**, add this code just inside the options for this screen:
```diff
<Stack.Screen
-  options={{
-    headerShown: false,
-  }}
+ options={{
+  ...Platform.select({
+    web: {
+      presentation: "transparentModal",
+     animation: "fade",
+    },
+    default: {
+      presentation: "modal",
+   },
+  }),
+ headerShown: false,
+ }}
/>
```

(don't forget to import `Platform` from `react-native`)

<details>
  <summary>Expand to just get the whole function for easy copying</summary>

```tsx
options={{
  ...Platform.select({
    web: {
      presentation: "transparentModal",
      animation: "fade",
    },
    default: {
      presentation: "modal",
    },
  }),
  headerShown: false,
}}
```

</details>

This will make this screen present like a transparent modal, but it will not look very obvious until we shrink the size of the content, so the transparent area will show up underneath it.

Let's use Nativewind's _responsive breakpoints_ to adjust the window size. If you're not used to Nativewind or Tailwind, that's OK. We're just going to do a very little bit to get a taste of it.

Nativewind works by letting you use a standard set of descriptive CSS classes to style components. You put these in the `className` prop instead of `style`. What's neat about it is that your styling code looks completely the same across React Web and React Native.

_Breakpoints_ let you put something in front of one of these classes that says it only applies in certain scenarios. So, `sm:bg-black` means, "only set the background to black when the width is greater than the "small" size breakpoint (>640 device pixels)". Nativewind syntax defaults to "mobile" size. You only use breakpoints when targeting larger screens.

3. Let's update the top-level `View` in **[workId].tsx** so it is centered and only takes up some of the screen when the screen is wider, so we can see our modal effect in action.

```diff
- <View className="flex-1">
+ <View className="sm:my-20 sm:w-3/4 sm:self-center">
```

🏃**Try it.** Open an artwork and change the screen size. It should show the modal when the screen is wider, but look like a pretty typical mobile full screen view otherwise.

4. However, it doesn't look like a very compelling modal because the background behind it doesn't change. Let's add some shade by wrapping everything in **[workId].tsx** in another view that adds said shade:
```tsx
<View
  className={classNames(
    "bg-white sm:bg-black",
    "sm:bg-opacity-20",
    "flex-1 justify-center",
    Platform.OS === "android" && "pt-safe",
  )}
>
  <!-- everything else -->
</View>
```

> [!NOTE]  
> Hello, new friend! `classNames` can be used to organize all these bits of Nativewind! Import it as such: `import classNames from "classnames";`

### How would I do this without Nativewind?
There's some fancy media query hooks for React Native that give you programatic access to breakpoints, which you can then use inline in `style`. It's also pretty straightforward to roll your own:

```ts
import { useWindowDimensions } from 'react-native';

export function useMediaQuery() {
  const { width } = useWindowDimensions();
  return { 
    isSm: width >= 640,
    isMd: width >= 768, 
    isLg: width >= 1024, 
    isXl: width >= 1280 
  };
}
```

## Bonus
- Hide the tab bar on mobile when navigating to an exhibit.

## See the solution
Switch to branch: `01-router-101-solution`

## Next exercise
[Module 02](02-api-routes-and-auth.md)
