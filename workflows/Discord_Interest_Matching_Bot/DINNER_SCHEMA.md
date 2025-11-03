# Small Group Dinners - Airtable Schema Extension

## Overview
Extension to the Discord Interest Matching Bot for organizing weekly small group dinners. This is a **separate feature** from 1-on-1 virtual matches.

## New Tables

### Table 5: Dinner_Events

Stores individual dinner event instances (one per group per week).

| Field Name | Type | Description | Options/Format |
|------------|------|-------------|----------------|
| `Event ID` | Auto Number | Primary key | Auto-generated |
| `Week Of` | Date | Week identifier | Monday of the week |
| `Day` | Single Select | Dinner day | Wednesday, Thursday |
| `Date` | Date | Actual dinner date | Auto-calculated from Week Of + Day |
| `Time` | Single Line Text | Dinner time | e.g., "7:00 PM" |
| `Location Name` | Single Line Text | Restaurant/venue name | e.g., "Pizzeria Delfina" |
| `Location Address` | Single Line Text | Full address | |
| `Location Area` | Single Select | Neighborhood/area | Mission, SOMA, Marina, Downtown, etc. |
| `Location Link` | URL | Google Maps link | |
| `Group Size` | Number | Number of attendees | 5-6 |
| `Theme/Topic` | Single Line Text | Optional dinner theme | e.g., "Startups", "AI/ML", "Casual" |
| `Status` | Single Select | Event status | Planning, Confirmed, Completed, Cancelled |
| `Attendees` | Link to Record | Linked RSVPs | Links to Dinner_RSVPs |
| `Attendee Count` | Count | Number of RSVPs | Auto-count from Attendees |
| `Discord Thread ID` | Single Line Text | Group chat thread | For group coordination |
| `Created At` | Created Time | Event creation | Auto-generated |
| `Confirmed At` | Date | When finalized | |
| `Notes` | Long Text | Admin notes | Special accommodations, etc. |

---

### Table 6: Dinner_RSVPs

Tracks who's interested in attending dinners each week.

| Field Name | Type | Description | Options/Format |
|------------|------|-------------|----------------|
| `RSVP ID` | Auto Number | Primary key | Auto-generated |
| `User` | Link to Record | User attending | Links to Users table |
| `Week Of` | Date | Week signing up for | Monday of the week |
| `Preferred Day` | Single Select | Preference | Wednesday, Thursday, Either |
| `Preferred Areas` | Multiple Select | Location preferences | Mission, SOMA, Marina, Downtown, etc. |
| `Dietary Restrictions` | Multiple Select | Food restrictions | Vegetarian, Vegan, Gluten-Free, None, etc. |
| `Dietary Notes` | Long Text | Additional details | Allergies, specific needs |
| `Status` | Single Select | RSVP status | Interested, Assigned, Confirmed, Attended, Cancelled |
| `Assigned Event` | Link to Record | Assigned dinner | Links to Dinner_Events |
| `Confirmed` | Checkbox | User confirmed attendance | Boolean |
| `Attended` | Checkbox | Actually attended | Boolean |
| `Submitted At` | Created Time | When RSVP submitted | Auto-generated |
| `Reminder Sent` | Checkbox | Reminder sent | Boolean |
| `Topics of Interest` | Multiple Select | Conversation topics | Tech, Business, Hobbies, Career, etc. |

---

### Table 7: Dinner_Feedback

Post-dinner feedback from attendees.

| Field Name | Type | Description | Options/Format |
|------------|------|-------------|----------------|
| `Feedback ID` | Auto Number | Primary key | Auto-generated |
| `Dinner Event` | Link to Record | Which dinner | Links to Dinner_Events |
| `User` | Link to Record | Attendee | Links to Users |
| `Overall Rating` | Single Select | Dinner experience | 1 - Poor, 2 - Fair, 3 - Good, 4 - Great, 5 - Excellent |
| `Group Dynamics` | Single Select | How was the group | 1, 2, 3, 4, 5 |
| `Conversation Quality` | Single Select | Quality of discussions | 1, 2, 3, 4, 5 |
| `Location Rating` | Single Select | Venue quality | 1, 2, 3, 4, 5 |
| `Would Attend Again` | Single Select | Future interest | Yes - Definitely, Yes - Maybe, Not Sure, No |
| `Favorite Moments` | Long Text | Highlights | Free-form |
| `Suggestions` | Long Text | Improvements | Free-form |
| `Topics Discussed` | Multiple Select | What was talked about | Tech, Business, Career, Personal, Hobbies, etc. |
| `Met Interesting People` | Checkbox | Made connections | Boolean |
| `Submitted At` | Created Time | Feedback time | Auto-generated |

