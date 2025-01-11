
# Module 04: Shared routes

### Goal
Use groups and group precedence to make the same route go to two different places, depending on how arrive there.

We'll enable navigating from the Home tab to an exhibit work without breaking navigation by sharing the exhibits stack screen between the Home and Exhibits tab. 

Then, we'll enable sharing of links to works of art to visitors who are not logged in by creating an alternate entry point for them outside of the logged-in protected routes that uses the same URL.

### Concepts
- Create distinct routes that share the same outward-facing URL, but use `(group)` syntax to differentiate their position in the heirarchy. 
  - Take advantage of alphabetic precedence to ensure your "dominant" route is used when you want it to be.
- Use groups to create links that behave differently depending on whether you're navigating to them from another route or you're using them as your initial entry point.
- Share the same screen with two different sibling routes with array (e.g., `(route1, route2)`) syntax

### Tasks
- Turn `/exhibits/[exhibit]` into a shared route that is a sibling of both the Home and Exhibits tabs, so it pushes onto either while staying within the same tab. However, if you share a link to an exhibit, it will always open the Exhibits tab.
- Create an alternate entry point into `works/[workId]` that lets anyone look at a work of art shared by a user, even if they're not logged in (Instagram-style)

# Exercises

## Exercise 1: Two stacks, one route: what hath the Home tab wrought?
Let's do some tiny, seemingly-inconsequential thing...that will set off a ripple effect throughout our code, snowballing into a pretty-involved refactor.

1. In the Home tab, wouldn't it be nifty if you could click on the exhibit name and it would take you to the exhibit? Let's do that. In **(app)/(tabs)/index.tsx**, you'll see that each square is implemented by an `Artwork` component. Head into **Artwork.tsx** and wrap the exhibit name in a `Link` and `Pressable`:

```diff
+<Link asChild href={`/exhibits/${exhibitName}`}>
+  <Pressable>
    <View className="flex-row items-center gap-x-2">
      <Text className="text-l uppercase font-bold tracking-widest">
        {exhibitName}
      </Text>
      <View className="flex-1 h-0.5 bg-shade-2" />
    </View>
+  </Pressable>
+ </Link>
```

🏃**Try it** Click on the link. You can go to exhibits, yay!

I don't know if I trust this thing, though. Let's put it through its paces. These scenarios work best in a browser, though you could test them in the mobile app using deep links. **Refresh the browser before each one to reset navigation history**:
- **(A)** Start on Home tab (don't go anywhere else) -> click on an exhibit -> go back
- **(B)** Go to Exhibits tab -> go back to Home tab -> click on an exhibit -> go back

These are both a little weird, right? 

For A), there's no back button! For B), there is a back button, but it takes you back to the Exhibits tab. But that's not "back" for where you were coming from.

A) is happening because you are navigating directly to the non-initial screen in the exhibits stack, and the initial screen never gets loaded prior to that. B) works around that to some extent by loading the initial screen by rendering the tab (which only happens when you navigate to it), but now you're deep in a stack, so back takes you out to the top of the exhibits stack instead of to the Home tab.

Let's fix this! Individual exhibits should always be children of the `exhibits` route, but back should take you back to where you were regardless of where you come from.

### Creating some space: tabs inside of groups
This is going to affect the `(tabs)/index` and `(tabs)/exhibits` routes, so lets add an extra layer to each of those so we have some space to define a shared stack layout and shared routes between the two. Since the "layer" will be route groups, they will not actually affect the URL's.

1. Create an **app/(app)/(tabs)/(home)** folder and put **(tabs)/index.tsx** inside it.
2. Create an **app(app)/(tabs)/(exhibits)** folder and put **(tabs)/exhibits** inside it (including everything inside that folder).
3. Update **app(app)/(tabs)/_layout.tsx** to account for the new home for the index route:
```diff
- <TabTrigger name="index" href="/" asChild>
+ <TabTrigger name="index" href="/(home)" asChild>
```

🏃**Try it** Move around between Home and exhibits. Hopefully a) it works, and b) it's not any worse than before.

4. We want to share the `exhibits/[exhibitName]` route (and nothing else!) between the two tabs. So, let's do that. Create a new folder, **(tabs)/(home,exhibits)**, and create an **exhibits** folder under that. Move **[exhibitName].tsx** to **(tabs)/(home,exhibits)/exhibits**.

