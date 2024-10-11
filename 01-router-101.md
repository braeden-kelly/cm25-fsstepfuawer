
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
- Add the `works/[id]` dynamic route so you can navigate to individual works of art
- In a very slight nod to responsive design, customize the `works/[id]` route to use a transparent modal, centering the artwork in the middle of the screen on larger screens.

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

1. There's a **visit.tsx** file in the **new-screens** folder. Drag that file to the **(app)** folder. This is a _route group_, representing the "logged-in" experience of the app (this implies a "logged out" experience; we'll get back to that).
2. In your web browser, try navigating to it by using the URL bar, by entering: `http://localhost:8081/visit`. Notice how the route group folder doesn't factor in the URL. These are just for organizing routes and defining their behavior without affecting the URL.
3. Try adding a `Link` to the "visit" page. The very top of the **index** page will work:
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

1. Move **visit.tsx** (either from **new-screens** or from whereever you left it in Exercise 1) to **(app)/(tabs)**.
2. Your app should live-reload to show a new tab. Check out **(app)/(tabs)/_layout.tsx**. This layout defines a tab navigator, so it's saying any routes at this level should be arranged as tabs.
3. The Visit tab looks pretty basic right now. Not the right icon or text. We can information in **_layout.tsx** to change the behavior and appearance of the tab. Add this between the departments and profile tab:
```tsx
<RNTabs.Screen
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
Notice that not everything in that tabs route is just a single file. There's the **departments** folder, with multiple routes. This is a stack nested inside a tab.
<!-- the API calls these departments, but exhibits sounds nicer, hence the difference, maybe I'll fix this -->
Let's experiment with some different scenarios here. **Refresh the browser / shake and refresh your app before each one to reset navigation history**:
a. Exhibits tab -> click on a department -> go back (pretty normal stack-in-tabs)
b. Exhibits tab -> Home tab -> click on a department name above an artwork -> go back (normal-ish, going back to the index of Exhibits instead of Home is interesting)
c. Home tab -> click on a department -> go back (oh wait, you can't, that's bad!)
d. (browser-only) Home tab -> click on a department -> reload page (can't go anywhere! real bad)

4. Fix c) and d) by adding the following to **(app)/departments/_layout.tsx**:
```tsx
export const unstable_settings = {
  // Ensure any route can link back to `/`
  initialRouteName: "index",
};
```

This says that, when inside the **departments** route, the index should be the first route. The ensures that, even if you go directly to a department, the departments list will be underneath it.

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

There's already a dynamic route in the app: `/departments/[department]`. When you navigate to that route, the URL will contain the actual department, e.g., `/departments/Textiles`

Notice how the artworks on a department page don't go anywhere yet. Let's fix that.

1. Copy **works/[workId].tsx** out of **new-screens** and into **(app)/works/**
2. In **[department].tsx**, wrap the artwork in a `Link`, navigating to `works/[workId]`:
```diff
renderItem={({ item }) => (
+  <Link asChild href={`/works/${item.id}/` as Href}>
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

## Exercise 4: A little bit of responsiveness
Head over to your browser, and make the window wide. Now open an artwork. Real "blown up mobile app" vibes, right? Let's use some navigator props and creative styling to make this a centered modal on web, floating above the underlying content.

1. In **works/[workId].tsx**, add this code just inside the top-level `View`:
```diff
<Stack.Screen
  options={{
    title: work?.title || "Loading...",
+    presentation: "transparentModal",
+    animation: "fade",
    headerShown: false,
  }}
/>
```

This will make this screen present like a transparent modal, but it will not look very obvious until we shrink the size of the content, so the transparent area will show up underneath it.

2. Add the file **utils/useMediaQuery.ts** and put this code inside:
```ts
import { useWindowDimensions } from 'react-native';

export function useMediaQuery() {
  const { width, height } = useWindowDimensions();
  return { isLandscape: width > height };
}
```

You could of course use more sophisticated breakpoints, but one of the simplest things we can do to be responsive is to account for a landscape orientation, so we'll start there.

Back in **[workId].tsx**, let's wrap the screen in some views that account for landscape orientation and add some vertical / horizontal space with some shade to give us the desired modal effect.

4. Reference the hook in **[workId].tsx**:

```tsx
const { isLandscape } = useMediaQuery();
```

5. Wrap the entire contents of the screen in these views:
```tsx
 // TBD: re-writing this in not-nativewind
```

🏃**Try it.** Open an artwork and change the screen size. It should show the modal when the screen is wider, but look like a pretty typical mobile full screen view otherwise.

## Bonus
- Hide the tab bar on mobile when navigating to an exhibit.

## See the solution
Switch to branch: `01-router-101-solution`

## Next exercise
[Exercise 2](02-dynamic-routes.md)
