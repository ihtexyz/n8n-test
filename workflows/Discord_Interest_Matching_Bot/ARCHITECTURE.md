# System Architecture - Discord Interest Matching Bot

## 🏗️ High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         DISCORD SERVER                          │
│  (Users interact with bot via DMs and slash commands)          │
└────────────┬────────────────────────────────────┬───────────────┘
             │                                    │
             │ /register                          │ Match notifications
             │ /update-profile                    │ Feedback requests
             │ /help                              │
             ▼                                    ▼
┌─────────────────────────────────────────────────────────────────┐
│                         DISCORD BOT                             │
│              (n8n Discord Integration Nodes)                    │
└────────────┬────────────────────────────────────┬───────────────┘
             │                                    │
             ▼                                    ▼
┌──────────────────────────┐         ┌──────────────────────────┐
│   WORKFLOW 1             │         │   WORKFLOW 2             │
│   User Registration      │         │   AI Matching Engine     │
│                          │         │                          │
│   Triggers:              │         │   Triggers:              │
│   • Discord slash cmd    │         │   • Schedule (6 hours)   │
│   • Airtable webhook     │         │   • Manual execution     │
│                          │         │                          │
│   Actions:               │         │   Actions:               │
│   1. Check existing user │         │   1. Fetch active users  │
│   2. Send form link      │         │   2. Get past matches    │
│   3. Store in Airtable   │         │   3. AI compatibility    │
│   4. Send confirmation   │         │   4. Create matches      │
│                          │         │   5. Notify users        │
└────────────┬─────────────┘         └─────────┬────────────────┘
             │                                 │
             ▼                                 ▼
┌──────────────────────────┐         ┌──────────────────────────┐
│   WORKFLOW 3             │◄────────┤   WORKFLOW 4             │
│   Meeting Scheduling     │         │   Feedback Collection    │
│                          │         │                          │
│   Triggers:              │         │   Triggers:              │
│   • Workflow 2 callback  │         │   • Schedule (6 hours)   │
│   • Cal.com webhook      │         │   • Meeting completion   │
│                          │         │                          │
│   Actions:               │         │   Actions:               │
│   1. Create Cal event    │         │   1. Find past meetings  │
│   2. Send booking links  │         │   2. Request feedback    │
│   3. Send reminders      │         │   3. Store in Airtable   │
│   4. Confirm booking     │         │   4. Analyze patterns    │
│                          │         │   5. Update algorithm    │
└────────────┬─────────────┘         └─────────┬────────────────┘
             │                                 │
             ▼                                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                         AIRTABLE DATABASE                       │
│                                                                 │
│   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│   │   USERS     │  │   MATCHES   │  │  FEEDBACK   │          │
│   │             │  │             │  │             │          │
│   │ • Profiles  │  │ • User 1/2  │  │ • Ratings   │          │
│   │ • Interests │  │ • Score     │  │ • Comments  │          │
│   │ • Location  │  │ • Status    │  │ • Duration  │          │
│   │ • Skills    │  │ • Cal Link  │  │ • Topics    │          │
│   └─────────────┘  └─────────────┘  └─────────────┘          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
             │                                 │
             ▼                                 ▼
┌──────────────────────────┐         ┌──────────────────────────┐
│       OPENAI API         │         │       CAL.COM API        │
│                          │         │                          │
│   • GPT-4 Analysis       │         │   • Event creation       │
│   • Compatibility Score  │         │   • Booking links        │
│   • Match Reasoning      │         │   • Calendar sync        │
│   • Pattern Learning     │         │   • Webhooks             │
└──────────────────────────┘         └──────────────────────────┘
```

## 📊 Data Flow Diagram

### 1. User Registration Flow

```
User types /register in Discord
         │
         ▼
Discord Bot receives command
         │
         ▼
Check if user exists in Airtable
         │
    ┌────┴────┐
    │         │
  Exists   New User
    │         │
    ▼         ▼
 Already    Send Airtable
 Registered  Form Link
    │         │
    │         ▼
    │    User fills form
    │         │
    │         ▼
    │    Webhook triggers
    │         │
    │         ▼
    │    Store in Airtable
    │    (Users table)
    │         │
    └────┬────┘
         │
         ▼
Send confirmation DM
         │
         ▼
User is now in matching pool
```

### 2. AI Matching Flow

```
Schedule Trigger (every 6 hours)
         │
         ▼
Fetch all Active users from Airtable
         │
         ▼
Fetch existing matches (avoid duplicates)
         │
         ▼
Generate all possible user pairs
(User1, User2) where not already matched
         │
         ▼
For each pair:
  ┌──────────────────────────┐
  │ Send to OpenAI GPT-4     │
  │ with matching prompt     │
  │                          │
  │ Analyze:                 │
  │ • Shared interests (40%) │
  │ • Career alignment (30%) │
  │ • Location/timezone (20%)│
  │ • Goals alignment (10%)  │
  └──────────┬───────────────┘
             │
             ▼
  OpenAI returns:
  • Compatibility score (0-100)
  • Reasoning text
  • Confidence level
         │
         ▼
