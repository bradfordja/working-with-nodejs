# Node Security project using OAuth

## Implementing OAuth authentication in a Node.js application often involves integration with third-party providers, such as Google or Facebook, as they provide OAuth authentication mechanisms.

For this example, let's use the **Passport.js** library with **OAuth2.0** for Google Authentication.

### Setup:

1. Initialize a new Node.js application with `npm init`.
2. Install the necessary packages:
```bash
npm install express passport passport-google-oauth20 express-session
```

### Code:

1. **Setup Passport with Google OAuth Strategy**:

```javascript
// config/passport-setup.js

const passport = require('passport');
const GoogleStrategy = require('passport-google-oauth20');

passport.use(
    new GoogleStrategy({
        // options for the google strategy
        clientID: YOUR_GOOGLE_CLIENT_ID,
        clientSecret: YOUR_GOOGLE_CLIENT_SECRET,
        callbackURL: '/auth/google/redirect'
    }, (accessToken, refreshToken, profile, done) => {
        // passport callback function
        // Usually, here you'll check if user exists in DB and if not, create one, then call done
        // For now, we're just returning the profile
        return done(null, profile);
    })
);
```

2. **Routes**:
```javascript
// routes/authRoutes.js

const router = require('express').Router();
const passport = require('passport');

router.get('/login', (req, res) => {
    res.send('Login with Google');
});

router.get('/google', passport.authenticate('google', {
    scope: ['profile']
}));

router.get('/google/redirect', passport.authenticate('google'), (req, res) => {
    res.send(req.user);  // This will display user info after successful authentication
});

router.get('/logout', (req, res) => {
    req.logout();
    res.send('Logged out');
});

module.exports = router;
```

3. **App Entry Point**:
```javascript
// app.js

const express = require('express');
const authRoutes = require('./routes/authRoutes');
const passportSetup = require('./config/passport-setup');

const app = express();

// For maintaining user sessions
app.use(require('express-session')({
    secret: 'your_secret_key',  // Again, it's better to store secrets as environment variables
    resave: false,
    saveUninitialized: false
}));

// Initialize Passport
app.use(passport.initialize());
app.use(passport.session());

app.use('/auth', authRoutes);

app.listen(3000, () => {
    console.log("Server started on http://localhost:3000");
});
```

### Explanation:

1. **Passport Setup**: We use the Google OAuth2.0 strategy from Passport. You will need to get your `clientID` and `clientSecret` from the Google Developer Console by creating a new project and setting up OAuth2.0 credentials.

2. **Routes**:
    - **Login**: Basic route prompting the user to login with Google.
    - **Google**: This route redirects users to Google's consent screen.
    - **Google Redirect**: After consent, Google redirects here with an OAuth code, which Passport exchanges for user info.
    - **Logout**: Ends the user's session.

3. **App Entry Point**: This sets up the server, integrates Passport, and establishes our routes.

### Use-case:

1. A user visits the `/auth/login` route and sees the prompt to login with Google.
2. The user clicks and gets redirected to Google's consent screen through the `/auth/google` route.
3. After granting permission, Google redirects the user to `/auth/google/redirect` with an OAuth code.
4. Passport exchanges this code for user info and sets up a user session.
5. The user can now access protected resources. To log out, they visit the `/auth/logout` route.

For a production system, you would also handle storing users in a database, error handling, token refreshing, ensuring security, and more. This is a simplified example to demonstrate the core process.