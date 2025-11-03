# Discord Interest Matching Bot

An intelligent AI-powered Discord bot that matches people based on their interest graphs (similar jobs, location, hobbies, interests, etc.) using n8n workflows, LangChain, Airtable, and Cal.com.

## 🎯 Overview

This system automatically connects people in your Discord community through **two complementary features**:

### Feature 1: 1-on-1 Virtual Matches (AI-Powered)
1. **User Registration** - Users fill out a short Airtable form with their preferences
2. **AI Matching** - LangChain/OpenAI analyzes profiles and finds compatible matches
3. **Meeting Scheduling** - Matched pairs receive a Cal.com link to book a time
4. **Feedback Collection** - Users rate conversations to improve the algorithm

### Feature 2: Small Group Dinners (Balanced Diversity)
1. **Weekly Invitations** - Monday invites for Wednesday/Thursday dinners
2. **RSVP Collection** - Users sign up by Tuesday with preferences
3. **Group Formation** - Algorithm creates balanced groups of 5-6 people
4. **Location Assignment** - Restaurant details sent day before
5. **In-Person Dinners** - Meet for casual networking over food
6. **Feedback Loop** - Collect ratings to improve future groupings

**📖 See [DINNERS_README.md](DINNERS_README.md) for complete small group dinners documentation**

## 🏗️ Architecture

The system consists of **8 main n8n workflows**:

### 1-on-1 Virtual Matching (Workflows 1-4)

### 1. User Registration Workflow (`1_User_Registration_Workflow.json`)
- Triggered when users type `/register` in Discord
- Checks if user already exists
- Sends Airtable form link for preferences
- Stores user data in Airtable Users table
- Sends confirmation DM

### 2. AI Matching Workflow (`2_AI_Matching_Workflow.json`)
- Runs every 6 hours on a schedule
- Fetches all active users from Airtable
- Retrieves existing matches to avoid duplicates
- Uses OpenAI to calculate compatibility scores (0-100)
- Creates top 10 matches with scores ≥60
- Sends match notifications to both users
- Triggers scheduling workflow

### 3. Meeting Scheduling Workflow (`3_Meeting_Scheduling_Workflow.json`)
- Creates Cal.com event for matched pairs
- Sends booking links via Discord DMs
- Sends reminders for unscheduled meetings
- Updates match status when meeting is booked
- Handles Cal.com webhooks for confirmations

### 4. Feedback Collection Workflow (`4_Feedback_Collection_Workflow.json`)
- Runs every 6 hours to find completed meetings
- Sends feedback form to both users 2+ hours after meeting
- Stores ratings and comments in Airtable
- Marks matches as completed when both users submit
- Weekly analysis for algorithm improvements
- Sends admin reports on matching success

### Small Group Dinners (Workflows 5-8)

### 5. Dinner Interest Collection Workflow (`5_Dinner_Interest_Collection.json`)
- Monday 9 AM: Send weekly dinner invitations
- Collect RSVPs via Airtable form (day preference, location, dietary needs)
- Tuesday 5 PM: Close RSVPs and trigger group formation
- `/dinner-rsvp` Discord command for quick signup

### 6. Dinner Group Formation Workflow (`6_Dinner_Group_Formation.json`)
- Balanced algorithm creates diverse groups of 5-6 people
- Factors: industry diversity, experience mix, dietary compatibility, location preferences
- Avoids repeat pairings within 4 weeks
- Splits by day preference (Wednesday/Thursday)
- Sends group assignment DMs to all attendees

### 7. Dinner Reminders Workflow (`7_Dinner_Reminders.json`)
- Day before (9 AM): Assign restaurant location, send detailed reminders
- Creates Discord group thread for coordination
- Day of (3 PM): Final check-in reminder in thread
- Updates event status to "Confirmed"

### 8. Dinner Feedback Workflow (`8_Dinner_Feedback.json`)
- Day after (10 AM): Send feedback form to attendees
- Collects ratings: overall, group dynamics, location, conversation quality
- Marks RSVPs as "Attended" when feedback submitted
- Sunday (10 AM): Weekly analysis and admin report
- Tracks engagement metrics and improvement suggestions

## 📋 Prerequisites

