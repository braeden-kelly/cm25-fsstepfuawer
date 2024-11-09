# Side Quests
A collection of Router-y things: pick one or two that sound interesting and try them out!

### How these work
This last section is like that part of a long video game where you could either go to the final boss or perform a bunch of side quests to buff up your characters. I don't know who the final boss is in this case, but these are the side quests!

These are a little more self-guided and some are better documented than others. All of them are organized like a user story/ fit critera/ tasking, along with some documentation resources, so you can see if you can figure them out on your own. A handful have some sample code that you can use to walk your way to a solution (those might be the best ones to try if figuring these out yourself sounds a little complicated).

## Side Quests Vol. 1: React Navigation v.7 Goodies 

_Difficulty: lower, more guidance offered_

### Hiding the tab bar when deep inside a stack inside of a tab
_Unlocked after module 01 (standard tab navigator) or 03 (headless tabs version)_

#### User Story
As a mobile user, when navigating to an exhibit from the Exhibits tab, keeping the tab bar visible feels too busy. If it wasn't there, I could see more artwork, and it's easy enough to back out to the top screen of the Exhibits tab if I want to go to another tab.

#### Fit criteria
- For native mobile users, hide the tab bar when on `exhibit/[exhibitName]`

#### Tasks

[You can also follow along with a step by step with code examples](/sidequests/hide-tab-bar.md)

1. Follow the [split triggers guide](/sidequests/shared/split-triggers.md) to decouple available tab routes from the tab bar UI.
2. Inside of the tabs layout, use Expo Router's `useSegments` to read the segments of the URL, so you can see when you're on `exhibits` vs `exhibits/[exhibitName]`
3. Set the `hideTabs` variable based on being on a non-index route of a tab and when the platform isn't web
4. Hide the "visual tabs" when `hideTabs` is true.

