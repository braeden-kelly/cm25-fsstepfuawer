# Sidequest: Show a hamburger menu and drawer on mobile web instead of tabs

### Goal
Bottom tabs are perhaps not quite what we'd want on mobile web. Let's try a hamburger menu that opens a drawer. However, the drawer will really just contain tab buttons that otherwise work the same as our other tab layouts.

React Navigation 7 now includes a [drawer layout](https://reactnavigation.org/docs/drawer-layout/) package that decouples the drawer from the navigator, making a great pairing with headless tabs.

## Decoupling tabs and tabs

One neat thing about headless tabs is that the "triggers" (what defines the screen as available routes to be invoked by tab buttons), can be listed separately from the actual tab buttons themselves. Thus, we need to list the `<TabTrigger>` components inside a `<TabList>` without UI, and then list the tab buttons wrapped in `<TabTrigger>` separately to actually render the buttons that invoke the triggers.

We will need this, as most of the tab layout will go inside the `Drawer` as children, but the buttons will go in `renderDrawerContent`.

> [!NOTE]
> If you have a route in your tabs folder but it doesn't have a trigger, you won't be able to navigate to it! Could be useful for authentication states...

All steps below are within **(tabs)/_layout.tsx**.

1. Follow the [split triggers guide](shared/split-triggers.md)

2. Add some imports (run `npm install react-native-drawer-layout`):

```tsx
import { useMediaQuery } from "@/constants/useMediaQuery";
import { Drawer } from "react-native-drawer-layout";
import { FontAwesome } from "@expo/vector-icons";
```

3. Add a state variable for opening/closing the drawer and grab `isModule` to detect small screens:

```tsx
const { isMobile } = useMediaQuery();

const [open, setOpen] = React.useState(false);
```

4. Before the return statement already there, let's add a separate return for our mobile web UI, including the drawer:

```tsx
if (isMobile && Platform.OS === "web") {
  return (
    <Drawer
      open={open}
      onOpen={() => setOpen(true)}
      onClose={() => setOpen(false)}
      renderDrawerContent={() => {
        return (
          
        );
      }}
    >
      <View className="flex-1">
        <View className="flex-row justify-between items-center py-3 px-6">
          <FontAwesome name="bars" size={24} onPress={() => setOpen(true)} />
          <View className={classNames("h-10 w-52")}>
            <Image
              source={require("@/assets/images/logo.svg")}
              className="w-full h-full"
            />
          </View>
        </View>
        <Tabs className="flex-1">
          <View className="flex-1">
            <TabSlot />
          </View>
          {tabList}
        </Tabs>
      </View>
    </Drawer>
  );
}
```

At this point, you should be able to open the drawer (but nothing's there).

5. You could try adding the `tabVisual` variable to `renderDrawerContent`. Spoiler alert: you get an error because the tab triggers aren't inside of `Tabs`. How do we solve this chicken/egg problem?

6. Use plain `Link` components (import from `expo-router`):

```tsx
renderDrawerContent={() => {
  return (
    <View>
      <Link href="./" asChild>
        <TabButton icon="museum">Home</TabButton>
      </Link>
      <Link href="./exhibits" asChild>
        <TabButton icon="palette">Exhibits</TabButton>
      </Link>
      <Link href="./visit" asChild>
        <TabButton icon="map">Visit</TabButton>
      </Link>
      <Link href="./profile" asChild>
        <TabButton icon="person">Profile</TabButton>
      </Link>
    </View>
  );
}}
```

At this point, you should be able to open new tabs. Sorry the tabs look rough, though! The drawer doesn't close, either, when you navigate.

7. Setup an effect to make the drawer change when the route changes. Import `usePathname` from `expo-router`:

```tsx
const pathname = usePathname();

  useEffect(() => {
    if (isMobile && Platform.OS === "web") {
      setOpen(false);
    }
  }, [pathname, isMobile]);
```

8. If you want to make the drawer tabs look nice, that's up to you!