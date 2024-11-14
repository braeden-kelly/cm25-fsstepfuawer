
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
- Redirect out of the `(app)` route if the user is not logged in and they try to navigate to that area, sending them to the login screen.

# Exercises

## Exercise 1. Add the `api/works/[workId]/fav` API route

API routes fit right into the Expo Router folder structure, matching their URL's using the same rules as client-side pages. So, route groups, dynamic routes, etc. apply to API routes.

An API route is defined by **+api** in the filename. API routes implement GET, POST, etc. functions. So, if you have a `GET` function in **app/api/works/[workId]/fav+api.ts**, you can go to `http://localhost:8081/api/works/[workId]/fav` in your browser and get a result.

> [!TIP]
> Even though you could put API routes right next to frontend routes, we recommend having a top-level **api** folder, so you don't have to worry about breaking your backend while reorganizing your frontend routes.

### Add the GET request

1. Create a folder called **[workId]** in **app/api/works/[workId]** and a file called **fav+api.ts** in **app/api/works/[workId]/fav+api.ts**. Open this file and add the GET function:

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
Don't sweat what the "database" is right now, the key lesson here is reading the request, doing something with it, and returning a response. If you look at it, you'll probably be horrified, as it's just a bunch of text files.

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
  const database = new Database();
  await database.setFavoriteStatus(workId, status);
  // make a json response
  return Response.json(status);
}
```

You could test this out right now with something like Postman, but we're about to write up GET and POST in the next exercise.

## Exercise 2. Call the API route (GET and POST) from your client code
The favorite button on a work of art (the star icon) doesn't do anything yet, but there are placeholders already for reading and writing favorites status when a work is rendered.

To keep this clean, these queries are encapsulated in their own custom hooks in **/app/(app)/works/[workId].tsx**, which wrap Tanstack query calls:
```tsx
 // query art API for the work
  const workQuery = useWorkByIdQuery(id);
  const work = workQuery.data;

  // read fav status
  const favQuery = useFavStatusQuery(id);
  const isFav = favQuery.data;