---

### Table 8: Dinner_Locations

Database of approved dinner venues (optional but recommended).

| Field Name | Type | Description | Options/Format |
|------------|------|-------------|----------------|
| `Location ID` | Auto Number | Primary key | Auto-generated |
| `Name` | Single Line Text | Venue name | |
| `Area` | Single Select | Neighborhood | Mission, SOMA, Marina, Downtown, etc. |
| `Address` | Single Line Text | Full address | |
| `Google Maps Link` | URL | Location link | |
| `Cuisine Type` | Multiple Select | Food type | Italian, Asian, Mexican, American, etc. |
| `Price Range` | Single Select | Cost level | $, $$, $$$, $$$$ |
| `Group Friendly` | Checkbox | Good for groups | Boolean |
| `Capacity` | Number | Max group size | 6-10 |
| `Notes` | Long Text | Special info | Reservations needed, noise level, etc. |
| `Times Used` | Count | Usage frequency | Linked to Dinner_Events |
| `Avg Rating` | Rollup | Average feedback | From Dinner_Feedback |
| `Active` | Checkbox | Currently using | Boolean |

---

## Updated Users Table Fields

Add these fields to the existing **Users** table:

| Field Name | Type | Description | Options/Format |
|------------|------|-------------|----------------|
| `Dinner Interest` | Checkbox | Wants to join dinners | Boolean |
| `Dinner Frequency` | Single Select | How often | Weekly, Bi-weekly, Monthly, Occasional |
| `Dietary Restrictions` | Multiple Select | Food restrictions | Vegetarian, Vegan, Gluten-Free, None, etc. |
| `Preferred Areas` | Multiple Select | Location preferences | Mission, SOMA, Marina, Downtown, etc. |
| `Total Dinners Attended` | Count | Attendance count | Linked to Dinner_RSVPs |
| `Last Dinner Date` | Rollup | Most recent dinner | Max date from Dinner_RSVPs |

---

## Table Relationships

```
Users (1) ────────── (Many) Dinner_RSVPs
                            │
                            │ (Many-to-One)
                            │
                            ▼
                     Dinner_Events (1) ────── (1) Dinner_Locations
                            │
                            │ (One-to-Many)
                            │
                            ▼
                     Dinner_Feedback
```

---

## Sample Data

### Sample Dinner Event
```json
{
  "Week Of": "2025-11-04",
  "Day": "Wednesday",
  "Date": "2025-11-06",
  "Time": "7:00 PM",
  "Location Name": "Pizzeria Delfina",
  "Location Address": "3611 18th St, San Francisco, CA 94110",
  "Location Area": "Mission",
  "Location Link": "https://maps.google.com/?q=Pizzeria+Delfina+Mission",
  "Group Size": 6,
  "Theme/Topic": "Tech & Startups",
  "Status": "Confirmed",
  "Attendees": ["recRSVP1", "recRSVP2", "recRSVP3", "recRSVP4", "recRSVP5", "recRSVP6"]
}
```

### Sample RSVP
```json
{
  "User": "recUser123",
  "Week Of": "2025-11-04",
  "Preferred Day": "Either",
  "Preferred Areas": ["Mission", "SOMA"],
  "Dietary Restrictions": ["Vegetarian"],
  "Dietary Notes": "",
  "Status": "Assigned",
  "Assigned Event": "recEvent456",
  "Confirmed": true,
  "Topics of Interest": ["Tech", "Career", "Hobbies"]
}
```

