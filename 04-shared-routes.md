
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

### Works page doppleganger
We want there to basically be two ways to access `/works/[workId]`:
1. When you're logged in and browsing, as a modal on top of the tab layout.
2. When you're receiving a direct link, as its own separate fullscreen layout.

Starting from #2, we could branch off difference scenarios such as, when you're logged in, it goes straight into #1 and only shows #2 when you're not logged in, but we're going to focus for now on making #2 work in all cases.

This is where **route groupss** can help. They let us define what is in essance the same URL (as far as the user is concerned) in multiple places:
- **app/(app)/works/[workId]** has an outward-facing URL of **/works/[workId]**
- **app/(direct)/works/[workId]** _also_ has an outward-facing URL of **/works/[workId]**

There's a few different ways you could navigate to each distinct route:
- In your URL bar of application code, address it specifically, including the groups (e.g., **app/(direct)/works/[workId]**)
- Depending on where you are in the navigation tree already, if you just use the outward-facing URL (e.g., **works/[workId]**), you'll go to whichever route is closest.

## See the solution
Switch to branch: `04-shared-routes-solution`

## Bonus
- What if, instead of offering the public page for a work, whenever you got a direct link to a work of art but weren't logged in, you were presented with the login screen, and then after logging in, it took you to the work? In essence, it would stash your deep link until you took care of the prerequisite of logging in (so many apps don't do this!).

## Next exercise
[Exercise 5](02-dynamic-routes.md)

