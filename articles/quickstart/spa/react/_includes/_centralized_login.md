<!-- markdownlint-disable MD002 MD041 MD034 -->

<%= include('../../_includes/_login_preamble', { library: 'React' }) %>

## Create a Sample Application

::: note
The following tutorial creates a new React application using `create-react-app`, and presents some common ways to build React applications, in terms of its structure and naming conventions. If you are using this guide to integrate the Auth0 SDK into your own React application, you may need to adjust some of the steps to suit your scenario.
:::

If you don't already have an existing application, you can create one using the [Create React App](https://facebook.github.io/create-react-app/) CLI tool. Using the terminal, find a location on your drive where you want to create the project and run the following commands:

```bash
# Create the application using create-react-app
npx create-react-app my-app

# Move into the project directory
cd my-app
```

(npx comes with npm 5.2+ and higher, see [instructions for older npm versions](https://gist.github.com/gaearon/4064d3c23a77c74a3614c498a8bb1c5f))

### Install initial dependencies

After creating a new React app using `create-react-app` install `react-router`, which doesn't come as standard with the boilerplate project. The [Auth0 Client SDK](https://github.com/auth0/auth0-spa-js) should also be added.

Install these two packages using the following command in the terminal:

```bash
npm install react-router-dom @auth0/auth0-spa-js
```

### Install the Auth0 React wrapper

Create a new file in the `src` directory called `react-auth0-wrapper.js` and populate it with the following content:

```js
// src/react-auth0-wrapper.js
import React, { useState, useEffect, useContext } from "react";
import createAuth0Client from "@auth0/auth0-spa-js";

const DEFAULT_REDIRECT_CALLBACK = () =>
  window.history.replaceState({}, document.title, window.location.pathname);

export const Auth0Context = React.createContext();
export const useAuth0 = () => useContext(Auth0Context);
export const Auth0Provider = ({
  children,
  onRedirectCallback = DEFAULT_REDIRECT_CALLBACK,
  ...initOptions
}) => {
  const [isAuthenticated, setIsAuthenticated] = useState();
  const [user, setUser] = useState();
  const [auth0Client, setAuth0] = useState();
  const [loading, setLoading] = useState(true);
  const [popupOpen, setPopupOpen] = useState(false);

  useEffect(() => {
    const initAuth0 = async () => {
      const auth0FromHook = await createAuth0Client(initOptions);
      setAuth0(auth0FromHook);

      if (window.location.search.includes("code=")) {
        const { appState } = await auth0FromHook.handleRedirectCallback();
        onRedirectCallback(appState);
      }

      const isAuthenticated = await auth0FromHook.isAuthenticated();

      setIsAuthenticated(isAuthenticated);

      if (isAuthenticated) {
        const user = await auth0FromHook.getUser();
        setUser(user);
      }

      setLoading(false);
    };
    initAuth0();
    // eslint-disable-next-line
  }, []);

  const loginWithPopup = async (params = {}) => {
    setPopupOpen(true);
    try {
      await auth0Client.loginWithPopup(params);
    } catch (error) {
      console.error(error);
    } finally {
      setPopupOpen(false);
    }
    const user = await auth0Client.getUser();
    setUser(user);
    setIsAuthenticated(true);
  };

  const handleRedirectCallback = async () => {
    setLoading(true);
    await auth0Client.handleRedirectCallback();
    const user = await auth0Client.getUser();
    setLoading(false);
    setIsAuthenticated(true);
    setUser(user);
  };
  return (
    <Auth0Context.Provider
      value={{
        isAuthenticated,
        user,
        loading,
        popupOpen,
        loginWithPopup,
        handleRedirectCallback,
        getIdTokenClaims: (...p) => auth0Client.getIdTokenClaims(...p),
        loginWithRedirect: (...p) => auth0Client.loginWithRedirect(...p),
        getTokenSilently: (...p) => auth0Client.getTokenSilently(...p),
        getTokenWithPopup: (...p) => auth0Client.getTokenWithPopup(...p),
        logout: (...p) => auth0Client.logout(...p)
      }}
    >
      {children}
    </Auth0Context.Provider>
  );
};
```

This is a set of custom [React hooks](https://reactjs.org/docs/hooks-intro.html) that enable you to work with the Auth0 SDK in a more idiomatic way, providing functions that allow the user to log in, log out, and information such as whether the user is logged in.

The next few sections will integrate these hooks into the various components that make up the app.

### Create the Navbar component

Create a new folder inside the `src` folder called `components`. This is where you will house all the components for this application.

Create a new component in the `components` folder called `Navbar.js`. This component will be responsible for showing the login and logout buttons:

```jsx
// src/components/NavBar.js

import React from "react";
import { useAuth0 } from "../react-auth0-wrapper";

const NavBar = () => {
  const { isAuthenticated, loginWithRedirect, logout } = useAuth0();

  return (
    <div>
      {!isAuthenticated && (
        <button
          onClick={() =>
            loginWithRedirect({})
          }
        >
          Log in
        </button>
      )}

      {isAuthenticated && <button onClick={() => logout()}>Log out</button>}
    </div>
  );
};

export default NavBar;
```

Here the component renders two buttons, for logging in and logging out, depending on whether the user is currently authenticated.

Notice the use of `useAuth0` — provided by the wrapper you created in the previous section — which provides the functions needed in order to log in, log out, and determine if the user is logged in through the `isAuthenticated` property.

### Integrate the SDK

In order for the authentication system to work properly, the application components should be wrapped in the `Auth0Provider` component that is provided by the SDK wrapper created earlier in the tutorial. This means that any components inside this wrapper will be able to access the Auth0 SDK client.

Open the `src/index.js` file and replace its contents with the following:

```jsx
// src/index.js

import React from "react";
import ReactDOM from "react-dom";
import App from "./App";
import * as serviceWorker from "./serviceWorker";
import { Auth0Provider } from "./react-auth0-wrapper";
import config from "./auth_config.json";

// A function that routes the user to the right place
// after login
const onRedirectCallback = appState => {
  window.history.replaceState(
    {},
    document.title,
    appState && appState.targetUrl
      ? appState.targetUrl
      : window.location.pathname
  );
};

ReactDOM.render(
  <Auth0Provider
    domain={config.domain}
    client_id={config.clientId}
    redirect_uri={window.location.origin}
    onRedirectCallback={onRedirectCallback}
>
    <App />
  </Auth0Provider>,
  document.getElementById("root")
);

serviceWorker.unregister();
```

Notice that the `App` component is now wrapped in the `Auth0Provider` component, where the details about the Auth0 domain and client ID are specified. The `redirect_uri` prop is also specified here. Doing this here means that you don't need to pass this URI to every call to `loginWithRedirect`, and it keeps the configuration in one place.

Also notice the function `onRedirectCallback`, which tries to route the user to the right place once they have logged in. For example, if the user tries to access a page that requires them to be authenticated, they will be asked to log in. When they return to the application, they will be forwarded to the page they were originally trying to access thanks to this function.

Next, create a new file `auth_config.json` in the `src` folder, and populate it with the following:

```json
{
  "domain": "${account.namespace}",
  "clientId": "${account.clientId}"
}
```

:::note
The values for `domain` and `clientId` should be replaced with those for your own Auth0 app.
:::

Next, open the `App.js` file in the `src` folder, populate it with the following content:

```js
// src/App.js

import React from "react";
import NavBar from "./components/NavBar";
import { useAuth0 } from "./react-auth0-wrapper";

function App() {
  const { loading } = useAuth0();

  if (loading) {
    return (
      <div>Loading...</div>
    );
  }

  return (
    <div className="App">
      <header>
        <NavBar />
      </header>
    </div>
  );
}

export default App;
```

This replaces the default content created by `create-react-app` and simply shows the `NavBar` component you created earlier.

> **Checkpoint**: At this point, you should be able to go through a complete authentication cycle, logging in and loggin out. Start the application from the terminal using `yarn start` and browse to http://localhost:3000 (if the application does not open automatically). From there, clicking the **Log in** button should redirect you to the Auth0 Login Page where you will be given the opportunity to log in.
> Once you are logged in, control returns to your application and you should see that the **Log out** button is now visible. Clicking this should log you out of the application and return you to an unauthenticated state.

## Read the User Profile

When a user is logged in, the associated **user profile** information can be retrieved. Typically this is used to display their name and profile picture.

To display this information to the user, create a new file called `Profile.js` inside the `components` folder and populate it with the following content:

```jsx
// src/components/Profile.js

import React from "react";
import { useAuth0 } from "../react-auth0-wrapper";

const Profile = () => {
  const { loading, user } = useAuth0();

  if (loading || !user) {
    return (
      <div>Loading...</div>
    );
  }

  return (
    <>
      <img src={user.picture} alt="Profile" />

      <h2>{user.name}</h2>
      <p>{user.email}</p>
      <code>{JSON.stringify(user, null, 2)}</code>
    </>
  );
};

export default Profile;
```

Notice here that the `useAuth0` hook is again being used, this time to retrieve the user's profile information (through the `user` property) and a `loading` property that can be used to display some kind of "loading" text or spinner graphic while the user's data is being retrieved.

In the UI for this component, the user's profile picture, name, and email address is being retrieved from the `user` property and displayed on the screen.

To access this page, modify the `App.js` file to include a router so that the profile page may be displayed on the screen. The `App.js` file should now look something like this:

```jsx
// src/App.js

import React from "react";
import NavBar from "./components/NavBar";

// New - import the React Router components, and the Profile page component
import { BrowserRouter, Route, Switch } from "react-router-dom";
import Profile from "./components/Profile";

function App() {
  return (
    <div className="App">
      {/* New - use BrowserRouter to provide access to /profile */}
      <BrowserRouter>
        <header>
          <NavBar />
        </header>
        <Switch>
          <Route path="/" exact />
          <Route path="/profile" component={Profile} />
        </Switch>
      </BrowserRouter>
    </div>
  );
}

export default App;
```

Notice that a `BrowserRouter` component has been included, and that two routes have been defined — one for the home page, and another for the profile page.

To complete this step, open the `Navbar.js` file and modify the navigation bar's UI to include a link to the profile page. In addition, import the `Link` component at the top of the file.

The `NavBar` component should now look something like this:

```jsx
// src/components/NavBar.js
// .. other imports

// NEW - import the Link component
import { Link } from "react-router-dom";

const NavBar = () => {
  const { isAuthenticated, loginWithRedirect, logout } = useAuth0();

  return (
    // .. code removed for brevity

    {isAuthenticated && <button onClick={() => logout()}>Log out</button>}

    {/* NEW - add a link to the home and profile pages */}
    {isAuthenticated && (
      <span>
        <Link to="/">Home</Link>&nbsp;
        <Link to="/profile">Profile</Link>
      </span>
    )}

    //..
  );
};

export default NavBar;
```

> **Checkpoint**: Go ahead and run the project one more time. Now if the user is authenticated and you navigate to the `/profile` page, you will see their profile data. See how this content disappears when you log out.

## Secure the Profile Page

The profile page should be protected in such a way that if the user tries to access it directly without logging in, they will be automatically redirected to Auth0 to log in. Currently the user would just see a blank page.

To fix this, a Higher-Order Component can be created that will wrap any component to check if is authenticated.

Start by creating a new component `components/PrivateRoute.js` that can wrap another component. Populate it with the following content:

```jsx
// src/components/PrivateRoute.js

import React, { useEffect } from "react";
import { Route } from "react-router-dom";
import { useAuth0 } from "../react-auth0-wrapper";

const PrivateRoute = ({ component: Component, path, ...rest }) => {
  const { loading, isAuthenticated, loginWithRedirect } = useAuth0();

  useEffect(() => {
    if (loading || isAuthenticated) {
      return;
    }
    const fn = async () => {
      await loginWithRedirect({
        appState: { targetUrl: path }
      });
    };
    fn();
  }, [loading, isAuthenticated, loginWithRedirect, path]);

  const render = props => isAuthenticated === true ? <Component {...props} /> : null;

  return <Route path={path} render={render} {...rest} />;
};

export default PrivateRoute;
```

This component takes another component as one of its arguments. It makes use of the [`useEffect` hook](https://reactjs.org/docs/hooks-effect.html) to redirect to the user to the login page if they are not yet authenticated.

If the user is authenticated, the redirect will not take place and the component that was specified as the argument will be rendered instead. In this way, components that require the user to be logged in can be protected simply by wrapping the component using `PrivateRoute`.

### Protect application routes

With the `PrivateRoute` component in place, the application router can now be modified to secure the `/profile` route, ensuring that users must log into the application in order to see it.

Open `App.js` once again, import the `PrivateRoute` component, and update the router so that the `Profile` component is wrapped by the `PrivateRoute` component:

```jsx
// src/App.js

// .. other imports removed for brevity

// NEW - import the PrivateRoute component
import PrivateRoute from "./components/PrivateRoute";

function App() {
  return (
    {/* other components removed for brevity */}

    {/* NEW - Modify the /profile route to use PrivateRoute instead of Route */}
    <PrivateRoute path="/profile" component={Profile} />
  );
}

export default App;
```

> **Checkpoint**: Run the project again. Now if the user is not authenticated and you navigate to the `/profile` page through the URL bar in the browser, you will be sent through the authentication flow, and will see the Profile page upon your return.