```

Follow `useFavStatusQuery()` to its file inside **data/hooks/useFavStatusQuery.ts**. These are standard RESTful API's that are called via the standard `fetch` API under the hood. Let's update **useFavStatusQuery.ts** to implement the actual GET request via fetch:

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
  <summary>Expand to just get the whole function for easy copying</summary>

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

Let's do the same with **data/hooks/useFavStatusMutation.ts**, implementing the POST:

```diff
mutationFn: async (favStatus: { workId: string; status: boolean }) => {
  const { workId, status } = favStatus;
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
  <summary>Expand to just get just the whole function for easy copying</summary>

  ```tsx
export const useFavStatusMutation = function () {
  const queryClient = useQueryClient();
  const { authToken } = useAuth();

  // Queries
  const query = useMutation({
    mutationFn: async (favStatus: { workId: string; status: boolean }) => {
      const { workId, status } = favStatus;
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
> What's with `authToken`? Uh...just think of it as foreshadowing. Stay tuned for Exercise 3!

🏃**Try it:** Navigate to a work and try to favorite it. Try to unfavorite it. You should see that star fill and unfill.

🏃**Try it (2):** Check out the profile tab! It's filling up your favorites now, thanks to the `/api/works/favs` API route, which was already wired up.

## Exercise 3: Shut the front door: authentication (frontend edition)

The ability to favorite artwork isn't very compelling without the ability to login and identify yourself (currently everything saves to a single user 🙈). Let's present a login screen that will always show until the user logs in.

### Protected routes
The concept behind this is _protected routes_. Within the `(app)` route group, we'll redirect out to the `login` route if certain conditions are not met (such as having a valid auth token). This _protects_ everything under **(app)** from being accessed by someone who isn't logged in.

We can add this protection inside **(app)/_layout.tsx**. When you navigate within a route, the layout is rendered before any of the child routes are rendered. So, if the **_layout** file "renders" a redirect to `login`, the child routes never render and the user sees the login page. 

Meanwhile, `login` route will attempt to navigate back to `(app)` once login is successful. This causes the layout to be rendered again. This time, the layout detects that the user has a valid auth token, and proceeds with rendering the child route.

### Lock the door (kick out logged-out users)
1. Create a **login.tsx** file inside of **app**. You can copy the code from our [login.tsx starter here](/files/02/login.tsx).
2. In **app/(app)/_layout.tsx**, add a conditional to redirect if `authToken` is not set:

```diff
export default function layout() {
+  const { authToken } = useAuth();
+
+  if (!authToken) {
+    return <Redirect href="/login" />;
+  }

  return (
```

Don't forget to `import { useAuth } from "@/data/hooks/useAuth";` and `import { Redirect } from "expo-router";`

🏃**Try it:** You should now be "trapped" at the login page. In a browser, try setting another URL. It should not work.

### Provide a key (let in logged-in users)
The "Log in" button on the login page is already wired up to call the `login` function, which sets the auth token, but a) `login` itself doesn't do anything, and b) it doesn't navigate anywhere. Let's fix that:

3. In **api/hooks/useAuth.ts**, you'll see we already have a Jotai store set up to store `authToken`, and that's wired to async storage. But there's nothing in the `login` function. Let's just set the token for now:
```diff
const login = async (email: string, password: string) => {
+    await setAuthToken("whatever");
  };
```

🏃**Try it:** Type anything into the email/password form and press Login. Nothing will happen, but refresh the page/app. You should be logged in! The reload forced the app to try to go back into `(app)` (Expo Router looks for the first `index` route, which is in your tabs). This time, it found an auth token because pressing Login stored it in local storage.

4. Back in **login.tsx**, use Expo Router imparatively to navigate to `/(app)` after calling `login()`:
```diff
export default function LoginScreen() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");

  const { login } = useAuth();

+  const router = useRouter();

  return (
    <View className="flex-1 justify-center items-center gap-y-4 bg-shade-0">
      <TextField label="Email" text={email} setText={setEmail} autofocus />
      <TextField
        label="Password"
        text={password}
        setText={setPassword}
        isSecure
      />
      <Pressable
        onPress={async () => {
          await login(email, password);
+          router.replace('/(app)');
        }}
      >
        <View className="py-4 px-8 bg-tint">
          <Text className="text-white">Log in</Text>
        </View>
      </Pressable>
    </View>
  );
}
```

Don't forget to `import { useRouter } from "expo-router";`

`replace` removes history from the stack and ensures that you can't go back with a back button or swipe gesture.

🏃**Try it:** Log in and log out a few times. It should feel like an actual login workflow.

> [!NOTE]  
> The Expo Router instructions demonstrate this workflow using React context, which would be the built-in way to share state like an auth token across your app. But this app uses a little bit of Jotai, and Jotai enables reactive data, causing an automatic rerender of **app/(app)/_layout.tsx**, which is why all logging out needs to do is update the `authToken`.

## Exercise 4: Tighten the locks: wire the authentication to the backend
Let's add the actual login API route and wire things up to that.

> [!WARNING]  
> Do not interpret any code in here as anything even vaguely resembling a secure authentication system.

1. Add the **app/api/login+api.ts** route, with this code:
```ts
import { Database } from '@/data/api/database';

export async function POST(request : Request) {
  const body = await request.json();
  const database = new Database();
  const authToken = await database.login(body.email, body.password);
  return Response.json({ authToken});
}
```

2. Update `login` inside of **data/hooks/useAuth.ts** to call this API:
```diff
const login = async (email: string, password: string) => {
-  await setAuthToken("whatever");
+  const response = await fetch(`/api/login`, {
+    method: "POST",
+    headers: {
+      Accept: "application.json",
+      "Content-Type": "application/json",
+    },
+    cache: "default",
+    body: JSON.stringify({ email, password }),
+  });
+  const data = await response.json();
+  await setAuthToken(data.authToken);
};
```

<details>
  <summary>Expand to just get the whole function for easy copying</summary>

  ```tsx
const login = async (email: string, password: string) => {
  const response = await fetch(`/api/login`, {
    method: "POST",
    headers: {
      Accept: "application.json",
      "Content-Type": "application/json",
    },
    cache: "default",
    body: JSON.stringify({ email, password }),
  });
  const data = await response.json();
  await setAuthToken(data.authToken);
};
  ```

</details>

3. Update both of the the requests inside **app/api/works/[workId]/fav+api.ts** to read the auth token from the header:
```diff
- const database = new Database();
+ const database = new Database(request.headers.get('authToken'));
```

(Again, don't worry about what this database is doing (I assure you it's unremarkable / possibly horrifying), the important thing is that we passed data through a header that we can read on the other end.)

🏃**Try it:** Login and logout a few times. Hopefully it all still works! The unique "users" in this not-realistic simulation are actually ID'ed by a hash of the email and password 🙈, so try the same email and password to see your data persist. (Again, please don't do this in real life!)

## See the solution
[Solution PR](https://github.com/keith-kurak/expo-router-london-2024-starter/pull/2)

## Next exercise
[Module 03](03-headless-tabs-and-responsiveness.md)


