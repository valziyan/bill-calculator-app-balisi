# Supabase Setup Instructions

## 1. Get Your Supabase Anon Key

1. Go to your Supabase project dashboard: https://supabase.com/dashboard
2. Select your project
3. Go to **Settings** → **API**
4. Copy the **anon/public** key
5. Replace `YOUR_SUPABASE_ANON_KEY` in `js/supabase-config.js` with your actual key

## 2. Create Database Tables

1. Open your Supabase project dashboard
2. Go to **SQL Editor**
3. Click **New Query**
4. Copy and paste the contents of `schema.sql` file (or open the file and copy all SQL)
5. Click **Run** to execute the SQL

This will create:

-   The `calculations` table with all required columns
-   Indexes for better query performance
-   Row Level Security (RLS) policies to ensure users can only access their own data

**Alternative:** You can also open `schema.sql` in your project and copy the SQL directly from there.

## 3. Enable Email Authentication

1. Go to **Authentication** → **Providers** in your Supabase dashboard
2. Make sure **Email** provider is enabled
3. Configure email templates if needed (optional)

## 4. Test the Application

1. Open `index.html` in your browser
2. Register a new account
3. Check your email for verification (if email confirmation is enabled)
4. Login with your credentials
5. Try calculating a bill - it should save to the database
6. Check the Results page - it should display your calculations

## Protected Pages

The following pages require authentication and will redirect to login if not authenticated:

-   `user_dashboard.html`
-   `calculator_page.html`
-   `results_page.html`
-   `profile_page.html`

## Notes

-   Make sure to replace `YOUR_SUPABASE_ANON_KEY` in `js/supabase-config.js`
-   The Supabase URL is already configured: `https://nrdadjisfwvgjgyshdqc.supabase.co`
-   All calculations are stored per user (user_id)
-   Row Level Security ensures users can only see their own data