### Required Services

1. **n8n Instance** (v1.0+)
   - Self-hosted or cloud: https://n8n.io
   - Install LangChain nodes: `npm install @n8n/n8n-nodes-langchain`

2. **Discord Bot**
   - Create bot at: https://discord.com/developers/applications
   - Enable: Message Content Intent, Server Members Intent
   - Bot permissions: Send Messages, Read Messages, Embed Links
   - Invite bot to your server

3. **Airtable Account**
   - Free or paid plan: https://airtable.com
   - Create workspace and base
   - Generate Personal Access Token

4. **OpenAI API**
   - Account: https://platform.openai.com
   - API key with GPT-4 access
   - Recommended model: `gpt-4` or `gpt-4-turbo`

5. **Cal.com Account**
   - Free or paid: https://cal.com
   - Generate API key
   - Set up default event types

## 🚀 Installation

### Step 1: Set Up Airtable

1. Create a new Airtable base called "Discord Interest Matching"
2. Follow the schema in `AIRTABLE_SCHEMA.md` to create:
   - **Users** table (stores user profiles)
   - **Matches** table (tracks all matches)
   - **Feedback** table (stores post-meeting ratings)
3. Create Airtable forms:
   - User Registration Form (linked to Users table)
   - Feedback Form (linked to Feedback table)
4. Generate Personal Access Token:
   - Go to https://airtable.com/account
   - Create token with `data.records:read` and `data.records:write` scopes
   - Save your Base ID from API docs

### Step 2: Configure Discord Bot

