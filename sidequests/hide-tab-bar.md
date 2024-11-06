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

1. Setup two separate variables, `tabVisual` (where we'll define buttons) and `tabList` (where we'll define triggers)

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

3. The `tabList` will have just the triggers, no buttons, but the triggers will define the `href` to show that they're the source of truth for what route a trigger navigates to:

```tsx
const tabList = (
  <TabList>
    <TabTrigger name="index" href="/" />
    <TabTrigger name="exhibits" href="/exhibits" reset="always" />
    <TabTrigger name="visit" href="/visit" />
    <TabTrigger name="profile" href="/profile" />
  </TabList>
);
```

4. Meanwhile, `tabVisual` will be a list of triggers with buttons inside. No `href` will be defined, as they will use the `href` from the `tabList` trigger based on the `name`. Like this:

```tsx
<TabTrigger name="profile" asChild>
  <TabButton icon="person">Profile</TabButton>
</TabTrigger>
```

Do this for all the buttons, enclosing them in a `View`. There will be an extra style you'll need to add to account for a default style that was inside `TabList`.

<summary>If you need help, here's the solution for what's inside of tabVisual</summary>
<details>

```tsx
const tabVisual = (
    <View
      className={classNames(
        "flex-row justify-between",
        "py-3 sm:py-6",
        "px-6 sm:px-8",
        "mx-2 sm:mx-0",
        "sm:justify-end sm:gap-x-4 sm:shadow-sm",
        "bg-white",
        "bottom-safe-offset-2 sm:bottom-safe-offset-0", // keep the tabs above safe ares
        "rounded-full sm:rounded-none", // round the corners
        "absolute right-0 left-0 sm:relative", // position above content
        "shadow-sm sm:shadow-none" // yum, shadows!
      )}
    >
      <TabTrigger name="index" asChild>
        <TabButton icon="museum">Home</TabButton>
      </TabTrigger>
      <TabTrigger name="exhibits" asChild reset="always">
        <TabButton icon="palette">Exhibits</TabButton>
      </TabTrigger>
      <TabTrigger name="visit" asChild>
        <TabButton icon="map">Visit</TabButton>
      </TabTrigger>
      <TabTrigger name="profile" asChild>
        <TabButton icon="person">Profile</TabButton>
      </TabTrigger>
    </View>
  );
```

</details>
</summary>

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