Filter matches where score ≥ 60
         │
         ▼
Sort by score (highest first)
         │
         ▼
Take top 10 matches
         │
         ▼
For each match:
  1. Create record in Airtable
  2. Send Discord DM to both users
  3. Trigger Scheduling Workflow
```

### 3. Meeting Scheduling Flow

```
Triggered by Matching Workflow
         │
         ▼
Receive match data (User1, User2, Score)
         │
         ▼
Fetch full user details from Airtable
         │
         ▼
Create Cal.com event type
  • Title: "Interest Match: User1 & User2"
  • Duration: 60 minutes
  • Hosts: Both users
  • Type: COLLECTIVE (both must agree)
         │
         ▼
Get unique booking link
         │
         ▼
Update match record in Airtable
  • Set "Cal Link Sent" = true
  • Store booking URL
  • Set Status = "Pending"
         │
         ▼
Send Discord DM with Cal link to User1
         │
         ▼
Send Discord DM with Cal link to User2
         │
         ▼
Wait for booking...
         │
    ┌────┴────┐
    │         │
  Booked   Not Booked
    │         │
    ▼         ▼
Cal.com   After 3 days
Webhook   Send reminder
    │         │
    └────┬────┘
         │
         ▼
Update Airtable:
  • Set "Meeting Scheduled" = true
  • Store meeting date/time
  • Set Status = "Scheduled"
         │
         ▼
Send confirmation to both users
```

### 4. Feedback Collection Flow

```
Schedule Trigger (every 6 hours)
         │
         ▼
Find matches where:
  • Meeting date is in past (2+ hours ago)
  • Feedback received < 2
  • Within last 7 days
         │
         ▼
For each match:
  Fetch User1 and User2 details
         │
         ▼
  Send feedback form link via Discord DM
  to both users
         │
         ▼
Users fill out Airtable form:
  • Rating (1-5)
  • Conversation quality (1-5)
  • Would meet again? (Yes/Maybe/No)
  • Topics discussed
  • Comments
  • Meeting duration
         │
         ▼
Form submission triggers webhook
         │
         ▼
Store feedback in Airtable
  • Link to Match record
  • Link to User record
         │
         ▼
Check if both users submitted
         │
    ┌────┴────┐
    │         │
   Yes        No
    │         │
    ▼         ▼
Mark match  Wait for
"Completed" other user
    │
    ▼
Send thank you DM
         │
         ▼
Weekly Analysis (every Sunday):
  1. Aggregate all feedback
  2. Calculate average ratings
  3. Identify popular topics
  4. Generate algorithm suggestions
  5. Send report to admin
```

## 🔄 Workflow Dependencies

```
┌─────────────────┐
│   Workflow 1    │
│  Registration   │
└────────┬────────┘
         │
         │ Creates users in Airtable
         │
         ▼
┌─────────────────┐
│   Workflow 2    │  ──┐
│  AI Matching    │    │
└────────┬────────┘    │
         │             │
         │ Triggers    │
         │             │
         ▼             │
┌─────────────────┐    │
│   Workflow 3    │    │
│   Scheduling    │    │
└────────┬────────┘    │
         │             │
         │ Creates     │ All workflows
         │ meetings    │ read/write to
         │             │ Airtable
         ▼             │
┌─────────────────┐    │
│   Workflow 4    │  ──┘
│   Feedback      │
└────────┬────────┘
         │
         │ Updates algorithm
         │
         └─────┐
               ▼
         Improves future matches
```

## 🗃️ Database Schema Relationships

```
┌──────────────────────┐
│       USERS          │
│────────────────────  │
│ id (PK)              │
│ Discord ID           │◄────┐
│ Name, Email          │     │
│ Location, Job        │     │
│ Hobbies, Interests   │     │
│ Skills, Timezone     │     │
│ Status (Active/...)  │     │
└──────────┬───────────┘     │
           │                 │
           │ Linked Record   │
           │                 │
           ▼                 │
┌──────────────────────┐     │
│      MATCHES         │     │
│────────────────────  │     │
│ id (PK)              │     │
│ User 1 (FK) ─────────┼─────┘
│ User 2 (FK) ─────────┼─────┐
│ Match Score          │     │
│ Match Reason         │     │
│ Status               │     │
│ Cal Link Sent        │     │
│ Meeting Scheduled    │     │
│ Meeting Date         │     │
└──────────┬───────────┘     │
           │                 │
           │ Linked Record   │
           │                 │
           ▼                 │
