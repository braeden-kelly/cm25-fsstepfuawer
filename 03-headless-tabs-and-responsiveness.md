
# Module 03: Headless tabs and responsive design

### Goal
Fully-customize the tab bar so its much easier to move around and attach custom behavior to depending whether you're in the mobile app or desktop web.

### Concepts
- Using headless tab navigators to fully-customize tab navigation
- Using Nativewind selectors to reposition and reformat the tabs based on the width of the screen
- Attach additional behaviors to the tab bar

### Tasks
- Replace the React Navigation tab bar with your own custom tab bar using the headless components in `expo-router/ui`
- Move the tab bar to the top of the page when on desktop web and make it look more like simple text header links than a mobile tab bar
- Automatically go back to the top of the stack when you navigate to Exhibits.

### Helpful links

# Exercises

## Exercise 1: Intro to headless tabs
Until now, we've been using the `Tabs` component from `expo-router`, which more-or-less a proxy for React Navigation's bottom tabs. These tabs look fine on a mobile app, but don't really scale to desktop web. There's tons of customization points, but even if you touch almost all of them, there's still a lot of built-in styles that you'll need to account for in order to customize the built-in tabs 100% to your specifications.

What if we could just... start over? Rebuild tabs from scratch?

That's what headless tabs are. They give you unstyled building blocks for a whole tab navigator, and you can arrange them however you want. They're imported from `expo-router/ui`. Let's the meet the roster of components:
- `Tabs`: The outer wrapper. Defines the boundaries of the navigator
- `TabSlot`: Where the currently-selected tab's contents are rendered
- `TabList`: The `TabTrigger` components inside this define which tabs are in the navigator
- `TabTrigger`: Defines the route and a "name" for each individual tab. Are used to wrap actual tab button UI component. Multiple triggers can go to the same route if they share the same name (only the triggers inside `TabList` define an `href`, the others just wrap buttons going to that `href`).

### Dive right in
Let's go big and swap out the entire tab navigator for a new one with headless tabs.

1. In **app/(app)/(tabs)/_layout.tsx**, remove all the contents of `TabLayout` and replace them with a headless tabs layout:
```tsx
return (
  <View className="flex-1">
    <Tabs className="flex-1">
      <View className="flex-1">
        <TabSlot />
      </View>
      {tabs}
    </Tabs>
  </View>
);
```

2. We split out the `tabs` into a separate variable to make it a little easier to read. Add the `tabs` variable:
```tsx
const tabs = (
  <TabList className="py-2 px-8">
    <TabTrigger name="index" href="/" asChild>
      <TabButton icon="museum">Home</TabButton>
    </TabTrigger>
    <TabTrigger name="departments" asChild href="/departments">
      <TabButton icon="palette">Exhibits</TabButton>
    </TabTrigger>
    <TabTrigger name="visit" asChild href="/visit">
      <TabButton icon="map">Visit</TabButton>
    </TabTrigger>
    <TabTrigger name="profile" asChild href="/profile">
      <TabButton icon="person">Profile</TabButton>
    </TabTrigger>
  </TabList>
);
```

Also be sure to import everything: 
- `import { Tabs, TabList, TabSlot, TabTrigger } from "expo-router/build/ui";`
- `import { TabButton } from "@/components/TabButton";`

🏃**Try it:** Tabs will look a little different, but they definitely should be there. Congrats, the tabs are all yours now to style as you wish.

3. Whoever made this workshop was nice enough top provide a basic `TabButton`, but you can't even tell which one is selected. `TabTrigger` provides an `isFocused` prop to its children, which can be used to adjust styles dynamically:

```diff
  <MaterialIcons
+    className={isFocused ? " color-tint" : ""}
    name={icon}
    size={24}
  />
  <Text
-    className="text-md"
+    className={" text-md" + (isFocused ? " sm:text-white color-tint" : "")}
  >
    {children}
  </Text>
</Pressable>
```

> [!NOTE]  
> Using variables to construct `className` is totally fine in Nativewind as long as what's being added/removed are entire class names.

## Exercise 2: Responsive tabs
We could do a million things to optimize these tabs for mobile and desktop web, but today we have a few modest goals for these tabs when the screen is wider than 640dp:
- Move the tabs from the center to the top right
- Ditch the icons
- Add a website title / logo to occupy the blank space in the top left.

