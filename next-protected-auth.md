# Presentation

Authentication is a basic feature of most Web Applications, so we, [at Flexper](https://flexper.hashnode.dev/), decided to make a library to simplify how to implement it inside our NextJS applications.

In this article we will walk through the basic settings of our library [next-protected-auth](https://github.com/flexper/next-protected-auth) for user authentication by creating two routes to create a simple proof of concept, a public one '**/'** and a protected one, '**/protected'**, only available if a user is logged.

# Install

First of all, let's install the library inside [a basic NextJS application](https://nextjs.org/docs/api-reference/create-next-app)

```bash
npm i next-auth-protected
```

# Setup \_app.tsx

Now, let's create our \_app.tsx file which wraps every route of a NextJS project.

We use the **'useNextAuthProtectedHandler'** to configure how the library should behave. And wrap all of the child components of our app inside our **NextAuthProvider** context.

As you can see, we declare a *loginRoute* parameter, this is where a user will be redirected if he tries to access a protected route without being logged in. We also declare which routes will be accessible for non-logged users as well as how to refresh a logged-in user access token.

In the next step, we will see how the **refreshToken** function and other API calls are handled.

```typescript
// File pages/_app.tsx

import type { AppProps } from 'next/app';

import {
  NextAuthProvider,
  useNextAuthProtected,
  useNextAuthProtectedHandler,
} from 'next-protected-auth';

import { refreshToken } from '../src/myAPI'

function CustomApp({ children }) {
  useNextAuthProtectedHandler({
    publicURLs: ['/auth/signup', '/'],
    loginURL: '/auth/signin',
    renewTokenFct: (oldAccessToken) => {
       if (!oldAccessToken) {
        throw 'not connected';
      }

      const { accessToken } = await refreshToken();

      return accessToken as string;
    },
  });

  const { isConnected } = useNextAuthProtected();

  console.log({ isConnected });

  return <>{children}</>;
}

const MyApp = ({ Component, pageProps }: AppProps) => {
  return (
    <NextAuthProvider>
      <CustomApp>
        <Component {...pageProps} />
      </CustomApp>
    </NextAuthProvider>
  );
};

export default MyApp;
```

# API wrapper

Before moving any further, let's have a first look at our **myAPI** file.

In the *\_app.tsx* file above, we worked with the *'useNextAuthProtectedHandler'* that is using a `refreshToken` function from our API wrapper.

> It is good practice to have **both an accessToken and a refreshToken**.

The **accessToken** is used to **make all the API calls** of the application to the server and **should not have a great lifespan** as we want to prevent bad intended people to sniff and use its value for wrong purposes. Something like a day. With *next-protected-app*, the **accessToken** is stored inside **localStorage.**

The **refreshToken** on the other hand **serves to refresh the value of accessToken** without the user having to log in again, its lifespan is more something like a week, and after its expiration, the user will have to log in manually again.
With *next-protected-app*, the **refreshToken** is stored inside **a cookie**, it does not transit through every API request like the accessToken, yet it's good practice to make the user renew it once in a while.

In order to refresh the **accessToken** of a user, we simply make a request to our server *(http://my-api.io)* with the **credentials** option to also pass cookies to the server.

Those cookies contain a **refreshToken** as it is set server-side by our real API at *http://my-api.io*.

```typescript
// File src/myAPI.ts

const AUTH_BASE_PATH = 'http://my-api.io';

const refreshToken = async () => {
  const res = await fetch(`${AUTH_BASE_PATH}/refresh`, {
    method: 'POST',
    headers: {
      Accept: 'application/json',
    },
    credentials: 'include',
  });

  if (res.ok) {
    return res.json(); // An object with an 'accessToken' string value
  }

  throw res.json();
}
```

For the login, we will make a hook in order to synchronize the token value received from our API with the `NextAuthProvider` that exposes the auth context of *next-protected-auth.*

Let's add some code to our *myAPI* file.

```typescript
// File src/myAPI.ts

import { setAccessToken, useNextAuthProtected } from 'next-protected-auth';

//////////

type LoginBody = {
  email: string;
  password: string;
}

type LoginResponse = {
  expirationTime: string;
  accessToken: string;
}

//////////

const login = async ({ email, password }: LoginBody): Promise<LoginResponse> => {
  const res = await fetch(`${AUTH_BASE_PATH}/login`, {
    method: 'POST',
    headers: {
      Accept: 'application/json',
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(payload),
    credentials: 'include',
  });

  if (res.status !== 200) {
    throw new Error('Login error');
  }

  return res.json();
};

//////////

export const useLogin = () => {
  const { setIsConnected } = useNextAuthProtected();

  const [isLoading, setIsLoading] = useState<boolean>(false);
  const [error, setError] = useState<string>();

  const login = useCallback(
    async ({ email, password }: LoginBody) => {
      try {
        setIsLoading(true);

        const { accessToken } = await login({ email, password });

        setAccessToken(accessToken);
        setIsConnected(true);
      } catch (e) {
        setError(e);

        throw e;
      } finally {
        setIsLoading(false);
      }
    },
    [setIsConnected, setAccessToken, setIsLoading, setError],
  );

  return {
    login,
    isLoading,
    error,
  };
};
```

As you can see, our *useLogin* hook is plugged in with [next-protected-auth](https://github.com/flexper/next-protected-auth) to both know if a user has been properly logged in (through *setIsConnected),* and to set his *accessToken* across the application for API calls, that's why we wrapped our application in the *NextAuthProvider in the first place.*

# Signup / Signin

You are completely free to design how the page for **login and signup** should look as long as you are using our **useLogin** from above or some other form of abstraction of next-protected-auth base functions.

# Signout

The signout route is pretty straightforward as it just needs to unload the accessToken from our NextAuthProvider\*.\*

After a user landed on the page, we call the **logout** function from our myAPI wrapper and then redirect the user to the sign-in process through our */auth/signin* route.

It is expected that **server-side**, we remove the cookie from the request and invalidate the *refreshToken* value stored in DB.

```typescript
import { NextAuthProtectedLogout } from 'next-protected-auth';

import { logout } from '../src/myAPI'

const Signout = NextAuthProtectedLogout({
  preCallback: async () => {
    await logout();
  },
  callback: () => {
    window.location.replace('/auth/signin');
  },
});

export default Signout;
```

# Business routes

Now that we have set up our authentication flow with library configuration and routes for the user, let's create two business routes, a public one **/** and a protected one, only available if a user is logged **/protected**

```typescript
// File pages/index.tsx -> '/' route

import type { NextPage } from 'next';

const Home: NextPage = () => {
  return <>HomePage</>;
};

export default Home;
```

```typescript
// File pages/protected.tsx -> '/protected' route

import type { NextPage } from 'next';
import { useNextAuthProtected } from 'next-protected-auth';

const Protected: NextPage = () => {
  const { isConnected } = useNextAuthProtected();

  console.log('Only logged in users can see this page, is this user logged in ?', isConnected);

  return <>A protected route</>;
};

export default Protected;
```

Simple wasn't it?

# Conclusion

And there you have it! Once you've configured the token management routine, you can create public and private routes on the fly by just declaring the public ones in the *useNextAuthProtectedHandler* under *\_app.tsx* !

The *next-protected-auth* library was meant to simplify the authentication process and enforce good practices while also being the most flexible possible to adapt to most use cases.

Feel free to [submit issues on the GitHub](https://github.com/flexper/next-protected-auth) if you're having any trouble and happy coding!