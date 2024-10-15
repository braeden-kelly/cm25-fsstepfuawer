
# Module 02: API Routes and Authentication

### Goal
Add a backend to your Expo Router project that can handle authentication and storing user data, and prevent users from accessing protected routes if they're not authenticated.

### Concepts
- API routes: integrate a RESTful API backend right into your Expo Router project.
- Protected routes: checking for conditions within the main app layout to determine if the user is allowed to access that area of the app, and redirect them to a login screen if they are not.

### Tasks
- Finish wiring up the API route that reads and writes favorite status to artworks
- Add an API route for logging in (we'll simulate a simple username/password login)
- Store something in local storage to indicate if you are logged in or logged out
- Redirect out of the `(app)` route if the user is not logged in and they try to navigate to that atea, sending them to the login screen.

# Exercises

## Exercise 1. Add the `api/works/[workId]/fav` API route

API routes fit right into the Expo Router folder structure, matching their URL's using the same rules as client-side pages. So, route groups, dynamic routes, etc. apply to API routes.

An API route is defined by **+api** in the filename. API routes implement GET, POST, etc. functions. So, if you have a `GET` function in **app/api/works/[workId]/fav+api.ts**, you can go to `http://localhost:8081/api/works/[workId]/fav` in your browser and get a result.

> [!TIP]
> Even though you could put API routes right next to frontend routes, we recommend having a top-level **api** folder, so you don't have to worry about breaking your backend while reorganizing your frontend routes.

### Add the GET request

1. Let's add **app/api/works/[workId]/fav+api.ts** with that GET function:

```ts
import { Database } from '@/data/api/database';

export async function GET(request : Request, { workId }: Record<string, string>) {
  // read the favorites status from our database
  const database = new Database();
  const favStatus = await database.getFavoriteStatus(workId);
  // make a json response
  return Response.json(favStatus);
}
```
Don't sweat what the "database" is right now, the key lesson here is reading the request, doing something withit, and returning a response. If you look at it, you'll probably be horrified, as it's just a bunch of text files.

🏃**Try it.**
```
http://localhost:8081/api/works/92937/fav
```
in your browser. We can't write any favorites status yet, so you should get false. But... you should get something!

### Add the POST request

2. Now, add the POST function to the same file:
```ts
export async function POST(request : Request, { workId }: Record<string, string>) {
  // read the body for the payload
  const body = await request.json();
  const status = body.status;
  // write the updated status to our database
  const database = new Database(request.headers.get('authToken'));
  await database.setFavoriteStatus(workId, status);
  // make a json response
  return Response.json(status);
}
```

You could test this out right now with something like Postman, but we're about to write up GET and POST in the next exercise.

## Exercise 2. Call the API route (GET and POST) from your client code
The favorite button on a work of art (the star icon) doesn't do anything yet, but there are placeholders already for reading and writing favorites status when a work is rendered.

To keep this clean, these queries are encapsulated in their own custom hooks in **/app/(app)/works/[workId]/index.tsx**, which wrap Tanstack query calls:
```tsx
 // query art API for the work
  const workQuery = useWorkByIdQuery(id);
  const work = workQuery.data;

  // read fav status
  const favQuery = useFavStatusQuery(id);
  const isFav = favQuery.data;
```

Follow `useFavStatusQuery()` to its file inside **data/hooks**. These are standard RESTful API's that are called via the standard `fetch` API under the hood. Let's update **useFavStatusQuery.ts** to implement the actual GET request via fetch:

```diff
queryFn: async () => {
-  return false
+  const response = await fetch(`/api/works/${workId}/fav`, {
+    method: 'GET',
+    headers: {
+      authToken,
+    },
+  });
+  return await response.json();
},
```

<details>
  <summary>Expand to just get just the added code for easy copying</summary>

  ```tsx
export const useFavStatusQuery = function(workId: string) {
  const { authToken } = useAuth();
  // Queries
  const query = useQuery({
    queryKey: [`works:fav:${workId}`],
    queryFn: async () => {
      const response = await fetch(`/api/works/${workId}/fav`, {
        method: 'GET',
        headers: {
          authToken,
        },
      });
      return await response.json();
    },
  });

  return query;
}
  ```

</details>

Let's do the same with **useFavStatusMutation.ts**, implementing the POST:

```diff
mutationFn: async (favStatus: { workId: string; status: boolean }) => {
  const { id, status } = favStatus;
-  return false;
+  const response = await fetch(`/api/works/${workId}/fav`, {
+    method: "POST",
+    headers: {
+      Accept: "application.json",
+      "Content-Type": "application/json",
+      authToken,
+    },
+    cache: "default",
+    body: JSON.stringify({ status }),
+  });
+  return await response.json();
},
```

<details>
  <summary>Expand to just get just the added code for easy copying</summary>

  ```tsx
export const useFavStatusMutation = function () {
  const queryClient = useQueryClient();
  const { authToken } = useAuth();

  // Queries
  const query = useMutation({
    mutationFn: async (favStatus: { workId: string; status: boolean }) => {
      const { id, status } = favStatus;
      const response = await fetch(`/api/works/${workId}/fav`, {
        method: "POST",
        headers: {
          Accept: "application.json",
          "Content-Type": "application/json",
          authToken,
        },
        cache: "default",
        body: JSON.stringify({ status }),
      });
      return await response.json();
    },
    onSuccess: (data, variables) => {
      queryClient.setQueryData([`works:fav:${variables.workId}`], variables.status);
      queryClient.invalidateQueries({ queryKey: ['favs'] })
    },
  });

  return query;
};
  ```

</details>

> [!NOTE]  
> What's with `authToken`? Uh...just think of it as foreshadowing.

**Try it:** Navigate to a work and try to favorite it. Try to unfavorite it. You should see that star fill and unfill.

**Try it (2):** Check out the profile tab! It's filling up your favorites now, thanks to the `/api/works/favs` API route, which was already wired up.

## Exercise 3: Shut the front door: Authentication (frontend edition)

## Exercise 4: Tighten the locks: wire the authentication to the backend

## Bonus
- ???

## See the solution
Switch to branch: `01-hello-router-solution`

## Next exercise
[Exercise 2](02-dynamic-routes.md)