To accomplish these goals, we need to add Nativewind classes with responsive breakpoints (e.g., `sm:`) around the tab buttons (for positioning) and inside the `Button` component (for icon changes and other intra-button styling)

### Outside the tabs
🏃**Try it** after each of the steps by running the app in your web browser and expanding and contracting the window (the "Web Preview" VS Code extension is quite nice for this).

1. First move the tabs to the right and give them a little extra space:
```diff
+ <TabList className="py-2 px-8 sm:justify-end sm:gap-x-4">
- <TabList className="py-2 px-8">
```

2. Invert the tabs and the content, moving the tabs to the top:
```diff
  <View className="flex-1">
-    <Tabs className="flex-1">
+    <Tabs className="flex-1 sm:flex-col-reverse">
      <View className="flex-1">
        <TabSlot />
      </View>
      {tabs}
    </Tabs>
  </View>
```

3. Let's add that nice title we were talking about. Put this in the return statement... somewhere where you would typically put something absolutely-positioned that you want on-top of everything else:

```tsx
<View className="absolute left-2 top-0.5 flex-row items-center">
  <MaterialIcons name="museum" size={28} className="color-tint top-1" />
  <Text className="text-xl color-tint ml-2">Cleveland Museum of Art</Text>
</View>
```

4. Oops! It's still showing up on small screens. Fix that:

```diff
- <View className="absolute left-2 top-0.5 flex-row items-center">
+ <View className="hidden sm:inline absolute left-2 top-0.5 flex-row items-center">
```

### Inside the tabs
All the steps here are inside **TabButton.tsx**.

🏃**Try it** after each step just like you've been doing.

5. Let's get rid of the icon:
```diff
<MaterialIcons
-  className={isFocused ? " color-tint" : ""}
+  className={"sm:hidden" + (isFocused ? " color-tint" : "")}
  name={icon}
  size={24}
/>
```

6. Let's change the background entirely when the tab is selected:
```diff
<Pressable
  ref={ref}
  {...props}
-  className={"justify-between items-center gap-y-1 px-2 flex-col"}
+  className={"justify-between items-center gap-y-1 px-2 flex-col" + (isFocused ? " sm:bg-shade-3" : "")}
>
```

7. The blue tint seems to be a bit much. Remove that and make the text a little larger:

```diff
<Text
-  className={"text-md" + (isFocused ? " color-tint" : "")}
+  className={"text-md sm:text-lg" + (isFocused ? " sm:text-white color-tint" : "")}
>
  {children}
</Text>
```

### Exercise 3: Special tab behaviors (OK, just one)
Try going to Exhibits, clicking on a department, and then going to another tab and back again. You'll notice that it still is navigated to the department. We could do some tricky navigation stuff to ensure you end up back on the first tab, or we could let Expo Router do it for us.

1. In **app/(app)/(tabs)/_layout.tsx**, update the departments trigger to "reset":
```diff
- <TabTrigger name="departments" asChild href="/departments">
+ <TabTrigger name="departments" asChild href="/departments" reset="always">
```

🏃**Try it** Try navigating to a department, leaving the tab, and coming back again. It should reset to the first screen in the route.

Feel free to try some of the other possibilities here. You can trigger the "reset" based on a long press, or selectively reset only on web (where it's definitely a more typical behavior, vs mobile, which is more of a judgement call).

### Exercise 4: Floaty tabs

TBD!

## Bonus
- Everything works great if you start at the main route and open a work of art, but what if you copy the link to a work of art and share it with someone? (copy/paste it into another tab in your browser). You're stuck! Fix this with one line of code. (HINT: think `initialRouteName` from Module 01).
- Tabs are perfect for mobile, top tabs are great for desktop web, but what about mobile web? Drawers seem to be a little more common there. Let's turn the tabs into drawer items when the screen width is less than `sm` and the platform is web.
  - You could do this with the Expo Router drawer tutorial, which uses a full drawer navigator, but isn't a drawer really just differently-arranged tabs? Try it instead with something like the [drawer layout](https://reactnavigation.org/docs/drawer-layout/), instead putting the tab triggers inside the drawer content.

## See the solution
Switch to branch: `03-headless-tabs-solution`

## Next exercise
[Module 04](04-shared-routes.md)