### Sample Feedback
```json
{
  "Dinner Event": "recEvent456",
  "User": "recUser123",
  "Overall Rating": "5 - Excellent",
  "Group Dynamics": "5",
  "Conversation Quality": "5",
  "Location Rating": "4",
  "Would Attend Again": "Yes - Definitely",
  "Favorite Moments": "Great discussion about AI trends, met a potential collaborator",
  "Suggestions": "Maybe start 30 min earlier",
  "Topics Discussed": ["Tech", "Career", "Personal"],
  "Met Interesting People": true
}
```

---

## Airtable Views to Create

### Dinner_Events Views
1. **This Week's Dinners** - Date is current week
2. **Upcoming Confirmed** - Status = "Confirmed", Date >= Today
3. **Planning** - Status = "Planning"
4. **By Location Area** - Grouped by Location Area
5. **Wednesday Dinners** - Day = "Wednesday"
6. **Thursday Dinners** - Day = "Thursday"

### Dinner_RSVPs Views
1. **This Week's RSVPs** - Week Of = current week
2. **Pending Assignment** - Status = "Interested"
3. **Confirmed Attendees** - Confirmed = true
4. **By Preferred Day** - Grouped by Preferred Day
5. **Dietary Restrictions** - Filter for dietary needs

### Dinner_Feedback Views
1. **Recent Feedback** - Last 30 days
2. **High Ratings** - Overall Rating >= 4
3. **Needs Improvement** - Overall Rating <= 2
4. **By Event** - Grouped by Dinner Event

---

## Setup Instructions

### Step 1: Update Existing Tables
1. Open your "Discord Interest Matching" base
2. Go to **Users** table
3. Add the new fields listed above (Dinner Interest, Dinner Frequency, etc.)

### Step 2: Create New Tables
1. Create **Dinner_Events** table with all fields
2. Create **Dinner_RSVPs** table with all fields
3. Create **Dinner_Feedback** table with all fields
4. Create **Dinner_Locations** table with all fields (optional)

### Step 3: Configure Relationships
1. Link **Dinner_RSVPs.User** to **Users** table
2. Link **Dinner_RSVPs.Assigned Event** to **Dinner_Events**
3. Link **Dinner_Events.Attendees** to **Dinner_RSVPs**
4. Link **Dinner_Feedback.Dinner Event** to **Dinner_Events**
5. Link **Dinner_Feedback.User** to **Users**
6. Link **Dinner_Events.Location** to **Dinner_Locations** (if using)

### Step 4: Create Forms
1. **Weekly RSVP Form** - For users to sign up each week
   - Fields: User (lookup), Preferred Day, Preferred Areas, Dietary Restrictions, Topics of Interest
2. **Dinner Feedback Form** - Post-dinner survey
   - Fields: All feedback fields

### Step 5: Add Sample Data
1. Add 2-3 sample locations to **Dinner_Locations**
2. Create a test dinner event for next week
3. Create sample RSVPs to test grouping

---

## Balancing Algorithm Considerations

The group formation algorithm should aim for:

1. **Diversity Metrics**:
   - Mix of industries (no more than 2 from same industry)
   - Mix of experience levels (junior/mid/senior)
   - Mix of backgrounds (different companies)
   - Gender balance where possible

2. **Constraints**:
   - Dietary restrictions compatibility
   - Location preference alignment
   - Day preference matching
   - Avoid repeat groupings (don't match same people twice in 4 weeks)

3. **Randomization**:
   - Unlike 1-on-1 matching, use more randomness
   - Primary goal: expose people to diverse perspectives
   - Secondary goal: interesting conversations

---

## Admin Workflow

### Weekly Timeline

**Sunday/Monday**:
- Send Discord message: "Small group dinners this week! RSVP by Tuesday"
- Collect RSVPs via Airtable form

**Tuesday EOD**:
- Close RSVPs
- Run group formation algorithm
- Assign people to Wednesday/Thursday groups
- Book restaurant reservations

**Wednesday Morning**:
- Send reminders to Wednesday group
- Share Discord group thread
- Confirm final attendance

**Thursday Morning**:
- Send reminders to Thursday group
- Share Discord group thread
- Confirm final attendance

**Friday-Sunday**:
- Send feedback requests
- Collect and analyze responses
- Plan next week's dinners

---

**Version**: 1.0
**Last Updated**: 2025-11-02
**Works With**: Discord Interest Matching Bot v1.0