1. Go to https://discord.com/developers/applications
2. Click "New Application" and name it "Interest Matching Bot"
3. Go to "Bot" section:
   - Click "Add Bot"
   - Enable "Message Content Intent"
   - Enable "Server Members Intent"
   - Copy Bot Token (you'll need this)
4. Go to "OAuth2" > "URL Generator":
   - Select scopes: `bot`, `applications.commands`
   - Select permissions: `Send Messages`, `Read Messages`, `Embed Links`
   - Copy generated URL and invite bot to your server
5. (Optional) Set up slash commands:
   - `/register` - Start registration process
   - `/update-profile` - Update user preferences
   - `/help` - Get help information

### Step 3: Get API Keys

1. **OpenAI API Key**:
   - Go to https://platform.openai.com/api-keys
   - Create new secret key
   - Save it securely

2. **Cal.com API Key**:
   - Go to https://cal.com/settings/developer/api-keys
   - Generate new API key
   - Save it securely

### Step 4: Import Workflows to n8n

1. Open your n8n instance
2. Click "Workflows" > "Import from File"
3. Import all 4 workflow JSON files in order:
   - `1_User_Registration_Workflow.json`
   - `2_AI_Matching_Workflow.json`
   - `3_Meeting_Scheduling_Workflow.json`
   - `4_Feedback_Collection_Workflow.json`

### Step 5: Configure Credentials in n8n

Add credentials for each service:

1. **Discord Credentials**:
   - Type: Discord API
   - Bot Token: `<your-discord-bot-token>`

2. **Airtable Credentials**:
   - Type: Airtable Personal Access Token API
   - Access Token: `<your-airtable-token>`

3. **OpenAI Credentials**:
   - Type: OpenAI API
   - API Key: `<your-openai-key>`

4. **Cal.com Credentials**:
   - Type: Cal.com API
   - API Key: `<your-cal-api-key>`

### Step 6: Update Workflow Parameters

Edit each workflow and update the following:

**In Workflow 1 (User Registration):**
- Node "Get Airtable Form URL": Replace with your actual Airtable registration form URL
- All Airtable nodes: Select your base and tables

**In Workflow 2 (AI Matching):**
- All Airtable nodes: Select your base and tables
- Node "AI Match Scoring": Verify OpenAI model (`gpt-4` recommended)

**In Workflow 3 (Meeting Scheduling):**
- All Airtable nodes: Select your base and tables
- Cal.com nodes: Configure event type settings

**In Workflow 4 (Feedback Collection):**
- All Airtable nodes: Select your base and tables
- Node "Request Feedback": Replace with your actual Airtable feedback form URL

### Step 7: Activate Workflows

1. Test each workflow individually using the "Test Workflow" button
2. Once verified, activate all workflows:
   - Toggle "Active" switch on each workflow
3. Monitor executions in n8n's execution log

## 📖 User Guide

### For Users

**1. Register:**
```
/register
```
- Bot sends you an Airtable form link
- Fill out your profile (2-3 minutes)
- Submit and receive confirmation

**2. Get Matched:**
- Bot runs matching algorithm every 6 hours
- You'll receive a DM when matched
- See compatibility score and match reason

**3. Schedule Meeting:**
- Click the Cal.com link in your DM
- Book a time that works for both of you
- Receive confirmation and calendar invite

**4. Provide Feedback:**
- After your meeting, bot sends feedback form
- Rate the conversation (1-5 stars)
- Help improve future matches

### For Admins

**Monitor System:**
- Check n8n execution logs for errors
- Review Airtable for match statistics
- Weekly reports sent to admin channel (optional)

**Adjust Matching Frequency:**
- Edit Workflow 2 schedule trigger
- Default: Every 6 hours
- Options: 3 hours, 12 hours, daily, etc.

**Customize Matching Algorithm:**
- Edit the AI prompt in Workflow 2
- Adjust scoring weights:
  - Shared interests: 40% (default)
  - Career alignment: 30%
  - Geographic proximity: 20%
  - Goal alignment: 10%

## 🎛️ Configuration Options

### Matching Algorithm Tuning

Edit the AI prompt in **Workflow 2** > **AI Matching Prompt** node:

```
Scoring Weights (customize these):
- Shared interests, hobbies, skills: 40%
- Industry/career alignment: 30%
- Geographic proximity/timezone: 20%
- Alignment of goals: 10%
```

**Minimum Match Score:**
- Default: 60/100
- Edit in Workflow 2 > "Filter and Rank Matches" node
- Higher = more selective, Lower = more matches

**Maximum Matches Per Cycle:**
- Default: Top 10 matches
- Edit in Workflow 2 > "Filter and Rank Matches" node

### Notification Settings

**Reminder Timing:**
- Default: 3 days after match without booking
- Edit in Workflow 3 > "Find Pending Meetings" filter

**Feedback Request Timing:**
- Default: 2+ hours after meeting
- Edit in Workflow 4 > "Find Meetings Needing Feedback" filter

### Meeting Settings

**Default Meeting Duration:**
- Default: 60 minutes
- Edit in Workflow 3 > "Create Cal.com Event Type" node

**Meeting Platform:**
- Default: Zoom (via Cal.com)
- Options: Google Meet, Microsoft Teams, Custom
- Edit in Workflow 3 > locations array

## 📊 Data Schema

See `AIRTABLE_SCHEMA.md` for complete database structure.

**Key Tables:**
- **Users**: User profiles and preferences
- **Matches**: All match records with scores
- **Feedback**: Post-meeting ratings and comments

**Important Fields:**
- `Match Score`: 0-100 compatibility score
- `Status`: Pending → Scheduled → Completed
- `Feedback Received`: Count of feedback submissions

## 🧪 Testing

### Test User Registration
1. In Discord, type `/register`
2. Click the Airtable form link
3. Fill out with test data
4. Verify user appears in Airtable Users table
5. Check for confirmation DM

### Test Matching
1. Create at least 2 test users with similar interests
2. Manually trigger Workflow 2 (click "Execute Workflow")
3. Check Airtable Matches table for new records
4. Verify both users received DMs

### Test Scheduling
1. Use a test match from above
2. Click Cal.com link in DM
3. Book a test meeting
4. Verify Airtable updates and confirmation DM

### Test Feedback
1. Set a test match to "past meeting date"
2. Manually trigger Workflow 4
3. Check for feedback request DM
4. Submit feedback via form
5. Verify Airtable Feedback table

## 🔧 Troubleshooting

### Common Issues

**Issue: No matches being created**
- Check Airtable has active users (Status = "Active")
- Verify OpenAI credentials are valid
- Check n8n execution logs for errors
- Ensure users have enough profile data

**Issue: Discord messages not sending**
- Verify Discord bot is in the server
- Check bot has permission to DM users
- Users must have DMs enabled from server members
- Verify Discord credentials in n8n

**Issue: Cal.com links not working**
- Verify Cal.com API key is valid
- Check event type was created successfully
- Ensure users have valid email addresses
- Verify Cal.com account has active event types

**Issue: Feedback not being collected**
- Check meeting dates are in the past
- Verify Airtable form URL is correct
- Ensure webhooks are properly configured
- Check filter formula in "Find Meetings Needing Feedback"

### Debug Mode

Enable debug output in workflows:
1. Add "Sticky Note" nodes to visualize data flow
2. Use "Edit Fields (Set)" nodes to inspect JSON
3. Check n8n execution logs for detailed errors
4. Enable workflow execution history

## 🔒 Security & Privacy

### Best Practices

1. **Credentials**: Never share API keys or tokens
2. **User Data**: Inform users their data is stored in Airtable
3. **Consent**: Add privacy policy to registration form
4. **Access Control**: Limit Airtable base access to admins
5. **Data Retention**: Set up automatic deletion for old data

### GDPR Compliance

- Add "Delete My Data" command for users
- Store consent timestamps
- Provide data export functionality
- Implement right to be forgotten

## 📈 Scaling

### For Large Communities (1000+ users)

**Optimization Tips:**
1. **Matching Frequency**: Reduce to 12-24 hours
2. **Batch Matching**: Process in groups of 100 users
3. **Caching**: Store user vectors for faster comparisons
4. **Database**: Consider upgrading Airtable plan or migrating to PostgreSQL
5. **Rate Limits**: Add delays between Discord messages

**Performance Benchmarks:**
- 100 users: ~5 minutes per matching cycle
- 500 users: ~15-20 minutes per cycle
- 1000+ users: Consider splitting into segments

## 🤝 Contributing

### Ideas for Enhancements

- [ ] Add personality questionnaire (Myers-Briggs, Big Five)
- [ ] Implement group matching (3-4 people)
- [ ] Add video intro messages
- [ ] Create matching preferences (mentor/mentee)
- [ ] Build web dashboard for analytics
- [ ] Add Slack/Teams integration
- [ ] Implement machine learning for match prediction
- [ ] Create mobile app for easier registration

### How to Contribute

1. Fork this repository
2. Test your changes thoroughly
3. Update documentation
4. Submit pull request with description

## 📝 License

This project is provided as-is for educational and non-commercial use.

## 🙏 Acknowledgments

Built with:
- [n8n](https://n8n.io) - Workflow automation
- [OpenAI](https://openai.com) - AI matching algorithm
- [Airtable](https://airtable.com) - Database
- [Cal.com](https://cal.com) - Scheduling
- [Discord](https://discord.com) - Communication

## 📞 Support

### Resources
- n8n Documentation: https://docs.n8n.io
- n8n Community: https://community.n8n.io
- Airtable API Docs: https://airtable.com/api
- Cal.com API Docs: https://cal.com/docs/api-reference
- OpenAI API Docs: https://platform.openai.com/docs

### Getting Help

1. Check n8n execution logs for errors
2. Review Airtable data for inconsistencies
3. Test each workflow individually
4. Ask in n8n community forum
5. Check Discord bot permissions

## 🎉 Success Metrics

Track these KPIs in your Airtable:

- **Total Users Registered**: Count of active users
- **Matches Created**: Total matches with score ≥60
- **Meeting Completion Rate**: % of matches that resulted in meetings
- **Average Match Score**: Mean compatibility score
- **Average Feedback Rating**: Mean user satisfaction
- **Would Meet Again %**: Percentage of positive feedback

## 🔄 Maintenance

### Weekly Tasks
- Review feedback for algorithm improvements
- Check for failed workflow executions
- Monitor API usage and costs
- Update user preferences as needed

### Monthly Tasks
- Analyze matching success rates
- Adjust algorithm weights based on feedback
- Clean up old/inactive user records
- Review and optimize workflows

---

**Version**: 1.0
**Last Updated**: 2025-11-02
**Minimum n8n Version**: 1.0+
**Author**: Created with Claude Code

For questions or issues, refer to the troubleshooting section above or reach out to the n8n community.
