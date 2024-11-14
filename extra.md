# Extra Stuff
Just a fun collection of things learned along the way...

## Invoking a deep link on Expo Go
You could use your web browser with the `exp://` extension, or use this from your terminal (when connected to a simulator or over ADB):

```
npx uri-scheme exp://[your computer's ip address]:8081/--/works/137259
```
## Module 01

### Fixing the red squiggles in the tab navigator
```tsx
<Tabs
  screenOptions={{
    sceneStyle: { backgroundColor: colors.white },
    headerShown: false,
  }}
>
```

### Modal looks terrible on iPad
Oops, we didn't set the transparency right when the screen is wider but it's on mobile

## Other

### Incrementality adopt expo
https://expo.dev/blog/how-to-incrementally-adopt-expo

### Native Bottom tabs
Doing headless for web and [native bottom tabs](https://okwasniewski.github.io/react-native-bottom-tabs/docs/getting-started/quick-start.html) would be the perfect pairing (development build required)
