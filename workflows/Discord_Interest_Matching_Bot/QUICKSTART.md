# Quick Start Guide - Discord Interest Matching Bot

Get your Discord Interest Matching Bot running in **30 minutes**! ⚡

## ✅ Prerequisites Checklist

Before starting, make sure you have:

- [ ] n8n instance (self-hosted or cloud)
- [ ] Discord account with server admin access
- [ ] Airtable account (free tier works)
- [ ] OpenAI API account with credits
- [ ] Cal.com account (free tier works)

## 🚀 5-Step Setup

### Step 1: Create Discord Bot (5 mins)

1. Go to https://discord.com/developers/applications
2. Click **"New Application"** → Name it "Interest Matcher"
3. Go to **"Bot"** tab → Click **"Add Bot"**
4. Under "Privileged Gateway Intents", enable:
   - ✅ Message Content Intent
   - ✅ Server Members Intent
5. Click **"Reset Token"** → Copy and save your bot token
6. Go to **"OAuth2"** → **"URL Generator"**:
   - Scopes: `bot`, `applications.commands`
   - Permissions: `Send Messages`, `Embed Links`, `Use Slash Commands`
7. Copy the generated URL and open it to invite bot to your server

**Save this:** `DISCORD_BOT_TOKEN=your_token_here`

---

### Step 2: Set Up Airtable (10 mins)

1. **Create Base**:
   - Go to https://airtable.com
   - Click **"Add a base"** → **"Start from scratch"**
   - Name it: "Discord Interest Matching"

2. **Create Users Table**:
   - Rename "Table 1" to "Users"
   - Add these fields (click **"+"** to add field):
     ```
     ✅ User ID (Auto Number) - already exists as record ID
     ✅ Discord ID (Single Line Text)
     ✅ Discord Username (Single Line Text)
     ✅ Email (Email)
     ✅ Full Name (Single Line Text)
     ✅ Location (Single Line Text)
     ✅ Job Title (Single Line Text)
     ✅ Hobbies (Multiple Select) - add options: Gaming, Reading, Hiking, etc.
     ✅ Interests (Long Text)
     ✅ Skills (Multiple Select) - add options: Python, JavaScript, Design, etc.
     ✅ Timezone (Single Select) - add options: UTC-8, UTC-5, UTC+0, etc.
     ✅ Status (Single Select) - add options: Active, Paused, Inactive
     ```

3. **Create Matches Table**:
   - Click **"+"** next to "Users" to add new table
   - Name it: "Matches"
   - Add these fields:
     ```
     ✅ Match ID (Auto Number)
     ✅ User 1 (Link to Users table)
     ✅ User 2 (Link to Users table)
     ✅ Match Score (Number)
     ✅ Match Reason (Long Text)
     ✅ Status (Single Select) - options: Pending, Scheduled, Completed
     ✅ Cal Link Sent (Checkbox)
     ✅ Meeting Scheduled (Checkbox)
     ✅ Meeting Date (Date)
     ```

4. **Create Feedback Table**:
   - Add another table named "Feedback"
   - Add these fields:
     ```
     ✅ Feedback ID (Auto Number)
     ✅ Match (Link to Matches table)
     ✅ User (Link to Users table)
     ✅ Rating (Single Select) - options: 1, 2, 3, 4, 5
     ✅ Conversation Quality (Single Select) - options: 1, 2, 3, 4, 5
     ✅ Comments (Long Text)
     ```

5. **Create Forms**:
   - In Users table, click **"Forms"** → **"Create form"**
   - Add all user fields to the form
   - Click **"Share form"** → Copy the link
   - Do the same for Feedback table

6. **Get API Token**:
   - Go to https://airtable.com/account
   - Click **"Generate token"**
   - Give it a name: "n8n Integration"
   - Add scopes: `data.records:read`, `data.records:write`
   - Add access to your "Discord Interest Matching" base
   - Click **"Create token"** → Copy and save

7. **Get Base ID**:
   - Go to https://airtable.com/api
   - Select "Discord Interest Matching" base
   - Copy the Base ID (starts with "app...")

**Save these:**
```
AIRTABLE_TOKEN=your_token_here
AIRTABLE_BASE_ID=appXXXXXXXXXXXXXX
USERS_FORM_URL=https://airtable.com/shrXXXXXXXXXXXXXX
FEEDBACK_FORM_URL=https://airtable.com/shrYYYYYYYYYYYYYY
```

---

### Step 3: Get API Keys (3 mins)

1. **OpenAI**:
   - Go to https://platform.openai.com/api-keys
   - Click **"Create new secret key"**
   - Name it: "Discord Matcher"
   - Copy and save the key

2. **Cal.com**:
   - Go to https://cal.com/settings/developer/api-keys
   - Click **"Create new API key"**
   - Copy and save the key

**Save these:**
```
OPENAI_API_KEY=sk-...
CAL_API_KEY=cal_...
```

---

### Step 4: Import to n8n (7 mins)

