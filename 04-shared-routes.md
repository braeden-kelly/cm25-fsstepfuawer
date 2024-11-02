
# Module 04: Shared routes

### Goal
Use groups and group precedence to make the same route go to two different places, depending on how arrive there.

### Concepts
- Create distinct routes that share the same outward-facing URL, but use `(group)` syntax to differentiate their position in the heirarchy. 
  - Take advantage of alphabetic precedence to ensure your "dominant" route is used when you want it to be.
- Use groups to create links that behave differently depending on whether you're navigating to them from another route or you're using them as your initial entry point.
- Share the same screen with two different sibling routes with array (e.g., `(route1, route2)`) syntax

### Tasks
- Create an alternate entry point into `works/[workId]` that lets anyone look at a work of art shared by a user, even if they're not logged in (Instagram-style)
- Turn `/exhibits/[exhibit]` into a shared route that is a sibling of both the Home and Exhibits tabs, so it pushes onto either while staying within the same tab. However, if you share a link to an exhibit, it will always open the Exhibits tab.

### Helpful links

# Exercises

## Exercise 1: Instagram-style "public post" page
We want there to basically be two ways to access `/works/[workId]`:
1. When you're logged in and browsing, as a modal on top of the tab layout.
2. When you're receiving a direct link, as its own separate fullscreen layout.

> [!NOTE] Starting from #2, we could branch off difference scenarios such as, when you're logged in, it goes straight into #1 and only shows #2 when you're not logged in, but we're going to focus for now on making #2 work in all cases.

This is where _route groups_ can help. They let us define what is in essance the same URL (as far as the user is concerned) in multiple places:
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
import WorkIdScreen from "@/app/(app)/works/[id]/index";

export default WorkIdScreen;
```

🏃**Try it** In your browser, start at the initial route, open a work. Now, add `(direct)` in front of the works route, e.g., `http://localhost:8081/(direct)/works/123456`. Does it look a little different, like there's a modal with nothing behind it?

Now try the scenario we've been talking about this whole time, where you share the link to a work of art with someone else. Go back through the entry point, open a work of art, copy the URL and open it in another tab. Does it go to the `(direct)` version or the original?

### Route precedence
When Expo Router is resolving a url, and two routes match that url, it has to have rules in order to pick one. Of course it needs to match what's actually in the URL (e.g., whatever it picks needs to have `works` and `[workId]` in that order, though groups don't count against the matching and could be in-between those).

In the future, other resolution methods may be available, but currently, Expo Router uses alphabetical order to break ties between route groups.

`/works/[workId]` matches both:
- `(app)/works/[workId]`, and
- `(direct)/works/[workId]`

But `(app)` is before `(direct)` in the alphabet, so `(app)` wins. Let's fix that by renaming `(direct)`

4. Rename the **(direct)** folder to **(1-direct)**.

🏃**Try it** In your browser, start at the initial route, open a work. Now copy that URL and open it in a new tab. Hopefully it goes to the `(direct)` version now.

> [!NOTE] So, why does it go to the `(app)` version still if I start from the initial route? That's because Expo Router will look for the _nearest-matching_ route before it goes back through the top and looks at alphabetical order. So, order is important, but proximity > order.

### Cleaning things up
Let's make this new direct link look like something intentional and not just a broken modal.

5. Refactor everything inside of the `ScrollView` in **app/(app)/works/[workId].tsx** into a separate `ArtworkDetail` component that you can put inside a new **components/ArtworkDetail.tsx** file. The component will need to take at least one parameter, the `work` object that's returned from the `useWorkByIdQuery()` hook (feel free to do as Keith does and use the `any` type, he won't mind!).

<details>
  <summary>If you're running short on time or just don't feel like doing that, here's a suggested `ArtworkDetail` component you can paste right in:</summary>

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
  const { id } = useLocalSearchParams<{
    id: string;
  }>();

  const { authToken } = useAuth();

  // query art API for the work
  const workQuery = useWorkByIdQuery(id);
  const work = workQuery.data;

  // read fav status
  const favQuery = useFavStatusQuery(id);
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

<summary>Suggested code for **(1-direct)/works/[workId].tsx**</summary>
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
  const { id } = useLocalSearchParams<{
    id: string;
  }>();

  // query art API for the work
  const workQuery = useWorkByIdQuery(id);
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

I take it back - we're going to fix the logged-in behavior of what we just did to _really_ match Instagram, which either shows you the post-only view when you're logged out, or takes you right into the post as if you're logged in, with you feed underneath.

It really, really helps to understand something important about `initialRouteName` at this point: _it only applies 

