
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