🏃**Try it** Navigate to an exhibit from both Home and the Exhibits tabs. Notice only one of these works...

5. The navigation from the Home tab doesn't work because we haven't told Expo Router that `(home)` is now a stack. Create the **(tabs)/(home)/_layout.tsx** file with the following contents:

```tsx
import React from "react";
import { Stack } from "expo-router";

export default function StackLayout() {
  return <Stack />;
}
```

🏃**Try it** OK, I know it looks wrong, but does it work right? Going from Home to an exhibit and back again should take you back home. This is a big milestone!

6. The reason for those extra headers is quite simple. In the Home stack, we haven't hidden the headers. Add a reference to the index screen in **(tabs)/(home)/_layout.tsx**, hiding the header:

```diff
export default function StackLayout() {
-  return <Stack />;
+  return <Stack>
+    <Stack.Screen
+      name="index"
+      options={{
+        headerShown: false,
+      }}
+    />
+  </Stack>
}

```

🏃**Try it** No headers! Woo

~~### Route precedence
When Expo Router is resolving a url, and two routes match that url, it has to have rules in order to pick one. Of course it needs to match what's actually in the URL.

Try going to an exhibit, copying the URL, and opening it in another tab. Which tab (in your layout) is selected? When navigating to the deep link fresh, it'd make more sense if it took you inside the Exhibits tab, since the Home tab is just random stuff and going "back" from there wouldn't make much sense. We have one form of precedence here- array syntax order! The first matching route will be the one that's first in the route group array.

6. Rename **(home,exhibits)** to **(exhibits,home)**

