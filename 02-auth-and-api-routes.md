
# Module 02: Authentication (and API routes)

### Goal
Let's use protected routes to prevent users from accessing logged-in routes if they're not authenticated, regardless of whether they're on mobile or web. We'll also get a brief introduction to Expo Router's full stack capabilities.

### Concepts
- Protected routes: checking for conditions within the main app layout to determine if the user is allowed to access that area of the app, and redirect them to a login screen if they are not.
- API routes: integrate a RESTful API backend right into your Expo Router project.

### Tasks
- Store something in local storage to indicate if you are logged in or logged out
- Redirect out of the `(app)` route if the user is not logged in and they try to navigate to that area, sending them to the login screen.
- Add some API routes to fill out the logged-in user experience, allowing users to favorite artwork and see them later.

# Exercises
You might have noticed the ability to favorite artworks when you navigate to them. However, this isn't very compelling without the ability to login and identify yourself.

Let's get the navigation mechanics of login working by simulating logged-in / logged-out status via a local variable. Then we'll get it working end-to-end.

## Exercise 1: Shut the front door: authentication (frontend edition)

Let's present a login screen that will always show until the user logs in. You could probably guess that simple navigation requests (navigating to a login screen based on logged-in status) could do this. This could work on mobile, but we need to think universal here! On web, someone could just change the URL and end up one of the pages that you should only see when you're logged in. We need to prevent that. Even though you could live without it, this pattern ends up being very helpful for mobile, as well.

### Protected routes
The concept behind this is _protected routes_. Within the `(app)` route group, we'll redirect out to the `login` route if certain conditions are not met (such as having a valid auth token). This _protects_ everything under **(app)** from being accessed by someone who isn't logged in.

We can add this protection inside **(app)/_layout.tsx**. When you navigate within a route, the layout is rendered before any of the child routes are rendered. So, if the **_layout** file "renders" a redirect to `login`, the child routes never render and the user sees the login page. 

Meanwhile, `login` route will attempt to navigate back to `(app)` once login is successful. This causes the layout to be rendered again. This time, the layout detects that the user has a valid auth token, and proceeds with rendering the child route.

### Lock the door (kick out logged-out users ... that's everyone right now)
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

> [!INFO]
> What is this magical `useAuth` hook? Look inside. It's just a wrapper around a Jotai local state store, which is also wired to local storage. For auth, you need some kind of global state (so you can detect anywhere if you're logged out). The vanilla React way to do this would be with Context, and the [Expo docs](https://docs.expo.dev/router/reference/authentication/#example-authentication-context) even provide an example of this, but state libraries reduce this to a few lines of code, and they also often include turnkey integration with local storage (which you need to "remember" if the user is logged-in).

### Provide a key (let in logged-in users)
The "Log in" button on the login page is already wired up to call the `login` function, which sets the auth token, but a) `login` itself doesn't do anything, and b) it doesn't navigate anywhere. Let's fix that:

3. In **data/hooks/useAuth.ts**, you'll see we already have a Jotai store set up to store `authToken`, and that's wired to async storage. But there's nothing in the `login` function. We're not ready to worry about logging into an actual server. Let's just set the token for now to simulate logging in for now:
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
      <Button
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

🏃**Try it:** Log in and log out a few times. The logout button is on the Profile tab, and just clears out the auth token from local storage. It should feel like an actual login workflow.

> [!NOTE]  
>  The automatic rerender of **app/(app)/_layout.tsx** happens because Jotai hooks react to state changes, which in turn causes our `useAuth` hook to updaet. That's why all logging out needs to do is update the `authToken`.

## Exercise 2: Tighten the locks: wire the authentication to the backend
This is a workshop primarily about frontend navigation, but given that you've probably noticed the **api** folder, we should probably address that. Expo Router supports not only frontend navigation, but backend routes, as well. It can host a full-stack website in a single project, including a RESTful API that will also be used by the native mobile version of your app.

We're going to wire up just enough to making logging in useful, adding the `login` endpoint itself, and the endpoint for populating the Favorites tab.

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

🏃**Try it:** Login and logout a few times. Hopefully it all still works! The unique "users" in this not-realistic simulation are actually ID'ed by a hash of the email and password 🙈, so try the same email and password to see your data persist. (Again, please don't do this in real life!)

3. Add a **api/works/favs+api.ts** route, and paste in the following GET request:

```ts
import { Database } from "@/data/api/database";

export async function GET(request: Request) {
  const database = new Database(request.headers.get("authToken"));
  const favs = await database.getFavorites();
  return Response.json(favs);
}
```

2. Update **data/hooks/useFavsQuery.ts** to call the new API endpoint:
```diff
export const useFavsQuery = function () {
   const { authToken } = useAuth();
   const query = useQuery({
     queryKey: [`favs`],
     queryFn: async () => {
-      return [];
+      const response = await fetch(`/api/works/favs`, {
+        method: "GET",
+        headers: {
+          authToken,
+        },
+      });
+     return await response.json();
    },
  });

  return query;
};
```

<details>
  <summary>Expand to just get the whole function for easy copying</summary>

  ```tsx
export const useFavsQuery = function () {
  const { authToken } = useAuth();
  const query = useQuery({
    queryKey: [`favs`],
    queryFn: async () => {
      const response = await fetch(`/api/works/favs`, {
        method: "GET",
        headers: {
          authToken,
        },
      });
      return await response.json();
    },
  });

  return query;
};

  ```
</details>

🏃**Try it:** Fav some artwork. It should show up on the Favorites tab!

## See the solution
[Solution PR](https://github.com/keith-kurak/expo-router-codemash-2025-starter/pull/2)

## Next exercise
[Module 03](03-headless-tabs-and-responsiveness.md)


