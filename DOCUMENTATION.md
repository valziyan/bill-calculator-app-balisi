# Electric Bill Calculator - Supabase Implementation Documentation

## Table of Contents
1. [Introduction to Supabase](#introduction-to-supabase)
2. [Project Structure](#project-structure)
3. [Supabase Setup](#supabase-setup)
4. [Authentication System](#authentication-system)
5. [Database Operations](#database-operations)
6. [JavaScript Functions Explained](#javascript-functions-explained)
7. [Page Protection System](#page-protection-system)
8. [Complete Flow Examples](#complete-flow-examples)
9. [Best Practices](#best-practices)

---

## Introduction to Supabase

### What is Supabase?
Supabase is an open-source Firebase alternative that provides:
- **Authentication**: User registration, login, and session management
- **Database**: PostgreSQL database with real-time capabilities
- **Storage**: File storage
- **Row Level Security (RLS)**: Security policies to protect user data

### Why Use Supabase?
- **Easy to use**: Simple JavaScript API
- **Secure**: Built-in authentication and security
- **Free tier**: Great for learning and small projects
- **Real-time**: Can update data in real-time (not used in this project)

---

## Project Structure

```
bill-calculator-app-balisi/
├── index.html              # Homepage
├── login.html              # Login page
├── register.html           # Registration page
├── user_dashboard.html     # Dashboard (protected)
├── calculator_page.html    # Calculator (protected)
├── results_page.html       # Results display (protected)
├── profile_page.html       # User profile (protected)
├── css/
│   └── style.css          # Styling
├── js/
│   ├── supabase-config.js # Supabase client configuration
│   └── auth.js            # Authentication helper functions
├── schema.sql             # Database schema
└── SUPABASE_SETUP.md      # Setup instructions
```

---

## Supabase Setup

### 1. Configuration File (`js/supabase-config.js`)

This file initializes the Supabase client and provides authentication helper functions.

```javascript
// Supabase Configuration
const supabaseUrl = "https://nrdadjisfwvgjgyshdqc.supabase.co";
const supabaseKey = "YOUR_SUPABASE_ANON_KEY";

// Create Supabase client (supabase is available globally from CDN)
const { createClient } = supabase;
const supabaseClient = createClient(supabaseUrl, supabaseKey);
```

**Explanation:**
- `supabaseUrl`: Your Supabase project URL
- `supabaseKey`: Your public/anon API key (safe to expose in client-side code)
- `createClient()`: Creates a connection to your Supabase project
- `supabaseClient`: The client object used for all Supabase operations

**Why anon key?**
- The anon key is safe to use in client-side JavaScript
- Row Level Security (RLS) policies protect your data
- Users can only access their own data

### 2. Loading Supabase in HTML

In each HTML page that uses Supabase, we load it via CDN:

```html
<!-- Supabase CDN - Must be loaded first -->
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
<script src="js/supabase-config.js"></script>
<script src="js/auth.js"></script>
```

**Order matters!** The Supabase CDN must load before our config file.

---

## Authentication System

### How Authentication Works

1. **User Registration**: User creates an account with email/password
2. **Email Verification**: Supabase sends verification email (optional)
3. **User Login**: User signs in with email/password
4. **Session Creation**: Supabase creates a session (stored in browser)
5. **Session Check**: Pages check if user is logged in
6. **Logout**: User signs out, session is destroyed

### Authentication Functions (`js/supabase-config.js`)

#### 1. `checkAuth()` - Check if User is Logged In

```javascript
async function checkAuth() {
    const {
        data: { session },
        error,
    } = await supabaseClient.auth.getSession();
    if (error) {
        console.error("Error checking auth:", error);
        return null;
    }
    return session;
}
```

**What it does:**
- Checks if there's an active session in the browser
- Returns the session object if logged in, or `null` if not

**Usage:**
```javascript
const session = await checkAuth();
if (session) {
    console.log("User is logged in!");
} else {
    console.log("User is not logged in");
}
```

**Returns:**
- `session` object if logged in (contains user info)
- `null` if not logged in

#### 2. `getCurrentUser()` - Get Current User Information

```javascript
async function getCurrentUser() {
    const {
        data: { user },
        error,
    } = await supabaseClient.auth.getUser();
    if (error) {
        console.error("Error getting user:", error);
        return null;
    }
    return user;
}
```

**What it does:**
- Gets the current logged-in user's information
- Returns user object with email, id, metadata, etc.

**Usage:**
```javascript
const user = await getCurrentUser();
if (user) {
    console.log("User ID:", user.id);
    console.log("User Email:", user.email);
    console.log("User Name:", user.user_metadata.full_name);
}
```

**Returns:**
- `user` object with properties:
  - `id`: Unique user ID (UUID)
  - `email`: User's email address
  - `user_metadata`: Custom data (like full_name)
  - `created_at`: Account creation date

#### 3. `signOut()` - Logout User

```javascript
async function signOut() {
    const { error } = await supabaseClient.auth.signOut();
    if (error) {
        console.error("Error signing out:", error);
        return false;
    }
    return true;
}
```

**What it does:**
- Signs out the current user
- Destroys the session
- User must login again to access protected pages

**Usage:**
```javascript
const success = await signOut();
if (success) {
    window.location.href = "login.html";
}
```

**Returns:**
- `true` if logout successful
- `false` if error occurred

### Registration Implementation (`register.html`)

```javascript
async function handleRegister() {
    // Get form values
    const name = document.getElementById("reg_name").value;
    const email = document.getElementById("reg_email").value;
    const password = document.getElementById("reg_password").value;
    const confirmPassword = document.getElementById("reg_confirm_password").value;

    // Validation
    if (password !== confirmPassword) {
        alert("Passwords do not match!");
        return;
    }

    // Register user with Supabase
    const { data, error } = await supabaseClient.auth.signUp({
        email: email,
        password: password,
        options: {
            data: {
                full_name: name  // Store name in user metadata
            }
        }
    });

    if (error) {
        alert("Registration failed: " + error.message);
        return;
    }

    alert("Registration successful!");
    window.location.href = "login.html";
}
```

**Step-by-step explanation:**
1. Get form input values
2. Validate passwords match
3. Call `supabaseClient.auth.signUp()` with email, password, and metadata
4. Check for errors
5. Redirect to login page on success

**Key points:**
- `signUp()` creates a new user account
- `options.data` stores custom information (full_name)
- Email verification may be required (check Supabase settings)

### Login Implementation (`login.html`)

```javascript
async function handleLogin() {
    const email = document.getElementById("email").value;
    const password = document.getElementById("password").value;

    // Sign in with Supabase
    const { data, error } = await supabaseClient.auth.signInWithPassword({
        email: email,
        password: password
    });

    if (error) {
        alert("Login failed: " + error.message);
        return;
    }

    // Success - redirect to dashboard
    window.location.href = "user_dashboard.html";
}
```

**Step-by-step explanation:**
1. Get email and password from form
2. Call `supabaseClient.auth.signInWithPassword()`
3. Check for errors
4. Redirect to dashboard on success

**Key points:**
- Creates a session that persists in the browser
- Session is automatically checked on page loads
- Invalid credentials return an error

---

## Database Operations

### Database Schema

The `calculations` table stores electricity bill calculations:

```sql
CREATE TABLE calculations (
    id BIGSERIAL PRIMARY KEY,              -- Auto-incrementing ID
    user_id UUID NOT NULL,                 -- Links to auth.users
    month TEXT NOT NULL,                   -- Month name
    power_consumption DECIMAL(10, 2),      -- kWh consumed
    cost_per_kwh DECIMAL(10, 2),          -- Cost per unit
    result DECIMAL(10, 2),                 -- Calculated total
    created_at TIMESTAMP DEFAULT NOW()    -- When created
);
```

### Row Level Security (RLS)

RLS ensures users can only access their own data:

```sql
-- Users can only INSERT their own calculations
CREATE POLICY "Users can insert their own calculations"
    ON calculations FOR INSERT
    TO authenticated
    WITH CHECK (auth.uid() = user_id);

-- Users can only SELECT their own calculations
CREATE POLICY "Users can view their own calculations"
    ON calculations FOR SELECT
    TO authenticated
    USING (auth.uid() = user_id);
```

**How it works:**
- `auth.uid()` returns the current user's ID
- Policies check if `user_id` matches `auth.uid()`
- Users can only see/modify their own records

### Saving Calculations (`calculator_page.html`)

```javascript
// Get current user
const user = await getCurrentUser();
if (!user) {
    alert("You must be logged in!");
    return;
}

// Calculate result
let result_value = parseFloat(cost_per_kwh.value) * 
                   parseFloat(power_consumption.value);

// Save to Supabase
const { data, error } = await supabaseClient
    .from("calculations")                    // Table name
    .insert([                                // Insert operation
        {
            user_id: user.id,               // Link to user
            month: month.value,
            power_consumption: parseFloat(power_consumption.value),
            cost_per_kwh: parseFloat(cost_per_kwh.value),
            result: result_value
        }
    ])
    .select();                               // Return inserted data

if (error) {
    console.error("Error saving:", error);
    alert("Failed to save: " + error.message);
} else {
    console.log("Saved successfully:", data);
}
```

**Step-by-step explanation:**
1. Get current user (must be logged in)
2. Calculate the bill amount
3. Use `.from("calculations")` to specify the table
4. Use `.insert([...])` to add a new row
5. Use `.select()` to return the inserted data
6. Handle errors or success

**Supabase Query Builder:**
- `.from()`: Select table
- `.insert()`: Add new rows
- `.select()`: Return data
- `.eq()`: Filter (equals)
- `.order()`: Sort results

### Fetching Calculations (`results_page.html`)

```javascript
async function loadCalculations() {
    // Get current user
    const user = await getCurrentUser();
    if (!user) return;

    // Fetch calculations from Supabase
    const { data, error } = await supabaseClient
        .from("calculations")                    // Table name
        .select("*")                             // Select all columns
        .eq("user_id", user.id)                  // Filter: user_id = current user
        .order("created_at", { ascending: false }); // Sort: newest first

    if (error) {
        console.error("Error loading:", error);
        return;
    }

    // Display data in table
    data.forEach(calc => {
        // Create table row for each calculation
        // ...
    });
}
```

**Step-by-step explanation:**
1. Get current user
2. Use `.from("calculations")` to select table
3. Use `.select("*")` to get all columns
4. Use `.eq("user_id", user.id)` to filter by user
5. Use `.order()` to sort by date (newest first)
6. Loop through results and display

**Query methods:**
- `.select("*")`: Get all columns
- `.select("column1, column2")`: Get specific columns
- `.eq("column", value)`: WHERE column = value
- `.order("column", { ascending: false })`: ORDER BY column DESC

---

## JavaScript Functions Explained

### Helper Functions (`js/auth.js`)

#### 1. `requireAuth()` - Require User to be Logged In

```javascript
async function requireAuth() {
    const session = await checkAuth();
    if (!session) {
        window.location.href = "login.html";
        return false;
    }
    return true;
}
```

**What it does:**
- Checks if user is logged in
- Redirects to login if not authenticated
- Returns `true` if authenticated, `false` if not

**Usage:**
```javascript
const isAuthenticated = await requireAuth();
if (!isAuthenticated) {
    // User was redirected, code below won't run
    return;
}
// User is authenticated, continue...
```

#### 2. `initAuth()` - Initialize Authentication Check

```javascript
async function initAuth() {
    const session = await checkAuth();
    if (!session) {
        window.location.href = "login.html";
    }
    return session;
}
```

**What it does:**
- Checks authentication on page load
- Redirects if not logged in
- Returns session if authenticated

**Usage:**
```javascript
document.addEventListener("DOMContentLoaded", async () => {
    const session = await initAuth();
    if (session) {
        // User is logged in, load page content
        loadUserData();
    }
});
```

#### 3. `redirectIfAuthenticated()` - Redirect if Already Logged In

```javascript
async function redirectIfAuthenticated() {
    const session = await checkAuth();
    if (session) {
        window.location.href = "user_dashboard.html";
    }
}
```

**What it does:**
- Checks if user is already logged in
- Redirects to dashboard if authenticated
- Used on login/register pages

**Usage:**
```javascript
// On login.html or register.html
document.addEventListener("DOMContentLoaded", async () => {
    await redirectIfAuthenticated();
    // If user is logged in, they'll be redirected
    // Otherwise, show login/register form
});
```

---

## Page Protection System

### How Page Protection Works

Protected pages check authentication on load and redirect if not logged in.

### Example: Protected Page (`calculator_page.html`)

```html
<!-- Supabase scripts loaded first -->
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
<script src="js/supabase-config.js"></script>
<script src="js/auth.js"></script>

<script>
    // Check authentication when page loads
    document.addEventListener("DOMContentLoaded", async () => {
        await initAuth();  // Redirects to login if not authenticated
    });
</script>
```

**Flow:**
1. Page loads
2. `DOMContentLoaded` event fires
3. `initAuth()` checks if user is logged in
4. If not logged in → redirect to `login.html`
5. If logged in → page continues loading

### Protected Pages in This Project

- `user_dashboard.html` - Requires login
- `calculator_page.html` - Requires login
- `results_page.html` - Requires login
- `profile_page.html` - Requires login

### Public Pages

- `index.html` - Homepage (public)
- `login.html` - Login page (public, but redirects if already logged in)
- `register.html` - Registration page (public, but redirects if already logged in)

---

## Complete Flow Examples

### Example 1: User Registration Flow

```
1. User visits register.html
   └─> redirectIfAuthenticated() checks if already logged in
       └─> If logged in → redirect to dashboard
       └─> If not → show registration form

2. User fills form and clicks "Register"
   └─> handleRegister() function runs
       └─> Validates passwords match
       └─> Calls supabaseClient.auth.signUp()
           └─> Supabase creates user account
           └─> Stores user metadata (full_name)
           └─> Sends verification email (if enabled)
       └─> Shows success message
       └─> Redirects to login.html

3. User logs in
   └─> handleLogin() function runs
       └─> Calls supabaseClient.auth.signInWithPassword()
           └─> Supabase validates credentials
           └─> Creates session (stored in browser)
       └─> Redirects to user_dashboard.html
```

### Example 2: Calculating and Saving Bill

```
1. User visits calculator_page.html
   └─> initAuth() checks authentication
       └─> If not logged in → redirect to login.html
       └─> If logged in → show calculator form

2. User fills form and clicks "Calculate"
   └─> btn_calculate click event fires
       └─> Validates form inputs
       └─> getCurrentUser() gets user info
       └─> Calculates: result = power × cost_per_kwh
       └─> Displays result on page
       └─> Saves to database:
           └─> supabaseClient.from("calculations").insert([...])
               └─> RLS policy checks: auth.uid() = user_id
               └─> If valid → insert into database
               └─> Returns inserted data
       └─> Adds row to table using insertRow()
       └─> Clears form inputs
```

### Example 3: Viewing Results

```
1. User visits results_page.html
   └─> initAuth() checks authentication
       └─> If not logged in → redirect to login.html
       └─> If logged in → continue

2. Page loads calculations
   └─> loadCalculations() function runs
       └─> getCurrentUser() gets user ID
       └─> Fetches from database:
           └─> supabaseClient.from("calculations")
               .select("*")
               .eq("user_id", user.id)
               .order("created_at", { ascending: false })
               └─> RLS policy checks: auth.uid() = user_id
               └─> Returns only user's calculations
       └─> Loops through results
       └─> Creates table rows using insertRow()
       └─> Displays in table
```

---

## Best Practices

### 1. Always Check Authentication

```javascript
// ✅ Good: Check before database operations
const user = await getCurrentUser();
if (!user) {
    alert("Please login first!");
    return;
}

// ❌ Bad: Don't assume user is logged in
const { data } = await supabaseClient.from("calculations").select();
```

### 2. Handle Errors Properly

```javascript
// ✅ Good: Check for errors
const { data, error } = await supabaseClient.from("calculations").select();
if (error) {
    console.error("Error:", error);
    alert("Something went wrong: " + error.message);
    return;
}

// ❌ Bad: Ignore errors
const { data } = await supabaseClient.from("calculations").select();
```

### 3. Use Async/Await Correctly

```javascript
// ✅ Good: Use async/await
async function loadData() {
    const user = await getCurrentUser();
    const { data } = await supabaseClient.from("calculations").select();
}

// ❌ Bad: Don't forget await
async function loadData() {
    const user = getCurrentUser();  // Missing await!
    const { data } = supabaseClient.from("calculations").select();  // Missing await!
}
```

### 4. Validate User Input

```javascript
// ✅ Good: Validate before saving
if (!month.value || !power_consumption.value) {
    alert("Please fill in all fields!");
    return;
}

// ❌ Bad: Save without validation
await supabaseClient.from("calculations").insert([...]);
```

### 5. Use Row Level Security

```javascript
// ✅ Good: RLS handles security automatically
// Just filter by user_id, RLS ensures users only see their data
const { data } = await supabaseClient
    .from("calculations")
    .select("*")
    .eq("user_id", user.id);

// ❌ Bad: Don't bypass RLS or trust client-side only
```

---

## Common Patterns

### Pattern 1: Protected Page Template

```html
<!DOCTYPE html>
<html>
<head>
    <!-- ... -->
</head>
<body>
    <!-- Page content -->
    
    <!-- Supabase scripts -->
    <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
    <script src="js/supabase-config.js"></script>
    <script src="js/auth.js"></script>
    
    <script>
        // Check auth on page load
        document.addEventListener("DOMContentLoaded", async () => {
            const session = await initAuth();
            if (session) {
                // User is authenticated, load page data
                loadPageData();
            }
        });
        
        async function loadPageData() {
            // Your page-specific code here
        }
    </script>
</body>
</html>
```

### Pattern 2: Save to Database

```javascript
async function saveData() {
    // 1. Get user
    const user = await getCurrentUser();
    if (!user) return;
    
    // 2. Prepare data
    const dataToSave = {
        user_id: user.id,
        // ... other fields
    };
    
    // 3. Save to database
    const { data, error } = await supabaseClient
        .from("table_name")
        .insert([dataToSave])
        .select();
    
    // 4. Handle result
    if (error) {
        console.error("Error:", error);
        return;
    }
    
    console.log("Saved:", data);
}
```

### Pattern 3: Load from Database

```javascript
async function loadData() {
    // 1. Get user
    const user = await getCurrentUser();
    if (!user) return;
    
    // 2. Fetch data
    const { data, error } = await supabaseClient
        .from("table_name")
        .select("*")
        .eq("user_id", user.id)
        .order("created_at", { ascending: false });
    
    // 3. Handle result
    if (error) {
        console.error("Error:", error);
        return;
    }
    
    // 4. Display data
    data.forEach(item => {
        // Display each item
    });
}
```

---

## Summary

### Key Concepts

1. **Supabase Client**: Connection to your Supabase project
2. **Authentication**: User login/logout/session management
3. **Database Operations**: Insert, select, update, delete
4. **Row Level Security**: Automatic data protection
5. **Page Protection**: Redirect unauthorized users

### Key Functions

- `checkAuth()`: Check if user is logged in
- `getCurrentUser()`: Get current user info
- `signOut()`: Logout user
- `initAuth()`: Protect page on load
- `requireAuth()`: Require authentication
- `redirectIfAuthenticated()`: Redirect if already logged in

### Database Operations

- `.from("table")`: Select table
- `.insert([data])`: Add new rows
- `.select("*")`: Get data
- `.eq("column", value)`: Filter data
- `.order("column")`: Sort data

### Security

- Always use Row Level Security (RLS)
- Never trust client-side validation alone
- Always check authentication before database operations
- Use the anon key (safe for client-side)

---

## Additional Resources

- [Supabase Documentation](https://supabase.com/docs)
- [Supabase JavaScript Client](https://supabase.com/docs/reference/javascript/introduction)
- [Supabase Auth](https://supabase.com/docs/guides/auth)
- [Row Level Security](https://supabase.com/docs/guides/auth/row-level-security)

---

**End of Documentation**

