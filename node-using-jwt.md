# Node Security project using JWT

## Implementing JWT authentication in a Node.js application requires several steps. I'll guide you through the process and provide a simple implementation:

### Folder & File Structure:
1. `/models`: Contains the user model.
2. `/routes`: Contains the API endpoints.
3. `/middleware`: Contains JWT middleware to protect routes.
4. `app.js`: Entry point for our app.

### 1. Setting up the Environment:

First, we'll need to set up a new project:

```bash
mkdir jwt-auth-api
cd jwt-auth-api
npm init -y
```

Now, we'll install the necessary packages:

```bash
npm install express mongoose jsonwebtoken bcryptjs
```

### 2. User Model (models/userModel.js):

We will define a simple user schema with mongoose. Passwords are hashed before saving.

```javascript
import mongoose from 'mongoose';
import bcrypt from 'bcryptjs';

const userSchema = new mongoose.Schema({
    username: String,
    password: String
});

userSchema.pre('save', async function(next) {
    if (this.isModified('password')) {
        this.password = await bcrypt.hash(this.password, 8);
    }
    next();
});

export const User = mongoose.model('User', userSchema);
```

### 3. JWT Middleware (middleware/auth.js):

This middleware will be used to protect routes that require authentication.

```javascript
import jwt from 'jsonwebtoken';

const auth = async (req, res, next) => {
    try {
        const token = req.header('Authorization').replace('Bearer ', '');
        const decoded = jwt.verify(token, 'SECRET_KEY'); 
        // Replace 'SECRET_KEY' with your secret key or an environment variable.
        req.user = decoded;
        next();
    } catch (e) {
        res.status(401).send({ error: 'Please authenticate.' });
    }
};

export default auth;
```

### 4. Routes (routes/userRoutes.js):

We'll define endpoints for user registration, login, and a protected route.

```javascript
import express from 'express';
import { User } from '../models/userModel.js';
import bcrypt from 'bcryptjs';
import jwt from 'jsonwebtoken';
import auth from '../middleware/auth.js';

const router = express.Router();

router.post('/register', async (req, res) => {
    try {
        const { username, password } = req.body;
        const user = new User({ username, password });
        await user.save();
        res.status(201).send(user);
    } catch (error) {
        res.status(500).send(error.message);
    }
});

router.post('/login', async (req, res) => {
    try {
        const { username, password } = req.body;
        const user = await User.findOne({ username });
        
        if (!user || !await bcrypt.compare(password, user.password)) {
            return res.status(400).send({ error: 'Invalid login credentials.' });
        }

        const token = jwt.sign({ _id: user._id.toString() }, 'SECRET_KEY'); 
        // 'SECRET_KEY' should be stored securely.
        res.send({ token });
    } catch (error) {
        res.status(500).send(error.message);
    }
});

router.get('/profile', auth, (req, res) => {
    res.send(req.user);
});

export default router;
```

### 5. Main App (app.js):

```javascript
import express from 'express';
import mongoose from 'mongoose';
import userRoutes from './routes/userRoutes.js';

const app = express();

mongoose.connect('mongodb://localhost:27017/jwtAuthDB', {
    useNewUrlParser: true,
    useUnifiedTopology: true,
    useFindAndModify: false,
});

app.use(express.json());
app.use('/api/users', userRoutes);

const PORT = 3000;
app.listen(PORT, () => {
    console.log(`Server is running on port ${PORT}`);
});
```

### Explanation:

1. **User Model**: We have a simple user schema with a pre-save hook to hash the password before storing it.
2. **JWT Middleware**: This checks for a valid JWT token in the request headers and verifies it. If verified, the request is allowed to proceed.
3. **Routes**:
   - **/register**: Allows a user to register.
   - **/login**: Allows a user to login and returns a JWT token upon successful authentication.
   - **/profile**: A protected route which returns user data if a valid JWT token is provided.
4. **Main App**: Sets up Express and mongoose, and mounts the user routes.

### Use-cases:

1. **User Registration**: Send a POST request to `/api/users/register` with a `username` and `password` in the body.
2. **User Login**: Send a POST request to `/api/users/login` with correct `username` and `password` to receive a JWT token.
3. **Accessing Protected Route**: To access `/api/users/profile`, send a GET request with the JWT token in the header (`Authorization: Bearer YOUR_JWT_TOKEN`).

Note: For production, you should:
- Store the JWT secret securely (using environment variables, or tools like Vault).
- Handle JWT token expiration.
- Implement logout (by blacklisting tokens or using a token store).
- Use HTTPS to secure data transmission.