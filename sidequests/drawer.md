# Sidequest: Show a hamburger menu and drawer on mobile web instead of tabs

### Goal
Bottom tabs are perhaps not quite what we'd want on mobile web. Let's try a hamburger menu that opens a drawer. However, the drawer will really just contain tab buttons that otherwise work the same as our other tab layouts.

React Navigation 7 now includes a [drawer layout](https://reactnavigation.org/docs/drawer-layout/) package that decouples the drawer from the navigator, making a great pairing with headless tabs.

## Decoupling tabs and tabs

One neat thing about headless tabs is that the "triggers" (what defines the screen as available routes to be invoked by tab buttons), can be listed separately from the actual tab buttons themselves. Thus, we need to list the `<TabTrigger>` components inside a `<TabList>` without UI, and then list the tab buttons wrapped in `<TabTrigger>` separately to actually render the buttons that invoke the triggers.

We will need this, as most of the tab layout will go inside the `Drawer` as children, but the buttons will go in `renderDrawerContent`.

> [!NOTE]
> If you have a route in your tabs folder but it doesn't have a trigger, you won't be able to navigate to it! Could be useful for authentication states...

1. Setup two separate variables, `tabVisual` (where we'll define buttons) and `tabList` (where we'll define triggers)

2. Update the return statement to render both variables:

```diff
<Tabs className="flex-1 sm:flex-col-reverse">
  <View className="flex-1">
    <TabSlot />
  </View>
-  {tabs}
+  {tabList}
+  {tabVisual}
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

## Bless this mess - a hasty refactor
The only reason we did that refactor above 