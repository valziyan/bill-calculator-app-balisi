# Testing Guide

## Quick Test Checklist

### 1. Open the Application
- Open `index.html` in your web browser
- You should see the homepage with "Electric Bill Calculator" title

### 2. Test User Registration
1. Click **Register** button or go to `register.html`
2. Fill in the registration form:
   - Full Name: (e.g., "John Doe")
   - Email: (use a valid email address)
   - Password: (at least 6 characters)
   - Confirm Password: (must match)
3. Click **Register**
4. **Expected Result:**
   - Success message: "Registration successful! Please check your email..."
   - Redirects to login page
   - Check your email for verification (if email confirmation is enabled in Supabase)

### 3. Test User Login
1. Go to `login.html`
2. Enter your registered email and password
3. Click **Login**
4. **Expected Result:**
   - Successfully logs in
   - Redirects to `user_dashboard.html`
   - Welcome message shows your name/email

### 4. Test Page Protection
1. **Without logging in**, try to access:
   - `user_dashboard.html`
   - `calculator_page.html`
   - `results_page.html`
   - `profile_page.html`
2. **Expected Result:**
   - All pages should redirect to `login.html`

### 5. Test Calculator
1. After logging in, go to **Calculator** page
2. Fill in the form:
   - Month: (e.g., "January 2024")
   - Power Consumption: (e.g., 100)
   - Cost per kWh: (e.g., 10.50)
3. Click **Calculate**
4. **Expected Result:**
   - Shows result: "Your electric bill is: ₱ 1050.00"
   - Table appears with the calculation
   - Data is saved to Supabase database
   - Form clears after calculation

### 6. Test Results Page
1. Go to **Results** page
2. **Expected Result:**
   - Shows all your calculations in a table
   - Displays: Month, Power Consumption, Cost Per kWh, Result
   - Most recent calculations appear first

### 7. Test Multiple Calculations
1. Go back to Calculator
2. Create 2-3 more calculations with different values
3. Go to Results page
4. **Expected Result:**
   - All calculations should be displayed
   - Each calculation should have correct values

### 8. Test Logout
1. Click **Logout** in the navigation
2. **Expected Result:**
   - Successfully logs out
   - Redirects to `login.html`
3. Try accessing protected pages again
4. **Expected Result:**
   - Should redirect to login page

### 9. Test Profile Page
1. Login again
2. Go to **Profile** page
3. Update your profile information
4. Click **Update Profile**
5. **Expected Result:**
   - Profile updates successfully
   - Shows success message

## Troubleshooting

### If Registration/Login Fails:
- **Check Browser Console** (F12 → Console tab) for errors
- Verify your Supabase API key in `js/supabase-config.js`
- Check Supabase dashboard → Authentication → Providers (Email should be enabled)
- Verify the Supabase URL is correct

### If Calculations Don't Save:
- **Check Browser Console** for errors
- Verify the `calculations` table exists in Supabase
- Check Row Level Security policies are set up correctly
- Verify you're logged in (check session)

### If Results Page is Empty:
- Make sure you've created at least one calculation
- Check browser console for errors
- Verify database connection in Supabase dashboard
- Check if RLS policies allow SELECT operations

### Common Issues:
1. **CORS Errors**: Make sure Supabase URL and key are correct
2. **Authentication Errors**: Check if email confirmation is required in Supabase settings
3. **Database Errors**: Verify schema.sql was run successfully
4. **RLS Errors**: Check if Row Level Security policies are correctly set

## Verify Database in Supabase

1. Go to Supabase Dashboard → **Table Editor**
2. Select `calculations` table
3. You should see your saved calculations
4. Each row should have:
   - `user_id` (UUID)
   - `month` (text)
   - `power_consumption` (decimal)
   - `cost_per_kwh` (decimal)
   - `result` (decimal)
   - `created_at` (timestamp)

## Browser Console Commands

Open browser console (F12) and try:

```javascript
// Check if Supabase is loaded
console.log(supabaseClient);

// Check current session
checkAuth().then(session => console.log('Session:', session));

// Check current user
getCurrentUser().then(user => console.log('User:', user));
```

