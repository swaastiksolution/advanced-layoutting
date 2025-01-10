# advanced-layoutting

Dynamic rendering involves dynamically altering components, layouts, or even the content shown to users based on external conditions such as user roles, feature flags, API data, or device type.

Here’s how you can modify and extend the setup to enable dynamic rendering:

## Adding Dynamic Rendering to the Example
### 1. Render Layouts Dynamically Based on User Roles
Let’s enhance the config file to include role-based layouts. For example, show different components to admins and users.

#### Updated Config File

```
const config = {
  layouts: {
    default: {
      header: "Header",
      footer: "Footer",
      mainContent: "Main",
    },
    sidebarLayout: {
      header: "Header",
      sidebar: "Sidebar",
      mainContent: "Main",
    },
  },
  routes: [
    {
      path: "/",
      layout: "default",
      component: "HomePage",
      roles: ["user", "admin"], // Accessible to both user and admin
    },
    {
      path: "/about",
      layout: "default",
      component: "AboutPage",
      roles: ["user", "admin"], // Accessible to both
    },
    {
      path: "/dashboard",
      layout: "sidebarLayout",
      component: "DashboardPage",
      roles: ["admin"], // Accessible only to admins
    },
  ],
};

export default config;
```
----

## 2. Pass User Role Dynamically
Suppose you fetch the user role from an API or authentication context. Modify the App component to dynamically filter routes based on roles.
### Updated App Component
```
import React from "react";
import { BrowserRouter as Router, Routes, Route, Navigate } from "react-router-dom";
import LayoutRenderer from "./LayoutRenderer";
import config from "./config";
import { HomePage, AboutPage, DashboardPage } from "./pages";

const COMPONENT_MAP = {
  HomePage,
  AboutPage,
  DashboardPage,
};

function App() {
  // Example: Assume the user role comes from an authentication context or API
  const userRole = "user"; // Change this to "admin" to test admin-only routes

  return (
    <Router>
      <Routes>
        {config.routes.map(({ path, layout, component, roles }, index) => {
          const Component = COMPONENT_MAP[component];
          const Layout = config.layouts[layout];

          // Check if the user's role matches the route's allowed roles
          if (!roles.includes(userRole)) {
            return null; // Skip this route if the user doesn't have permission
          }

          return (
            <Route
              key={index}
              path={path}
              element={
                <LayoutRenderer layout={Layout}>
                  <Component />
                </LayoutRenderer>
              }
            /
          );
        })}
        {/* Fallback route for unauthorized users */}
        <Route path="*" element={<Navigate to="/" />} />
      </Routes>
    </Router>
  );
}

export default App;
```
----

## 3. Dynamic Rendering Based on Feature Flags
You can dynamically control which features or pages are enabled based on feature flags. Add a flag system to the config.

### Example Config with Feature Flags
```
const config = {
  featureFlags: {
    enableAboutPage: true,
    enableDashboard: false,
  },
  routes: [
    {
      path: "/",
      layout: "default",
      component: "HomePage",
    },
    {
      path: "/about",
      layout: "default",
      component: "AboutPage",
      featureFlag: "enableAboutPage", // Tied to a feature flag
    },
    {
      path: "/dashboard",
      layout: "sidebarLayout",
      component: "DashboardPage",
      featureFlag: "enableDashboard", // Tied to a feature flag
    },
  ],
};

export default config;

```
### Updated App to Handle Feature Flags
```
function App() {
  const featureFlags = config.featureFlags;

  return (
    <Router>
      <Routes>
        {config.routes.map(({ path, layout, component, featureFlag }, index) => {
          const Component = COMPONENT_MAP[component];
          const Layout = config.layouts[layout];

          // Check if the feature flag is enabled (if defined)
          if (featureFlag && !featureFlags[featureFlag]) {
            return null; // Skip this route if the feature is disabled
          }

          return (
            <Route
              key={index}
              path={path}
              element={
                <LayoutRenderer layout={Layout}>
                  <Component />
                </LayoutRenderer>
              }
            />
          );
        })}
        <Route path="*" element={<Navigate to="/" />} />
      </Routes>
    </Router>
  );
}
```

---

## 4. Fetch Layout/Route Configurations Dynamically from an API
You can fetch the configuration file dynamically from an API to make it even more flexible. For example:

### Fetch Config in App

```
import { useEffect, useState } from "react";

function App() {
  const [config, setConfig] = useState(null);

  useEffect(() => {
    // Simulate fetching config from an API
    fetch("/api/config")
      .then((res) => res.json())
      .then((data) => setConfig(data));
  }, []);

  if (!config) {
    return <div>Loading...</div>; // Show a loading screen while fetching config
  }

  return (
    <Router>
      <Routes>
        {config.routes.map(({ path, layout, component }, index) => {
          const Component = COMPONENT_MAP[component];
          const Layout = config.layouts[layout];

          return (
            <Route
              key={index}
              path={path}
              element={
                <LayoutRenderer layout={Layout}>
                  <Component />
                </LayoutRenderer>
              }
            />
          );
        })}
        <Route path="*" element={<Navigate to="/" />} />
      </Routes>
    </Router>
  );
}

export default App;
```

----

## 5. Device-Specific Rendering
You can dynamically render layouts/components based on the user’s device type (mobile, tablet, desktop). Use libraries like react-device-detect.

### Install react-device-detect
```
npm install react-device-detect
```

### Add Device Detection

```
import { isMobile, isTablet } from "react-device-detect";

function LayoutRenderer({ layout, children }) {
  const renderSidebar = layout.sidebar && !isMobile; // Hide sidebar on mobile devices

  return (
    <>
      {layout.header && <Header />}
      <div style={{ display: "flex" }}>
        {renderSidebar && <Sidebar />}
        <Main>{children}</Main>
      </div>
      {layout.footer && <Footer />}
    </>
  );
}
```
----

## Benefits of Dynamic Rendering
- Scalability: Easily handle complex use cases (roles, feature flags, device types).
- User Personalization: Serve tailored content to users based on role, location, or device.
- Better Resource Utilization: Avoid rendering components or routes unnecessarily.
- Centralized Control: Manage layouts, features, and pages without modifying code.
