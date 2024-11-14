# Extra Stuff
Just a fun collection of things learned along the way...

## Invoking a deep link on Expo Go
You could use your web browser with the `exp://` extension, or use this from your terminal (when connected to a simulator or over ADB):

```
exp://[your computer's ip address]:8081/--/works/137259
```
## Fixing the red squiggles in the tab navigator (module 01):

```tsx
<Tabs
  screenOptions={{
    sceneStyle: { backgroundColor: colors.white },
    headerShown: false,
  }}
>
```
