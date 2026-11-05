const express = require('express');
const session = require('express-session');

const app = express();
const PORT = 3000;

// Middleware to parse incoming request bodies (for login form submission)
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Configure session middleware
app.use(session({
    secret: 'your-secure-secret-key', // Secret key to sign the session ID cookie
    resave: false,                   // Don't save session if unmodified
    saveUninitialized: false,        // Don't create session until something is stored
    cookie: { 
        maxAge: 600000,              // Cookie expiration time in milliseconds (10 minutes)
        httpOnly: true               // Prevents client-side JS from accessing the cookie
    }
}));

// Mock user database for authentication
const MOCK_USER = {
    username: 'student',
    password: 'password123'
};

// Custom Authentication Middleware to protect routes
const requireAuth = (req, res, next) => {
    if (req.session && req.session.username) {
        next(); // User is authenticated, proceed to the route handler
    } else {
        res.status(401).send('<h1>401 Unauthorized</h1><p>Please <a href="/login">login</a> to view this page.</p>');
    }
};

// 1. PUBLIC ROUTE: Home / Landing Page
app.get('/', (req, res) => {
    res.send(`
        <h1>Welcome to the ExpressJS Session Lab</h1>
        <p><a href="/login">Go to Login</a></p>
        <p><a href="/dashboard">Go to Dashboard (Protected)</a></p>
    `);
});

// 2. AUTHENTICATION: GET Login Page
app.get('/login', (req, res) => {
    // If already logged in, redirect straight to dashboard
    if (req.session.username) {
        return res.redirect('/dashboard');
    }
    
    // Render a simple HTML login form
    res.send(`
        <h1>Login</h1>
        <form action="/login" method="POST">
            <label>Username: <input type="text" name="username" required /></label><br/><br/>
            <label>Password: <input type="password" name="password" required /></label><br/><br/>
            <button type="submit">Submit</button>
        </form>
    `);
});

// 3. AUTHENTICATION: POST Login Submission
app.post('/login', (req, res) => {
    const { username, password } = req.body;

    // Verify credentials
    if (username === MOCK_USER.username && password === MOCK_USER.password) {
        // --- PART (a) & (b): Setting Session Variables ---
        req.session.username = username;
        req.session.visitCount = 1; // Initialize visit counter upon first login
        
        res.redirect('/dashboard');
    } else {
        res.send('<h3>Invalid credentials. <a href="/login">Try again</a></h3>');
    }
});

// 4. PROTECTED ROUTE & DASHBOARD: Displays & updates session data
app.get('/dashboard', requireAuth, (req, res) => {
    // --- PART (a): Reading and Updating Session Data ---
    req.session.visitCount = (req.session.visitCount || 0) + 1;

    res.send(`
        <h1>Dashboard</h1>
        <p><strong>Status:</strong> Successfully Authenticated</p>
        <p><strong>Username (Session Data):</strong> ${req.session.username}</p>
        <p><strong>Visit Count (Session Data):</strong> ${req.session.visitCount}</p>
        <hr />
        <p><a href="/logout">Logout</a></p>
    `);
});

// 5. AUTHENTICATION & DESTRUCTION: Logout route
app.get('/logout', (req, res) => {
    // --- PART (a) & (b): Destroying Session Data ---
    req.session.destroy((err) => {
        if (err) {
            return res.send('Error logging out.');
        }
        res.clearCookie('connect.sid'); // Clear the session ID cookie from client browser
        res.send('<h3>Logged out successfully! <a href="/">Go Home</a></h3>');
    });
});

// Start the server
app.listen(PORT, () => {
    console.log(Server is running at http://localhost:${PORT});
});