🏃**Try it** Entering an exhibit, copy the URL, and open it in a new browser tab. It should always be inside the Exhibits tab. But what about the back button?~~
(this doesn't seem to be an issue anymore!)

### `initialRouteName`
Try navigating to an exhibit and refreshing the page. It deep-links to the exhibit, which is great, but where's the back button? 

We want a back button to show up whenever we deep link to an exhibit, and it should take you back to exhibits. Whenever you start at a deep link and don't see a back button, you should suspect it's related to setting `initialRouteName` somewhere.

Here's the snippet of code you want... somewhere:

```tsx
export const unstable_settings = {
  exhibits: {
    initialRouteName: "exhibits/index",
  },
};
```

This means that "when you're in the `(exhibits)` group, the initial route should be `exhibits/index`, even if you start on a different page.

7. Try putting this snippet in **(exhibits)/exhibits/_layout.tsx**. This makes some sense, as you're in the Exhibits stack when this is needed.

🏃**Try it** Does it work? A bit of a bummer, but makes some sense, as the rest of the stack isn't here.

8. Try putting this snippet closer to the rest of the stack, inside a new **(home,exhibits)/_layout.tsx**.

🏃**Try it** What's that error?

Yeah, so you can only have one layout per route. The route group folder shares layout between the `(home)` and `(exhibits)` routes, so now you have duplicate routes :-/

So... let's put all the layout in the route group folder!

9. Mentally merge the separate `(home)` and `(exhibits)` layouts together into a single layout in **(home,exhibits)/_layout.tsx**. It will look like this:

```tsx
import React from "react";
import { Stack } from "expo-router";

export const unstable_settings = {
  exhibits: {
    initialRouteName: "exhibits/index",
  },
};

export default function StackLayout() {
  return (
    <Stack>
      <Stack.Screen
        name="index"
        options={{
          headerShown: false,
        }}
      />
      <Stack.Screen
        name="exhibits/index"
        options={{
          headerShown: false,
        }}
      />
    </Stack>
  );
}
```

10. Delete **(home)/_layout.tsx** and **(home)/_layout.tsx**.

🏃**Try it** Go from Home to an exhibit, from Exhibits to an exhibit, refresh the page. Hopefully it looks good and you can use the back buttons whenever you need to.

## Exercise 2: Instagram-style "public post" page
We want there to basically be two ways to access `/works/[workId]`:
1. When you're logged in and browsing, as a modal on top of the tab layout.
2. When you're receiving a direct link, as its own separate fullscreen layout.

This is another place _route groups_ can help. Besides making the same URL work in different tabs, they can make the same URL work in completely different parts of the app:
- `app/(app)/works/[workId]` has an outward-facing URL of `/works/[workId]`
- `app/(otherway)/works/[workId]` _also_ has an outward-facing URL of `/works/[workId]`

There's a few different ways you could navigate to each distinct route:
- In your URL bar of application code, address it specifically, including the groups (e.g., `app/(otherway)/works/[workId]`)
- Depending on where you are in the navigation tree already, if you just use the outward-facing URL (e.g., `works/[workId]`), you'll go to whichever route is closest.

### Works page doppleganger
Let's start by just duplicating the route component somewhere else and navigating there.

1. Create the **app/(direct)** folder (we'll call it this because our goal is to go _directly_ there with a deep link)
2. Inside of **app/(direct)**, create the **works** folder, and then inside of that, **[workId].tsx**. So, the whole route will be `app/(direct)/works/[workId]` (under `direct`, the route is the same as or original route - it's _shared_).
3. Import the screen component you already have into this new **[workId].tsx** file:
```tsx
import WorkIdScreen from "@/app/(app)/works/[workId]";

export default WorkIdScreen;
```

🏃**Try it** In your browser, start at the initial route, open a work. Now, add `(direct)` in front of the works route, e.g., `http://localhost:8081/(direct)/works/137259`. Does it look a little different, like there's a modal with nothing behind it?

Now try the scenario we've been talking about this whole time, where you share the link to a work of art with someone else. Go back through the entry point, open a work of art, copy the URL and open it in another tab. Does it go to the `(direct)` version or the original?

### Route precedence, again.


We once again have the same URL that could use two different routes. Expo Router needs to figure out which one to use when. In this case, Expo Router uses alphabetical order to break ties between route groups.

`/works/[workId]` matches both:
- `(app)/works/[workId]`, and
- `(direct)/works/[workId]`

But `(app)` is before `(direct)` in the alphabet, so `(app)` wins. Let's fix that by renaming `(direct)`

4. Rename the **(direct)** folder to **(1-direct)**.

🏃**Try it** In your browser, start at the initial route, open a work. Now copy that URL and open it in a new tab. Hopefully it goes to the `(direct)` version now.

> [!NOTE]
> So, why does it go to the `(app)` version still if I start from the initial route? That's because Expo Router will look for the _nearest-matching_ route before it goes back through the top and looks at alphabetical order. So, order is important, but proximity > order.

### Cleaning things up
Let's make this new direct link look like something intentional and not just a broken modal.

5. Refactor everything inside of the `ScrollView` (including the `ScrollView` itself) in **app/(app)/works/[workId].tsx** into a separate `ArtworkDetail` component that you can put inside a new **components/ArtworkDetail.tsx** file. The component will need to take at least one parameter, the `work` object that's returned from the `useWorkByIdQuery()` hook (feel free to do as Keith does and use the `any` type, he won't mind!).

<details>
  <summary>If you're running short on time or just don't feel like doing that, here's a suggested ArtworkDetail component you can paste right in:</summary>

```tsx
import React from "react";
import { ScrollView, View, Text } from "react-native";
import { Image } from "expo-image";
import { useSafeAreaInsets } from "react-native-safe-area-context";
import { stripTags } from "@/lib/utils";

interface ArtworkDetailProps {
  work: any;
}

export const ArtworkDetail = ({ work }: ArtworkDetailProps) => {
  const insets = useSafeAreaInsets();

  return (
    <ScrollView
      contentContainerStyle={{ paddingBottom: insets.bottom }}
      contentContainerClassName="bg-white"
    >
      <View className="flex-1">
        <View className="py-4 px-4 h-96 w-96 self-center">
          <Image
            className="flex-1"
            source={{ uri: work && work.images.web.url }}
            contentFit="contain"
            transition={500}
          />
        </View>
        <Text className="font-semibold text-3xl px-4 pt-2">{work?.title}</Text>
        <View className="flex-1">
          <View className="px-4 gap-y-2 py-2">
            {work?.creators.length ? (
              <Text className="text-lg italic font-light mb-2">
                {work.creators[0].description}
              </Text>
            ) : null}
            <View className="flex-row gap-2 flex-wrap">
              <View className="bg-tint-2 px-4 py-2 self-start">
                <Text className="text-l font-bold">{work?.date_text}</Text>
              </View>
              {work?.technique.split(",").map((item: string) => (
                <View className="bg-tint px-4 py-2 self-start" key={item}>
                  <Text className="text-l font-bold text-white whitespace-nowrap line-clamp-1">
                    {item}
                  </Text>
                </View>
              ))}
            </View>
          </View>
          {work?.description && (
            <>
              <Text className="text-xl font-semibold px-4 py-2">
                Description
              </Text>
              <View className="px-4 gap-y-2 pb-2">
                <Text className="text-lg leading-6 font-light">
                  {stripTags(work.description)}
                </Text>
              </View>
            </>
          )}
          {work?.did_you_know && (
            <>
              <Text className="text-xl font-semibold px-4 py-2">
                Did you know?
              </Text>
              <View className="px-4 gap-y-2 py-2">
                <Text className="text-lg leading-6 font-light">
                  {stripTags(work.did_you_know)}
                </Text>
              </View>
            </>
          )}
        </View>
      </View>
    </ScrollView>
  );
};

```

</details>

6. Update **(app)/works/[workId].tsx** to use `ArtworkDetail` (replace the `Scrollview` and everything inside it, pass the `work`).

<details>
  <summary>Suggested code for [workId].tsx:</summary>

```tsx
import { View, Pressable, Platform } from "react-native";
import { router, useLocalSearchParams } from "expo-router";
import Icon from "@expo/vector-icons/MaterialIcons";
import { useWorkByIdQuery } from "@/data/hooks/useWorkByIdQuery";
import { useFavStatusQuery } from "@/data/hooks/useFavStatusQuery";
import { useFavStatusMutation } from "@/data/hooks/useFavStatusMutation";
import colors from "@/constants/colors";
import { LoadingShade } from "@/components/LoadingShade";
import classNames from "classnames";
import { useAuth } from "@/data/hooks/useAuth";
import { ArtworkDetail } from "@/components/ArtworkDetail";

export default function WorkScreen() {
  const { workId } = useLocalSearchParams<{
    workId: string;
  }>();

  const { authToken } = useAuth();

  // query art API for the work
  const workQuery = useWorkByIdQuery(workId);
  const work = workQuery.data;

  // read fav status
  const favQuery = useFavStatusQuery(workId);
  const isFav = favQuery.data;

  // update fav status
  const favMutation = useFavStatusMutation();

  return (
    <View
      className={classNames(
        "bg-white sm:bg-black",
        "sm:bg-opacity-20",
        "flex-1 justify-center",
        Platform.OS === "android" && "pt-safe",
      )}
    >
      <View className="flex-1 sm:my-20 sm:w-3/4 sm:self-center ">
        <View className="flex-row align-middle justify-between bg-white px-4 py-4 border-b border-b-shade-2">
          <View className="justify-center px-2">
            <Pressable
              onPress={() => {
                router.back();
              }}
              className="justify-center items-middle"
            >
              <Icon name="close" size={28} />
            </Pressable>
          </View>
          <View className="justify-center px-4">
            {authToken && (
              <Pressable
                className="active:opacity-50"
                disabled={favQuery.isLoading || favMutation.isPending}
                onPress={() => {
                  favMutation.mutate({ id, status: !isFav });
                }}
              >
                <Icon
                  name={isFav ? "star" : "star-border"}
                  color={colors.tint}
                  size={28}
                />
              </Pressable>
            )}
          </View>
        </View>
        <ArtworkDetail work={work} />
        <LoadingShade isLoading={workQuery.isLoading || favQuery.isLoading} />
      </View>
    </View>
  );
}

```

</details>

7. Make **(1-direct)/works/[workId].tsx** its own unique screen, rendering `ArtworkDetail` without the favorite button, but full screen and with some buttons to either login (if no `authToken`) or go to the tabs if they are logged in. (HINT: don't overthink it, they're actually the same route).

<summary>Suggested code for (1-direct)/works/[workId].tsx</summary>
<details>

```tsx
import React from "react";
import { View, Platform, Text } from "react-native";
import { Link, useLocalSearchParams } from "expo-router";
import { Image } from "expo-image";
import { useWorkByIdQuery } from "@/data/hooks/useWorkByIdQuery";
import { LoadingShade } from "@/components/LoadingShade";
import classNames from "classnames";
import { ArtworkDetail } from "@/components/ArtworkDetail";
import { useAuth } from "@/data/hooks/useAuth";

export default function WorkScreen() {
  const { workId } = useLocalSearchParams<{
    workId: string;
  }>();

  // query art API for the work
  const workQuery = useWorkByIdQuery(workId);
  const work = workQuery.data;

  const { authToken } = useAuth();

  return (
    <View
      className={classNames(
        "bg-white",
        "flex-1 justify-center",
        Platform.OS === "android" && "pt-safe"
      )}
    >
      <View className="flex-row items-center justify-between bg-white border-b border-b-shade-2 my-2 mx-4">
        <View className={classNames("h-14 w-56")}>
          <Image
            source={require("@/assets/images/logo.svg")}
            className="w-full h-full"
          />
        </View>
        <Link href="/">
          <Text className="text-lg font-semibold text-primary color-tint">
            {authToken ? "Home" : "Login"}
          </Text>
        </Link>
      </View>
      <View className="flex-1 sm:self-center">
        <ArtworkDetail work={work} />
        <LoadingShade isLoading={workQuery.isLoading} />
      </View>
    </View>
  );
}
```

</details>
</summary>

🏃**Try it** Try navigating to a work, copying the link, opening it in another tab, or even in another browser or private browser window to simulate being logged out.

## Exercise 1b: Of `initialRouteName`'s and `Redirect`'s

Now that we always have the direct link version of the works page working, let's make it _really_ match Instagram, which either shows you the post-only view when you're logged out, or takes you right into the post as if you're logged in, with you feed underneath.

It really, really helps to understand something important about `initialRouteName` at this point: _it only applies to when you first load up the app/website_. It's simply a mechanism to ensure that a deep link into the app pushes whatever routes need to be underneath the deep link's screen onto the history.

However, in this example, a redirect to the same deep link after the app has already been loaded would be helpful. Fortunately, there's now a way to apply `initialRouteName` config to in-app navigation: the `withAnchor` prop.

1. Let's try the redirect first without `withAnchor`. In **app/(1-direct)/works/[workId].tsx** add this after all the hooks, but before the return statement:
```tsx
 if (authToken) {
  return <Redirect href={`/(app)/works/${workId}`} />;
}
```

Don't forget to `import { Redirect } from "expo-router";`

🏃**Try it** If you're logged in, now reloading the page on a work will take you to the same place you were, but... you can't go anywhere else. No back button, no tabs. 

Recall earlier that, before you added the `(1-direct)` route, that a direct link would open up the tabs screen underneath the works screen and there would be a back button so you could get back to that. That was `initialRouteName` in action. Expo Router can't assume that you always want a particular screen opened first when navigating somewhere (how would it know your intentions), so now there's a parameter to tell Router to respect `initialRouteName` on a particular navigation operation if the route has not already been loaded.

2. Add `withAnchor`:

```diff
 if (authToken) {
-  return <Redirect href={`/(app)/works/${id}`} />;
+  return <Redirect href={`/(app)/works/${id}`} withAnchor />;
}
```

~~3. Oh wait, see those red squiggles? This API isn't _quite_ available yet ([so close!](https://github.com/expo/expo/pull/32847)). I expect you'll be able to use the `Redirect` component soon for this. For now you can do it like this, it's about the same. Maybe you lose 1 tick in performance:~~

This is equivalent to using the imparative API in this way:

```tsx
  const router = useRouter();

  useEffect(() => {
    if (authToken) {
      router.replace(`/(app)/works/${workId}`, { withAnchor: true });
    }
  }, [authToken, router, workId]);
```
(import `useRouter` from `expo-router`)

🏃**Try it** If you're logged in, now reloading the page on a work will take you to the same place you were, and everything works as before!

## See the solution
[Solution PR](https://github.com/keith-kurak/expo-router-codemash-2025-starter/pull/4)

## Next exercise
[Exercise 5](05-router-side-quests.md)