#### Documentation
- [Headless tabs (Triggers)](https://docs.expo.dev/router/advanced/custom-tabs/#tabtrigger)

### Moving the tabs to a drawer when on mobile web
_Unlocked after module 03_

#### User Story
As a mbile web user, bottom tabs don't seem like what I would typically use on a site like this. How about a hamburger menu with a drawer?

#### Fit criteria
- When on web and less than the `sm` width, show a hamburger menu with the same links as you would see on the tab bar

#### Tasks

[You can also follow along with a step by step with code examples](/sidequests/drawer.md)

1. Follow the [split triggers guide](/sidequests/shared/split-triggers.md) to decouple available tab routes from the tab bar UI.
2. `npm install react-native-drawer-layout`
3. It's going to be hard to do this entirely with Nativewind selectors, so just import `useMediaQuery` and create an alternate return statement in the tabs layout file if this is a mobile width and we're on web.
4. Wrap this new mobile web-specific layout in `Drawer` from `react-native-drawer-layout`. 
5. Make the drawer open when a hamburger button in the top left is pressed (`FontAwesome` has such an icon). Use a state variable to control the drawer open status.
6. Add the tab links in the drawer. You'll have to use regular `Link`'s because `TabTrigger`'s cannot be outside of the `Tabs` navigator, which has to be inside of the `Drawer`.
7. To close the drawer after navigating, import `usePathname` from `expo-router`, and use a `useEffect` to set the tab open variable to false when the pathname changes.

#### Documentation
- [Drawer Layout](https://reactnavigation.org/docs/drawer-layout/)

### Add a search bar in the `exhibits/[exhibitName]` header
_Unlocked after Module 01_

#### User Story
As a user, I'd like to search for artwork by name when looking inside an exhibit.

#### Tasks
1. Use `Stack.Screen` within **[exhibitName].tsx** to add the search bar within the header for that screen.
2. Follow the directions for adding the search bar below.
3. Based on the text input, filter works by their names.

#### Documentation
- [Search Bar in Header](https://reactnavigation.org/docs/elements#headersearchbaroptions)

### Native tabs / landscape side bar

#### User Story
As a user, too much space is taken up by tabs when in landscape on my phone. Put those tabs on the right side. Also use the native tabs, I like them better. Maybe animate the transitions, too.

#### Tasks
1. Resurrect your React Navigation tabs from pre-Module 03. Conditionally render them in the tabs layout file if you're on iOS or Android.
2. Remove the `portrait` lock from **app.json**.
3. Add a simple landscape orientation check in the tabs layout file (e.g., import `useScreenDimensions` from `react-native` and check if width > height)
4. When in landscape, apply the `left` tab bar position as described in the docs below
5. Make that left tab sidebar compact (icons over the name)
6. Maybe try adding that swooshy animation described below.
7. (?) While you're here, if you wanted to add that tasty frosted glass translucency, I don't think anyone will complain.

#### Documentation
- [Tab bar position](https://reactnavigation.org/docs/bottom-tab-navigator#tabbarposition)
- [tabs animation](https://reactnavigation.org/docs/bottom-tab-navigator#animations)

## Side Quests Vol. 2: Native integrations

_Difficulty: still not that hard, but now you need to build your native app_

### Quick Actions
_Unlocked after Module 01_

#### User Story
As a user, I would like to have a quick shortcut from my home screen to the `visit` tab, so I can quickly see the museum hours and how to get there. I typically think, "It's midnight and I want to go to the museum", and then I just go there and find out it's not open. Maybe if I could just long press the icon and get an option to check the times, I wouldn't do this.

#### Fit Criteria
- Add an iOS quick action / Android app shortcut that goes directly to the Visit tab

#### Tasks
[Follow the quick actions tutorial here](https://expo.dev/blog/expo-quick-actions)

## Side Quests Vol. 3: New nav patterns

_Difficulty: pretty tricky, a good bit of route refactoring_

### Allow full browsing for not-logged in users (more Netflix than Instragram)
_Unlocked after Module 02_

#### User Story
As a user, I should be able to browse all the exhibits even if I'm not logged in.

#### Fit criteria
- Don't force users to the login screen if they're not logged in
- Hide the `profile` route when the user isn't logged in
- Add a login button on the Home tab that takes you back to the login screen
- Don't allow favoriting if you're not logged in

#### Tasks
1. Remove the unauthenticated redirect in **(app)/_layout.tsx**.
2. In **(tabs)/_layout.tsx**, use the `useAuth` hook to hide the profile tab by removing it from the list of tag triggers.
3. In **works/[workId].tsx**, use the `useAuth` hook to hide the favorite button.
4. Add a simple Login button on the Home tab. Use `router.replace` to navigate to the `/login` route.

### Preserve deep links after authentication
_Unlocked after Module 02_

#### User Story
As a user, if someone sends me a link to a work of art, but I'm not logged in, I should be able to login and then still go directly to that artwork.

#### Fit criteria
- Instead of redirecting to the login screen when logged out, present it as a modal over the rest of the app.
- Dismiss the modal once the user is logged in

#### Tasks
1. Remove the unauthenticated redirect in **(app)/_layout.tsx**.
2. Import `Modal` from `react-native` in **(app)/_layout.tsx**.
3. If `authToken` is undefined, show the modal. If it's defined, hide the modal.
4. Put the contents of **login.tsx** into the modal (or just use the whole thing as a component).
5. Remove any navigation calls from the login page/ component.

## Side Quests Vol. 4: Web deployment

_Difficulty: probably not that hard, but takes longer than the Vercel/Netlify instructions due to my poor decisions in building this demo_

### Deploy as an Express.js app on Fly.io
_Unlocked anytime, but you probably wouldn't know the API routes worked until after module 02_

_Due to me building the data layer as just some JSON files, this app is server-ful. Your supported deployment option without refactoring is an Express.js app.

#### How to do it
- [Follow my guide from a previous workshop](https://github.com/keith-kurak/miami-masterclass-2024/blob/main/workshop/bonus-deploy.md)

#### Documentation
- [API Routes deployment](https://docs.expo.dev/router/reference/api-routes/#deployment)