1. **Open n8n** (http://localhost:5678 or your cloud URL)

2. **Add Credentials**:
   - Go to **Settings** → **Credentials** → **Add Credential**

   **Discord API:**
   - Search for "Discord"
   - Paste your `DISCORD_BOT_TOKEN`
   - Save as "Discord Bot Account"

   **Airtable:**
   - Search for "Airtable Personal Access Token"
   - Paste your `AIRTABLE_TOKEN`
   - Save as "Airtable Personal Access Token"

   **OpenAI:**
   - Search for "OpenAI"
   - Paste your `OPENAI_API_KEY`
   - Save as "OpenAI API"

   **Cal.com:**
   - Search for "Cal"
   - Paste your `CAL_API_KEY`
   - Save as "Cal.com API"

3. **Import Workflows**:
   - Click **"Workflows"** → **"Import from File"**
   - Import these files in order:
     1. `1_User_Registration_Workflow.json`
     2. `2_AI_Matching_Workflow.json`
     3. `3_Meeting_Scheduling_Workflow.json`
     4. `4_Feedback_Collection_Workflow.json`

4. **Update Each Workflow**:

   **Workflow 1 - User Registration:**
   - Open the workflow
   - Find node "Get Airtable Form URL"
   - Update URL to your `USERS_FORM_URL`
   - Find all Airtable nodes → Select your base and "Users" table
   - Click **"Save"**

   **Workflow 2 - AI Matching:**
   - Open the workflow
   - Find all Airtable nodes → Select base and tables:
     - "Get Active Users" → Users table
     - "Get Existing Matches" → Matches table
     - "Create Match in Airtable" → Matches table
   - Verify OpenAI credential is selected
   - Click **"Save"**

   **Workflow 3 - Meeting Scheduling:**
   - Open the workflow
   - Find all Airtable nodes → Select base and tables
   - Find Cal.com nodes → Verify credential is selected
   - Click **"Save"**

   **Workflow 4 - Feedback Collection:**
   - Open the workflow
   - Find node "Request Feedback from User 1"
   - Update feedback form URL to your `FEEDBACK_FORM_URL`
   - Do the same for "Request Feedback from User 2"
   - Find all Airtable nodes → Select base and tables
   - Click **"Save"**

---

### Step 5: Test & Activate (5 mins)

1. **Test Registration**:
   - In Discord, type anything to trigger bot (or set up slash command)
   - For testing, manually trigger Workflow 1
   - Fill out the Airtable form with test data
   - Check that user appears in Airtable Users table

2. **Test Matching**:
   - Create at least 2 test users with similar interests
   - Open Workflow 2
   - Click **"Execute Workflow"** (top right)
   - Check Airtable Matches table for new match
   - Verify Discord DMs were sent (check bot messages)

3. **Test Scheduling** (Optional):
   - Click the Cal.com link from test match DM
   - Verify Cal.com booking page opens

4. **Activate All Workflows**:
   - For each workflow, toggle **"Active"** switch (top right)
   - All 4 workflows should show green "Active" badge

---

## ✅ You're Done!

Your Discord Interest Matching Bot is now running! 🎉

### What Happens Now?

1. **Every 6 hours**: AI matching algorithm runs automatically
2. **When matched**: Users receive Discord DMs with compatibility scores
3. **After booking**: Cal.com sends calendar invites
4. **After meetings**: Users receive feedback requests
5. **Weekly**: System analyzes feedback to improve matching

### Next Steps

1. **Invite Users**: Share registration instructions in your Discord
2. **Monitor**: Check n8n execution logs for any errors
3. **Customize**: Adjust matching algorithm weights in Workflow 2
4. **Scale**: Once stable, invite more users to register

---

## 🐛 Quick Troubleshooting

**Problem: Discord bot not responding**
- ✅ Check bot is online (green status in Discord)
- ✅ Verify bot has permissions in your server
- ✅ Test Discord credential in n8n

**Problem: No matches being created**
- ✅ Ensure at least 2 users with Status = "Active" in Airtable
- ✅ Check OpenAI API has credits
- ✅ Review n8n execution log for errors

**Problem: Airtable not updating**
- ✅ Verify Airtable token has write permissions
- ✅ Check base ID is correct
- ✅ Ensure table names match exactly ("Users", "Matches", "Feedback")

**Problem: Cal.com links not working**
- ✅ Verify Cal.com API key is valid
- ✅ Check you have at least one event type in Cal.com
- ✅ Ensure emails are valid in user profiles

---

## 📚 Resources

- **Full Documentation**: See `README.md`
- **Airtable Schema**: See `AIRTABLE_SCHEMA.md`
- **n8n Docs**: https://docs.n8n.io
- **Discord Developer**: https://discord.com/developers/docs

---

## 💡 Pro Tips

1. **Start Small**: Test with 5-10 users before scaling
2. **Adjust Frequency**: Change matching schedule in Workflow 2 if needed
3. **Monitor Costs**: OpenAI API charges per request
4. **Backup Data**: Export Airtable base regularly
5. **Iterate**: Use feedback to improve matching criteria

---

**Need Help?** Check the full `README.md` or visit the n8n community forum.

Happy Matching! 🚀
