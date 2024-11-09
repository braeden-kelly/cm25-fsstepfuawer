# Split Triggers
_This howto is used by a few sidequests. It demonstates how to use one set of `TabTrigger`'s to define the available routes, and another to define the tab buttons._

All changes are in **(app)/(tabs)/_layout.tsx**.

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
      <TabTrigger name="exhibits" asChild>
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