┌──────────────────────┐     │
│     FEEDBACK         │     │
│────────────────────  │     │
│ id (PK)              │     │
│ Match (FK)           │     │
│ User (FK) ───────────┼─────┘
│ Rating               │
│ Conversation Quality │
│ Would Meet Again     │
│ Topics Discussed     │
│ Comments             │
│ Duration             │
└──────────────────────┘
```

## ⚙️ Component Interactions

### External Service Integrations

```
┌─────────────────────────────────────────────────────────┐
│                    n8n WORKFLOWS                        │
└───┬─────────┬─────────┬─────────┬──────────────────────┘
    │         │         │         │
    │         │         │         │
    ▼         ▼         ▼         ▼
┌─────┐  ┌─────┐  ┌─────┐  ┌─────────┐
│Discord  │Airtable OpenAI│  │Cal.com  │
│        │        │       │  │         │
│ Bot    │Database│ GPT-4 │  │Scheduler│
│ API    │  API   │  API  │  │   API   │
└─────┘  └─────┘  └─────┘  └─────────┘
   │        │        │          │
   │        │        │          │
   └────────┴────────┴──────────┘
              │
              ▼
        User Experience
```

### API Rate Limits & Considerations

| Service | Rate Limit | Cost | Mitigation |
|---------|-----------|------|------------|
| Discord | 50 req/sec | Free | Batch messages |
| Airtable | 5 req/sec | Free tier: 1,000/mo | Cache reads |
| OpenAI | Varies by tier | ~$0.01-0.03/match | Batch processing |
| Cal.com | 100 req/min | Free tier available | Queue requests |

## 🔐 Security Architecture

```
┌────────────────────────────────────────────┐
│           n8n Instance                     │
│  ┌──────────────────────────────────────┐ │
│  │      Credentials Store (Encrypted)    │ │
│  │  • Discord Bot Token                  │ │
│  │  • Airtable Personal Access Token     │ │
│  │  • OpenAI API Key                     │ │
│  │  • Cal.com API Key                    │ │
│  └──────────────────────────────────────┘ │
└────────────────────────────────────────────┘
         │
         │ HTTPS/TLS
         │
         ▼
┌────────────────────────────────────────────┐
│         External Services                  │
│  • All API calls over HTTPS               │
│  • OAuth 2.0 where supported              │
│  • Webhooks with signature verification   │
│  • No sensitive data in workflow files    │
└────────────────────────────────────────────┘
```

### Data Privacy

- **User Data**: Stored in Airtable with controlled access
- **PII**: Discord IDs, emails, names (with user consent)
- **Credentials**: Never logged or exposed in workflows
- **Webhooks**: Validated and rate-limited
- **GDPR**: Users can request data deletion

## 📈 Scalability Considerations

### Current Capacity

```
Small: 10-50 users
  • Matching time: ~30 seconds
  • OpenAI cost: ~$0.50/cycle
  • Airtable: Free tier sufficient

Medium: 50-500 users
  • Matching time: ~5-10 minutes
  • OpenAI cost: ~$5-10/cycle
  • Airtable: Paid tier recommended

Large: 500+ users
  • Matching time: 15-30 minutes
  • OpenAI cost: $20-50/cycle
  • Airtable: Pro tier or migrate to PostgreSQL
  • Consider: User segmentation, caching, async processing
```

### Optimization Strategies

1. **User Segmentation**: Split into interest-based groups
2. **Caching**: Store user vectors for quick comparisons
3. **Batch Processing**: Process matches in groups of 100
4. **Schedule Optimization**: Run during off-peak hours
5. **Database Migration**: Move to PostgreSQL for 1000+ users

## 🔧 Monitoring & Observability

```
┌────────────────────────────────────┐
│      n8n Execution Logs            │
│  • Workflow success/failure        │
│  • Execution duration              │
│  • Error stack traces              │
└───────────┬────────────────────────┘
            │
            ▼
┌────────────────────────────────────┐
│      Airtable Audit Log            │
│  • Record creation timestamps      │
│  • Field modification history      │
│  • Match completion rates          │
└───────────┬────────────────────────┘
            │
            ▼
┌────────────────────────────────────┐
│      Weekly Analytics Report       │
│  • Total matches created           │
│  • Average match score             │
│  • Meeting completion rate         │
│  • Average feedback rating         │
│  • Algorithm performance           │
└────────────────────────────────────┘
```

## 🎯 Performance Metrics

### Key Performance Indicators (KPIs)

```
Registration Funnel:
  Discord Command → Form View → Form Submit → Active User
       100%            85%          70%          65%

Matching Efficiency:
  Active Users → Potential Pairs → Matched → Meeting Booked
      100%           40%              15%          10%

Meeting Success:
  Booked → Completed → Positive Feedback → Would Meet Again
   100%       80%            70%                 60%

Algorithm Accuracy:
  Match Score 90+: 85% positive feedback
  Match Score 80-89: 75% positive feedback
  Match Score 70-79: 65% positive feedback
  Match Score 60-69: 50% positive feedback
```

---

**Version**: 1.0
**Last Updated**: 2025-11-02
**Maintained By**: n8n Community
