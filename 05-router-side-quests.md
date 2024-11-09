# Side Quests
A collection of Router-y things: pick one or two that sound interesting and try them out!

### How these work
This last section is like that part of a long video game where you could either go to the final boss or perform a bunch of side quests to buff up your characters. I don't know who the final boss is in this case, but these are the side quests!

These are a little more self-guided and some are better documented than others. All of them are organized like a user story/ fit critera/ tasking, along with some documentation resources, so you can see if you can figure them out on your own. A handful have some sample code that you can use to walk your way to a solution (those might be the best ones to try if figuring these out yourself sounds a little complicated).

## Side Quests Vol. 1: React Navigation v.7 Goodies 

_Difficulty: lower, more guidance offered_

### Hiding the tab bar when deep inside a stack inside of a tab
_Available after module 01 or 03, depending on how you want to do it_

#### User Story
As a mobile user, when navigating to an exhibit from the Exhibits tab, keeping the tab bar visible feels too busy. If it wasn't there, I could see more artwork, and it's easy enough to back out to the top screen of the Exhibits tab if I want to go to another tab.

#### Fit criteria
- For native mobile users, hide the tab bar when on `exhibit/[exhibitName]`

#### Tasks

[You can also follow along with a step by step with code examples](/sidequests/hide-tab-bar.md)

1. Inside of the tabs layout, use Expo Router's `useSegments` to read the segments of the URL, so you can see when you're on `exhibits` vs `exhibits/[exhibitName]`
2. Set a `hideTabs` variable based on being on a non-index route of a tab and when the platform isn't web
3. Separate the tab buttons from the `TabTrigger`'s that define the available tab routes, which will have an `href` but no inner contents. The list of `TabTrigger`'s that define the buttons will have no `href` but will have `TabButton` as a child.
4. Hide the "visual tabs" when `hideTabs` is true.

#### Documentation
- Headless tabs (Triggers)

### Moving the tabs to a drawer when on mobile web

#### User Story
As a mbile web user, bottom tabs don't seem like what I would typically use on a site like this. How about a hamburger menu with a drawer?

#### Fit criteria
- When on web and less than the `sm` width, show a hamburger menu with the same links as you would see on the tab bar

#### Tasks

[You can also follow along with a step by step with code examples](/sidequests/drawer.md)

1. Separate the tab buttons from the `TabTrigger`'s that define the available tab routes, which will have an `href` but no inner contents. The list of `TabTrigger`'s that define the buttons will have no `href` but will have `TabButton` as a child. (if you also did hiding the tab bar, you already did this).
2. `npm install react-native-drawer-layout`
3. It's going to be hard to do this entirely with Nativewind selectors, so just import `useMediaQuery` and create an alternate return statement in the tabs layout file if this is a mobile width and we're on web.
4. Wrap this new mobile web-specific layout in `Drawer` from `react-native-drawer-layout`. 
5. Make the drawer open when a hamburger button in the top left is pressed (`FontAwesome` has such an icon). Use a state variable to control the drawer open status.
6. Add the tab links in the drawer. You'll have to use regular `Link`'s because `TabTrigger`'s cannot be outside of the `Tabs` navigator, which has to be inside of the `Drawer`.
7. To close the drawer after navigating, import `usePathname` from `expo-router`, and use a `useEffect` to set the tab open variable to false when the pathname changes.

#### Documentation
- [Drawer Layout](https://reactnavigation.org/docs/drawer-layout/)

### Add a search bar in the `exhibits/[exhibitName]` header

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

#### User Story
As a user, I would like to have a quick shortcut from my home screen to the `visit` tab, so I can quickly see the museum hours and how to get there. I typically think, "It's midnight and I want to go to the museum", and then I just go there and find out it's not open. Maybe if I could just long press the icon and get an option to check the times, I wouldn't do this.

#### Fit Criteria
- Add an iOS quick action / Android app shortcut that goes directly to the Visit tab

#### Tasks
[Follow the quick actions tutorial here](https://expo.dev/blog/expo-quick-actions)

## Side Quests Vol. 3: New nav patterns

_Difficulty: pretty tricky, a good bit of route refactoring_

### Allow full browsing for not-logged in users
#### User Story
As a user, I should be able to browse all the exhibits even if I'm not logged in.

#### Fit criteria
- Don't force users to the login screen if they're not logged in
- Hide the `profile` route when the user isn't logged in
- Add a login button on the Home tab that takes you back to the login screen
- Don't allow favoriting if you're not logged in

#### Tasks

### Preserve deep links after authentication

#### User Story
As a user, if someone sends me a link to a work of art, but I'm not logged in, I should be able to login and then still go directly to that artwork.

#### Fit criteria
- Instead of redirecting to the login screen when logged out, present it as a modal over the rest of the app.
- Dismiss the modal once the user is logged in

#### Tasks

## Side Quests Vol. 4: Web deployment

_Difficulty: probably not that hard, just takes a while due to my poor decisions in building this demo_

### Deploy as an Express.js app on Fly.io
_Due to me building the data layer as just some JSON files, this app is server-ful. Your supported deployment option without refactoring is an Express.js app.

#### How to do it
[Follow my guide from a previous workshop](https://github.com/keith-kurak/miami-masterclass-2024/blob/main/workshop/bonus-deploy.md)

_This builds on the Net

## Side quests
1. React Navigation v.7 sidebar on iPad (available anytime)
2. React Navigation v.7 drawer layout for mobile web (best after Module 03)
3. React Navigation v.7 preload (available anytime)
4. React Navigation v.7 large header (available anytime)
5. Fly.io full stack deploy (available after Module 02)
6. Login as a modal, selectively disable functionality when logged out (available after Module 03)
7. Expo quick actions (available anytime)

## Copied from Bonuses

- For an app like this, it'd probably be more appropriate to allow it to be viewed unauthenticated, and then enable the Profile tab and favoriting capability once logged in. There could be a log in button on the Home tab that then pushes the `login` route onto the outer stack, which is then popped off once login is complete. It's not a small lift, but worth a try if you'd like an extra challenge. Tips:
  - Use the `useAuth()` hook to read the `authToken` and hide elements like the Profile tab and favorite button. Hiding tabs works differently in Expo Router (use a `null` `href` prop): https://docs.expo.dev/router/advanced/tabs/#advanced.
  - The transparent modal technique used in Module 01 would work great for the login screen on web.

  - Hide the tab bar on mobile when navigating to an exhibit.

  - Simulate an iPad interface (or really load this up on an iPad device or simulator), and have it use the floaty tabs instead of the web-optimized top tabs. Except, on iPad, floaty tabs should be narrow and centered, not taking up the whole width.

  - What if, instead of offering the public page for a work, whenever you got a direct link to a work of art but weren't logged in, you were presented with the login screen, and then after logging in, it took you to the work? In essence, it would stash your deep link until you took care of the prerequisite of logging in (so many apps don't do this!).