# 日常小島

A responsive personal planner prototype with Firebase Google sign-in.

## Enable Firebase Google sign-in

1. Create or select a project in the [Firebase Console](https://console.firebase.google.com/).
2. Add a Web app and copy its configuration.
3. In **Authentication → Sign-in method**, enable **Google**.
4. In **Authentication → Settings → Authorized domains**, add the domain where this site will run (for local testing, include `localhost`).
5. Copy `firebase-config.js.example` to `firebase-config.js`, then replace all placeholder values with the Web app configuration.
6. Serve this folder from a local web server or deploy it to a static host. ES modules and Firebase Auth do not work reliably when opened as a `file://` URL.

The app is hidden behind the sign-in gate unless Firebase Auth reports a signed-in user. `firebase-config.js` is intentionally ignored by Git. For production hosting, provide the public Firebase web configuration through your deployment process; Firebase web config is not a server secret, but API key restrictions and Firebase Security Rules should still be configured appropriately.
