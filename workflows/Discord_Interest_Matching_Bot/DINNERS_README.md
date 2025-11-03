# Small Group Dinners Feature

## 🍽️ Overview

The Small Group Dinners feature organizes weekly in-person dinners for 5-6 people from your Discord community. Unlike the 1-on-1 virtual matches, these dinners focus on **creating balanced, diverse groups** for casual face-to-face networking over food.

**Key Characteristics:**
- **Group Size**: 5-6 people per dinner
- **Schedule**: Wednesday or Thursday evenings at 7 PM
- **Location**: In-person at local restaurants
- **Selection**: Balanced/diverse mix (more random than AI-matched)
- **Frequency**: Weekly recurring
- **Goal**: Expose people to diverse perspectives and new connections

---

## 🎯 How It Works

### User Journey

```
Monday
   │
   ├─> Receive dinner invitation (Discord DM + Channel post)
   │
   ▼
Tuesday 5 PM
   │
   ├─> RSVP deadline - fill out preference form
   │
   ▼
Tuesday Evening
   │
   ├─> Bot creates balanced groups
   ├─> Assigns to Wednesday or Thursday
   ├─> Notifies you of your group
   │
   ▼
Wednesday/Thursday Morning
   │
   ├─> Receive location details
   ├─> Join Discord group thread
   ├─> Coordinate with group members
   │
   ▼
Dinner Day
   │
   ├─> Final reminder (3 PM)
   ├─> Meet your group at restaurant (7 PM)
   ├─> Enjoy dinner and conversations!
   │
   ▼
Next Day
   │
   └─> Feedback request - rate the experience
```

---

## 📊 The 4 Workflows

### Workflow 5: Interest Collection (`5_Dinner_Interest_Collection.json`)

**Purpose**: Collect weekly RSVPs and manage signups

**Triggers**:
- **Monday 9 AM**: Send dinner invitations
- **Webhook**: Handle RSVP form submissions
- **Discord Command**: `/dinner-rsvp` for quick signup
- **Tuesday 5 PM**: Close RSVPs and trigger group formation

**Actions**:
1. Send invitation to Discord channel
2. DM users who opted into dinners
3. Collect RSVP form data (day preference, location, dietary needs)
4. Store in Airtable Dinner_RSVPs table
5. Send confirmation DM

**Airtable Tables Used**:
- Users (read: who opted into dinners)
- Dinner_RSVPs (write: store signups)

---

### Workflow 6: Balanced Group Formation (`6_Dinner_Group_Formation.json`)

**Purpose**: Create diverse, balanced groups using custom algorithm

**Triggers**:
- Workflow 5 (Tuesday 5 PM)
- Manual execution

**Core Algorithm Logic**:

```javascript
// Balancing Factors (with scoring weights):

1. Industry Diversity (20 points)
   - Different industries = higher diversity
   - Avoid 2+ people from same industry

2. Experience Level Mix (15 points)
   - Mix junior/mid/senior levels
   - 5+ years difference = max points

3. Company Diversity (10 points)
   - Different companies preferred

4. Dietary Compatibility (20 points)
   - Group people with compatible restrictions
   - Ensure restaurant can accommodate all

5. Shared Interests (10 points)
   - Some overlap for conversation starters

6. Location Preferences (10 points)
   - Prefer shared geographic areas

7. Avoid Recent Pairings (-50 points penalty)
   - Don't match same people within 4 weeks
```

**Algorithm Steps**:
1. Split RSVPs by day preference (Wednesday/Thursday)
2. For each day, create groups of 5-6:
   - Start with random seed person
   - Add people who maximize diversity scores
   - Use some randomness (pick from top 3 candidates)
3. Handle remainders (distribute or create smaller group if 3+)
4. Create Dinner_Events records in Airtable
5. Update RSVPs with assigned event
6. Send group assignment DMs to all attendees