## Exercise 2: Two stacks, one route: fixing the wonky `exhibits/[exhibitName]` route.
The whole time, you've been able to go to the Exhibits tab, click on an exhibit, and view that Exhibit while staying in the Exhibits tab... or, you can click on an exhibit name above the selected works on the Home tab, and it would take you to the exhibit under Exhibits tab, but back would take you "back" to the Exhibits tab, instead of back to the Home tab. 

Worse, a bug has returned after adding headless tabs, where doing this before navigating to the Exhibits tab at least once will result in a missing back button, because `exhibits/index` never gets placed on the stack.

Let's fix this! Individual exhibits should always be children of the `exhibits` route, but back should take you back to where you were regardless of where you come from. And, because we need to pick one route when it comes to sharing an exhibit link, we will make it always pick the Exhibits tab.

### Creating some space: tabs inside of groups
This is going to affect the `(tabs)/index` and `(tabs)/exhibits` routes, so lets add an extra layer to each of those so we have some space to define a shared stack layout and shared routes between the two. Since the "layer" will be route groups, they will not actually affect the URL's.

1. Create an **app/(app)/(tabs)/(home)** folder and put **(tabs)/index.tsx** inside it.
2. Create an **app(app)/(tabs)/(exhibits)** folder and put **(tabs)/exhibits** inside it (including everything inside that folder).
3. Update **app(app)/(tabs)/_layout.tsx** to account for the new home for the index route:
```diff
- <TabTrigger name="index" href="/" asChild>
+ <TabTrigger name="index" href="/(home)" asChild>
```

🏃**Try it** Move around between Home and exhibits. Some stuff is probably working, but it probably looks odd (extra headers on exhibits??) and doesn't really go to all the right places.

4. Let's start sharing route information by sharing a layout between `home` and `exhibits`. Move **(tabs)/(exhibits)/exhibits/_layout.tsx** to a new folder, **(tabs)/(home,exhibits)**. Then add a second stack screen to make sure we're hiding the header whenever we're at the `exhibits` index:
```tsx
<Stack.Screen
  name="exhibits/index"
  options={{
    headerShown: false,
  }}
/>
```

🏃**Try it** The array synxtax of the route shares this layout whether you're inside the `(home)` or `(exhibits)` group, meaning that `exhibits` should not show an extra unwanted header bar no matter how we enter it.

<!--TODO: this may not be a good lesson above because it may depend on what's underneath it. Need to test!-->

### Stacking up right: staying in your tab
At this point, navigating to an exhibit from the home tab should still be moving you over to the Exhibits tab. That's because `exhibits/[exhibitName]` is only under `(exhibits)`. We need it to be under both `(home)` and `(exhibits)` in order to stay within the tab where you entered the route from.

5. Move **(app)/(tabs)/(exhibits)/exhibits** (and all of its contents) to **(home,exhibits)**.

🏃**Try it** Entering an exhibit from either tab should keep you in that tab. That's because the `exhibits/[exhibitName]` is in _both tabs_, and Expo Router is picking the nearest matching route.

### Route precedence, again
Try going to an exhibit, copying the URL, and opening it in another tab. Which tab (in your layout) is selected? When navigating to the deep link fresh, it'd make more sense if it took you inside the Exhibits tab, since the Home tab is just random stuff and going "back" from there wouldn't make much sense. We have one more form of precedence here- array syntax order! The first matching route will be the one that's first in the route group array.

6. Rename **(home,exhibits)** to **(exhibits,home)**

🏃**Try it** Entering an exhibit, copy the URL, and open it in a new browser tab. It should always be inside the Exhibits tab. But what about the back button?

### `initialRouteName`, again
We want a back button to show up whenever we deep link to an exhibit, and it should take you back to exhibits. Whenever you start at a deep link and don't see a back button, you should suspect it's related to setting `initialRouteName` somewhere.

7. In **(tabs)/(exhibits,home)/_layout.tsx**, update the `unstable_settings` to set the initial route to `exhibits/index` whenever you enter through the `(exhibits)` group (a mouthful!):
```tsx
export const unstable_settings = {
  initialRouteName: "index",
  exhibits: {
    initialRouteName: "exhibits/index",
  },
};
```

🏃**Try it** Things should be looking pretty good by now, whether you're navigating around or deep-linking to an exhibit.

## See the solution
Switch to branch: `04-shared-routes-solution`

## Bonus
- What if, instead of offering the public page for a work, whenever you got a direct link to a work of art but weren't logged in, you were presented with the login screen, and then after logging in, it took you to the work? In essence, it would stash your deep link until you took care of the prerequisite of logging in (so many apps don't do this!).

## Next exercise
[Exercise 5](02-dynamic-routes.md)

