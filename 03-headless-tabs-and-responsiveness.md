
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
  <TabList  className={classNames(
      "py-3 sm:py-6",
      "px-6 sm:px-8",
      "mx-2 sm:mx-0",
      "bg-white",
    )}>
    <TabTrigger name="index" href="/" asChild>
      <TabButton icon="museum">Home</TabButton>
    </TabTrigger>
    <TabTrigger name="exhibits" asChild href="/exhibits">
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

Also be sure to import everything and remove unused imports: 
```diff
+ import { Tabs, TabList, TabSlot, TabTrigger } from "expo-router/ui";
+ import { TabButton } from "@/components/TabButton";
+ import classNames from "classnames";
+ import { View } from "react-native";

- import { Tabs } from "expo-router";
- import customColors from "@/constants/colors";
- import colors from "@/constants/colors";
- import { TabBarIcon } from "@/components/TabBarIcon";
```

🏃**Try it:** Tabs will look a little different, but they definitely should be there. Congrats, the tabs are all yours now to style as you wish.

3. Whoever made this workshop was nice enough top provide a basic `TabButton`, but you can't even tell which one is selected. `TabTrigger` provides an `isFocused` prop to its children, which can be used to adjust styles dynamically. Update the `components/TabButton.tsx` component to add this functionality:

```diff
<Pressable
  ref={ref}
  {...props}
>
  <MaterialIcons
+   color={isFocused ? colors.tint : colors.black}
    name={icon}
    size={24}
  />
  <Text
-    className="text-md"
+    className={"text-md" + (isFocused ? " color-tint" : "")}
  >
    {children}
  </Text>
</Pressable>
```

> [!NOTE]  
> Using variables to construct `className` is totally fine in Nativewind as long as what's being added/removed are entire class names. Just make sure there's a space between the classnames: `text-md color-tint` is valid, but `text-mdcolor-tint` is not.

## Exercise 2: Responsive tabs
We could do a million things to optimize these tabs for mobile and desktop web, but today we have a few modest goals for these tabs when the screen is wider than 640dp:
- Move the tabs from the center to the top right
- Ditch the icons
- Add a website title / logo to occupy the blank space in the top left.

To accomplish these goals, we need to add Nativewind classes with responsive breakpoints (e.g., `sm:`) around the tab buttons (for positioning) and inside the `Button` component (for icon changes and other intra-button styling)

### Outside the tabs
🏃**Try it** after each of the steps by running the app in your web browser and expanding and contracting the window (the "Web Preview" VS Code extension is quite nice for this).

1. First move the tabs to the right and give them a little extra space (and a little shadow, why not?):
```diff
<TabList  className={classNames(
  "py-3 sm:py-6",
  "px-6 sm:px-8",
  "mx-2 sm:mx-0",
+  "sm:justify-end sm:gap-x-4 sm:shadow-sm",
  "bg-white",
)}>
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

```diff
@@ -38,7 +38,18 @@ export default function TabLayout() {
           <TabSlot />
         </View>
         {tabs}
-      </Tabs>
+      </Tabs>
+      <View
+        className={classNames(
+          "absolute left-6 top-5 h-10 w-52",
+        )}
+      >
+        <Image
+          source={require("@/assets/images/logo.svg")}
+          className="w-full h-full"
+        />
+      </View>
     </View>
   );
 }
```

Don't forget to `import { Image } from "expo-image";`

4. Oops! It's still showing up on small screens. Fix that:

```diff
<View
  className={classNames(
+    "hidden sm:inline",
    "absolute left-6 top-5 h-10 w-52",
  )}
>
  <Image
    source={require("@/assets/images/logo.svg")}
    className="w-full h-full"
  />
</View>
```

### Inside the tabs
All the steps here are inside **TabButton.tsx**.

🏃**Try it** after each step just like you've been doing.

5. Let's get rid of the icon for the wide screen format:
```diff
<MaterialIcons
  color={isFocused ? colors.tint : colors.black}
+  className="sm:hidden"
  name={icon}
  size={24}
/>
```

6. Let's make the tint color a line below the tab title instead of the title itself (and make the text larger):
```diff
<Pressable
  ref={ref}
  {...props}
+  className={isFocused ? " sm:border-b-tint sm:border-b-2" : ""}
>
```

```diff
<Text
-  className={" text-sm" + (isFocused ? " color-tint" : "")}
+  className={  "text-sm sm:text-lg" + (isFocused ? " color-tint sm:color-black" : "")}
>
  {children}
</Text>
```

### Exercise 3: Special tab behaviors (OK, just one)
Try going to Exhibits, clicking on an exhibit, and then going to another tab and back again. You'll notice that it still is navigated to the exhibit. We could do some tricky navigation stuff to ensure you end up back on the first tab, or we could let Expo Router do it for us.

1. In **app/(app)/(tabs)/_layout.tsx**, update the exhibits trigger to "reset":
```diff
- <TabTrigger name="exhibits" asChild href="/exhibits">
+ <TabTrigger name="exhibits" asChild href="/exhibits" reset="always">
```

🏃**Try it** Try navigating to a exhibit, leaving the tab, and coming back again. It should reset to the first screen in the route.

Feel free to try some of the other possibilities here. You can trigger the "reset" based on a long press, or selectively reset only on web (where it's definitely a more typical behavior, vs mobile, which is more of a judgement call).

### Exercise 4: Floaty tabs

With just a little bit of extra Nativewind magic, we can turn those regular small screen tabs into fancy floating tabs that exist above your content, and allow enough extra space that you can scroll the content past them.

1. Back in **(tabs)/_layout.tsx**, try these styles out:
```diff
<TabList
  className={classNames(
    "py-3 sm:py-6",
    "px-6 sm:px-8",
    "mx-2 sm:mx-0",
+    "bottom-safe-offset-2 sm:bottom-safe-offset-0", // keep the tabs above safe ares
+    "rounded-full sm:rounded-none", // round the corners
+    "absolute right-0 left-0 sm:relative", // position above content
+    "shadow-sm", // yum, shadows!
    "sm:justify-end sm:gap-x-4 sm:shadow-sm",
    "bg-white",
  )}
>
```

## See the solution
[Solution PR](https://github.com/keith-kurak/expo-router-london-2024-starter/pull/3)

## Next exercise
[Module 04](04-shared-routes.md)

