# Supabase Setup Guide

This guide will help you set up Supabase integration for the Pal Tracker to enable cloud-based data storage and synchronization.

## Prerequisites

1. Create a free account at [Supabase](https://supabase.com)
2. Create a new project in your Supabase dashboard

## Database Setup

### 1. Create Tables

Run the following SQL in your Supabase SQL editor to create the necessary tables:

```sql
-- Table for storing pal data (for modular updates)
CREATE TABLE pals (
    id SERIAL PRIMARY KEY,
    pal_no VARCHAR(10) UNIQUE NOT NULL,
    name VARCHAR(100) NOT NULL,
    elements TEXT[] NOT NULL,
    skill VARCHAR(200),
    work VARCHAR(500),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Table for storing user statistics
CREATE TABLE user_stats (
    id SERIAL PRIMARY KEY,
    user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
    pal_no VARCHAR(10) NOT NULL,
    caught INTEGER DEFAULT 0,
    has_medal BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    UNIQUE(user_id, pal_no)
);

-- Enable Row Level Security
ALTER TABLE pals ENABLE ROW LEVEL SECURITY;
ALTER TABLE user_stats ENABLE ROW LEVEL SECURITY;

-- Create policies
-- Allow everyone to read pal data
CREATE POLICY "Enable read access for all users" ON pals FOR SELECT USING (true);

-- Allow users to read/write their own stats
CREATE POLICY "Users can view own stats" ON user_stats FOR SELECT USING (auth.uid() = user_id);
CREATE POLICY "Users can insert own stats" ON user_stats FOR INSERT WITH CHECK (auth.uid() = user_id);
CREATE POLICY "Users can update own stats" ON user_stats FOR UPDATE USING (auth.uid() = user_id);
CREATE POLICY "Users can delete own stats" ON user_stats FOR DELETE USING (auth.uid() = user_id);
```

### 2. Populate Initial Pal Data

After creating the tables, you can populate the `pals` table with the initial data. This can be done via the Supabase dashboard or by running insert statements.

## Application Configuration

### 1. Get Your Supabase Credentials

1. Go to your Supabase project dashboard
2. Navigate to Settings > API
3. Copy your:
   - Project URL
   - Anon (public) key

### 2. Update the Application

In `index.html`, find the Supabase configuration section and replace the placeholder values:

```javascript
const SUPABASE_URL = 'https://your-project-ref.supabase.co'; // Replace with your URL
const SUPABASE_ANON_KEY = 'your-anon-key-here'; // Replace with your anon key
```

## Features

### Current Integration

- **Automatic Sync**: Data is automatically saved to Supabase when you update caught counts or medals
- **Fallback to Local**: If Supabase is not configured or fails, the app falls back to localStorage
- **Connection Status**: Visual indicator shows whether you're connected to cloud sync or using local storage only

### Future Enhancements

- **User Authentication**: Add user accounts to sync data across devices
- **Pal Data Management**: Allow administrators to update pal information through Supabase
- **Real-time Sync**: Sync data in real-time across multiple devices
- **Backup/Restore**: Export and import data from Supabase
- **Statistics Dashboard**: Advanced analytics using Supabase queries

## Troubleshooting

### Common Issues

1. **Connection Status shows "Local Only"**
   - Check that your Supabase URL and anon key are correctly configured
   - Verify your Supabase project is active
   - Check browser console for error messages

2. **Data not syncing**
   - Ensure Row Level Security policies are set up correctly
   - Check that tables exist in your Supabase database
   - Verify network connection

3. **Authentication issues** (future feature)
   - Will require additional setup for user authentication
   - Currently the app works anonymously with localStorage fallback

## Security Notes

- The anon key is safe to use in client-side code as it has limited permissions
- Row Level Security policies ensure users can only access their own data
- Never expose your service role key in client-side code

## Support

For issues with this integration, please check:
1. Supabase documentation: https://supabase.com/docs
2. Browser developer console for error messages
3. Supabase project logs in the dashboard