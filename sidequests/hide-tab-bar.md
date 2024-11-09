# Sidequest: Hide the tab bar when deep in a tab's stack when on mobile

### Goal
When a user is on mobile, let's save some screen real estate and hide the tab bar when the user is looking at an exhibit, forcing them to back out before navigating to another tab.

There's two versions of this:
- React Navigation tabs (available after Module 01)
- Headless tabs (available after Module 03)

They share some common bits, which we'll cover first.

## Common steps: setting the conditions for hiding
We can read segments of the URL for determining if we're in a nested route.

All of these steps are in **app/(app)/(tabs)/_layout.tsx**.

1. Grab the segments:
```tsx
import { useSegments } from "expo-router"

// ...

const segments = useSegments();
```

2. Find the segment in the URL of the `exhibits` route:
```tsx
import { findIndex } from "lodash"

// ...

const tabRouteIndex = findIndex(
  segments,
  // @ts-ignore
  (s) => s === "exhibits"
);
```

The index is used to determine if the route is at the end of the list of segments (which would be mean we're at the index for `exhibits` and the tabs should be visible)

3. Set a condition for hiding the tabs:
```tsx
  const hideTabs =
    tabRouteIndex !== -1 && // the route exists
    tabRouteIndex !== segments.length - 1 && // the route isn't at the end
    Platform.OS !== "web"; // we're on mobile
```

## Conditionally render tabs: headless edition (after Module 03)

One neat thing about headless tabs is that the "triggers" (what defines the screen as available routes to be invoked by tab buttons), can be listed separately from the actual tab buttons themselves. Thus, we need to list the `<TabTrigger>` components inside a `<TabList>` without UI, and then list the tab buttons wrapped in `<TabTrigger>` separately to actually render the buttons that invoke the triggers.

> [!NOTE]
> If you have a route in your tabs folder but it doesn't have a trigger, you won't be able to navigate to it! Could be useful for authentication states...

1. Follow the [split triggers guide](shared/split-triggers.md).

2. Update the return statement to render the triggers all the time, and the visuals when you don't want to hide the tabs:

```diff
<Tabs className="flex-1 sm:flex-col-reverse">
  <View className="flex-1">
    <TabSlot />
  </View>
-  {tabs}
+  {tabList}
+  {!hideTabs && tabVisual}
</Tabs>
```

## Conditionally render tabs: React Navigation edition (after Module 01)

This one's a little trickier. You may need to play with the styles to get it to look OK.

At the most basic level, it's like this:

```tsx
<Tabs
  screenOptions={{
    tabBarStyle: [
      {
        display: hideTabs ? "none" : "flex",
      },
    ],
  }}
>
// ...
</Tabs>
```

In practice, I've also had to absoultely position the bar to get it to not do weird jumping and clipping:
```tsx
position: "absolute",
height: bottom + 70,
```

And I've had to set backgrounds to eliminate flashing (more applicable on dark backgrounds):
```tsx
sceneContainerStyle={{ backgroundColor: /* ... */ }}
```

Start with the `display` prop and see what you need to do to make it work on all platforms.

## Other stuff you could do
- the button bar looks a little busy here on mobile web, so you could probably hide it from there, as well.