**Airtable Tables Used**:
- Dinner_RSVPs (read: this week's signups)
- Users (read: user profiles for diversity data)
- Dinner_Events (read: recent events to avoid repeats, write: new events)

---

### Workflow 7: Reminders & Confirmations (`7_Dinner_Reminders.json`)

**Purpose**: Assign locations and send reminders to attendees

**Triggers**:
- **Tuesday 9 AM**: For Wednesday dinners
- **Wednesday 9 AM**: For Thursday dinners
- **Wednesday/Thursday 3 PM**: Day-of check-in

**Actions**:

**Day Before (9 AM)**:
1. Fetch tomorrow's dinner events
2. Assign restaurant location based on preferences
3. Update event with location details (name, address, map link)
4. Mark event status as "Confirmed"
5. Send detailed reminder DM to each attendee:
   - Location name and address
   - Map link
   - Time (7 PM)
   - Tips for great dinner
6. Create Discord group thread for coordination
7. Store thread ID in event

**Day Of (3 PM)**:
1. Fetch today's dinners
2. Post final reminder in group thread
3. Encourage confirmations

**Airtable Tables Used**:
- Dinner_Events (read: upcoming events, write: location & thread ID)
- Dinner_RSVPs (read: attendees)
- Users (read: Discord IDs for DMs)

---

### Workflow 8: Post-Dinner Feedback (`8_Dinner_Feedback.json`)

**Purpose**: Collect feedback and analyze trends

**Triggers**:
- **Thursday 10 AM**: For Wednesday dinners
- **Friday 10 AM**: For Thursday dinners
- **Webhook**: Feedback form submissions
- **Sunday 10 AM**: Weekly analysis

**Actions**:

**Feedback Collection (Day After)**:
1. Fetch yesterday's dinners
2. Get all attendees for each event
3. Send feedback form DM to each person
4. On submission:
   - Save to Dinner_Feedback table
   - Mark RSVP as "Attended"
   - Send thank you DM
5. When all attendees submit, mark event "Completed"

**Weekly Analysis (Sunday)**:
1. Aggregate last week's feedback
2. Calculate average ratings:
   - Overall experience
   - Group dynamics
   - Conversation quality
   - Location rating
3. Compute engagement metrics:
   - "Would attend again" rate
   - "Made connections" rate
4. Identify popular topics discussed
5. Compile favorite moments and suggestions
6. Send admin report with insights

**Airtable Tables Used**:
- Dinner_Events (read: recent dinners, write: status)
- Dinner_RSVPs (read: attendees, write: attended flag)
- Users (read: Discord IDs)
- Dinner_Feedback (write: feedback data, read: for analysis)

---

## 🗄️ Airtable Schema

See `DINNER_SCHEMA.md` for complete details. Key tables:

### Dinner_Events
Stores each dinner instance with location, date, attendees.

### Dinner_RSVPs
Tracks who signed up, their preferences, assignment status.

### Dinner_Feedback
Post-dinner ratings and comments from attendees.

### Dinner_Locations (Optional)
Pre-approved restaurant database with ratings.

---

## ⚙️ Configuration

### Weekly Schedule

**Default Timeline**:
- **Monday 9 AM**: Invitations sent
- **Tuesday 5 PM**: RSVP deadline, groups formed
- **Tuesday Evening**: Group assignments sent
- **Wed/Thu 9 AM**: Location details sent (day before)
- **Wed/Thu 3 PM**: Final reminder (day of)
- **Wed/Thu 7 PM**: Dinners happen!
- **Thu/Fri 10 AM**: Feedback requests sent
- **Sunday 10 AM**: Weekly analysis

### Customization Options

**Change Dinner Days**:
- Edit Workflow 6 to use different days
- Update Workflow 7 schedule triggers
- Modify date calculations in code nodes

**Adjust Group Size**:
- Default: 5-6 people
- Edit `targetSize` in Workflow 6 algorithm
- Minimum: 3 people
- Maximum: 8 people (affects conversation quality)

**Modify Balancing Weights**:
- Edit `diversityScore()` function in Workflow 6
- Current weights:
  ```javascript
  Industry diversity: 20 points
  Experience mix: 15 points
  Company diversity: 10 points
  Dietary compatibility: 20 points
  Shared interests: 10 points
  Location overlap: 10 points
  Recent pairing penalty: -50 points
  ```

**Add Custom Locations**:
1. Populate Dinner_Locations table with approved restaurants
2. Modify Workflow 7 "Assign Location" node to query this table
3. Filter by area preferences and dietary restrictions

---

## 📍 Location Management

### Default Behavior
Workflow 7 randomly assigns from a hardcoded list of 3 restaurants.

### Recommended Enhancement
1. Create **Dinner_Locations** table in Airtable
2. Add restaurants with:
   - Name, address, area
   - Cuisine type
   - Price range
   - Group-friendly (boolean)
   - Dietary accommodations
3. Update Workflow 7 to:
   - Query locations matching group's preferred areas
   - Filter by dietary restrictions compatibility
   - Rotate locations to avoid repeats
   - Consider past ratings

### Sample Location Entry
```json
{
  "Name": "Pizzeria Delfina",
  "Address": "3611 18th St, San Francisco, CA 94110",
  "Area": "Mission",
  "Cuisine Type": ["Italian"],
  "Price Range": "$$",
  "Group Friendly": true,
  "Max Group Size": 8,
  "Dietary Options": ["Vegetarian", "Gluten-Free"],
  "Notes": "Great for groups, reservations recommended"
}
```

---

## 🎨 Customization Ideas

### 1. Themed Dinners
Add themes to make dinners more focused:
- "Founder Fridays" - entrepreneurs only
- "Tech Talks" - discuss latest tech trends
- "Career Chat" - professional development
- "Random Fun" - no work talk allowed

**Implementation**:
- Add "Theme" field to RSVP form
- Filter RSVPs by theme in Workflow 6
- Create separate groups per theme

### 2. Price Tiers
Let users choose budget level:
- $ - Casual ($15-25 per person)
- $$ - Mid-range ($25-40 per person)
- $$$ - Upscale ($40+ per person)

**Implementation**:
- Add "Price Preference" to RSVP form
- Group people with similar preferences
- Filter locations by price tier

### 3. Recurring Groups
Some users might want to meet the same group monthly:
- "Standing Dinner" option in RSVP
- Track affinity groups
- Schedule recurring events

### 4. Waitlist System
Handle overflow when > 6 people per day:
- Mark extra RSVPs as "Waitlist"
- Auto-promote if cancellations
- Notify waitlist on Fridays for next week

### 5. Sponsor/Host Rotation
Designate one person per group to:
- Make reservation
- Greet newcomers
- Facilitate introductions

**Implementation**:
- Rotate based on attendance count
- Give hosts a special role/badge
- Small perk (discount, priority RSVP)

---

## 🚨 Troubleshooting

### Issue: Not Enough RSVPs

**Problem**: < 5 people sign up for a day

**Solutions**:
1. Lower minimum group size to 3-4
2. Combine Wednesday + Thursday into one dinner
3. Send reminder on Monday afternoon
4. Offer incentives (first-time free drink)
5. Highlight past successes in invitations

### Issue: Too Many RSVPs

**Problem**: > 12 people for one day

**Solutions**:
1. Create multiple groups for same day
2. Add Friday as third option
3. Stagger time slots (6 PM and 8 PM)
4. Waitlist overflow for next week

### Issue: Dietary Restrictions Conflict

**Problem**: Group has incompatible restrictions

**Solutions**:
1. Choose universally accommodating restaurant
2. Split into 2 smaller groups
3. Let group vote on location via thread
4. Have admin manually reassign

### Issue: Last-Minute Cancellations

**Problem**: People cancel day-of, group < 3

**Solutions**:
1. Promote from waitlist immediately
2. Merge with another group if nearby
3. Reschedule to next week
4. Cancel if < 3 people, full refunds

### Issue: Location Assignment Fails

**Problem**: No restaurant matches preferences

**Solutions**:
1. Expand area search radius
2. Use default central location
3. Let group choose via Discord thread
4. Admin manually selects from Locations table

---

## 📈 Success Metrics

Track these KPIs in Airtable:

### Attendance Metrics
- **RSVP Rate**: % of opted-in users who RSVP each week
- **Show-up Rate**: % of RSVPs who actually attend
- **Repeat Rate**: % of attendees who return within 4 weeks

### Experience Metrics
- **Average Overall Rating**: Mean rating (1-5)
- **Group Dynamics Score**: How well group meshed
- **Conversation Quality**: Depth of discussions
- **Connection Rate**: % who "met interesting people"

### Engagement Metrics
- **Would Attend Again**: % who'd join another dinner
- **Feedback Response Rate**: % who submit feedback
- **Discord Thread Activity**: Messages per group thread

### Operational Metrics
- **Average Group Size**: Mean attendees per dinner
- **Cancellation Rate**: % of confirmed who cancel
- **Location Ratings**: Best/worst performing venues

---

## 🎓 Best Practices

### For Organizers

1. **Start Small**: Begin with 1 dinner per week, scale up
2. **Curate Locations**: Pre-vet restaurants for group-friendliness
3. **Set Expectations**: Clearly communicate everyone pays their own
4. **Monitor Threads**: Jump in if groups need help coordinating
5. **Iterate Quickly**: Use weekly feedback to improve immediately
6. **Recognize Regulars**: Thank repeat attendees, build community
7. **Share Stories**: Highlight success stories to encourage RSVPs

### For Participants

1. **RSVP Honestly**: Only sign up if you can commit
2. **Arrive on Time**: Others are waiting for you
3. **Be Present**: Put phone away, engage with group
4. **Ask Questions**: Be curious about others' stories
5. **Share Openly**: Don't monopolize, but do contribute
6. **Exchange Contacts**: Connect with people you click with
7. **Give Feedback**: Help improve future dinners

---

## 🔄 Integration with 1-on-1 Matches

The dinner feature complements the 1-on-1 virtual matching:

### Differences

| Aspect | 1-on-1 Matches | Small Group Dinners |
|--------|---------------|---------------------|
| **Format** | Virtual call | In-person dinner |
| **Size** | 2 people | 5-6 people |
| **Selection** | AI-powered compatibility | Balanced diversity |
| **Goal** | Deep 1-on-1 connection | Broad network exposure |
| **Frequency** | Continuous (every 6 hrs) | Weekly batches |
| **Scheduling** | Cal.com links | Fixed day/time |
| **Location** | Remote/global | Local/geographic |

### Synergies

1. **Progressive Relationship Building**:
   - Meet someone at dinner
   - Request 1-on-1 match later for deeper conversation

2. **Different Contexts**:
   - Dinners: See how people interact in groups
   - 1-on-1s: Have focused professional discussions

3. **Cross-Promotion**:
   - Mention dinners in 1-on-1 match DMs
   - Promote 1-on-1s at dinners

4. **Feedback Loop**:
   - Dinner feedback improves 1-on-1 algorithm
   - 1-on-1 matches can inspire dinner themes

---

## 🚀 Getting Started

### Quick Setup (30 mins)

1. **Extend Airtable** (10 mins):
   - Add fields to Users table (Dinner Interest, Dietary Restrictions)
   - Create Dinner_Events, Dinner_RSVPs, Dinner_Feedback tables
   - Create RSVP and Feedback forms

2. **Import Workflows** (10 mins):
   - Import workflows 5-8 to n8n
   - Update Airtable base IDs
   - Update form URLs

3. **Configure Locations** (5 mins):
   - Add 3-5 local restaurants to location list
   - Get addresses and map links
   - Note dietary accommodations

4. **Test Run** (5 mins):
   - Create test RSVPs in Airtable
   - Manually trigger Workflow 6
   - Verify groups formed correctly
   - Check Discord DMs sent

### First Real Week

**Monday**:
- Announce new dinner feature in Discord
- Explain how it works
- Encourage signups

**Tuesday**:
- Monitor RSVP count
- If < 5, send reminder
- Let Workflow 6 run at 5 PM

**Wednesday**:
- Check group threads for issues
- Confirm locations are set
- Be available for questions

**Thursday**:
- Repeat for Thursday groups

**Friday**:
- Collect feedback
- Thank participants
- Note improvements

**Sunday**:
- Review analytics
- Plan next week adjustments
- Share highlights with community

---

## 💡 Advanced Features

### 1. Auto-Reservations
Integrate with OpenTable API to auto-book tables:
- Workflow 7 makes reservations
- Confirm via email
- Add to event record

### 2. Expense Splitting
Integrate with Splitwise or Venmo:
- Create group expense
- Send split request after dinner
- Track who paid

### 3. Photo Sharing
Create shared album for each dinner:
- Google Photos or Dropbox link
- Post in Discord thread
- Build visual history

### 4. Leaderboard
Gamify attendance:
- Track dinner count per user
- Monthly "Most Social" award
- Badges for milestones (5, 10, 20 dinners)

### 5. Guest Invites
Let members bring +1:
- "Bring a Friend" option in RSVP
- Larger groups (7-8 people)
- Recruit new community members

---

## 📚 Resources

- **Main README**: Overall bot documentation
- **DINNER_SCHEMA.md**: Complete Airtable schema
- **AIRTABLE_SCHEMA.md**: Original 1-on-1 matching schema
- **ARCHITECTURE.md**: System architecture diagrams

---

**Version**: 1.0
**Last Updated**: 2025-11-02
**Works With**: Discord Interest Matching Bot v1.0
**Workflows**: 5-8 (Small Group Dinners)

Enjoy building community through food! 🍽️🎉
