# Monty Lookup Chat

A WhatsApp-integrated property lookup system for real estate developers, allowing customers to search and request floor plans, sales offers, brochures, and property information via a mobile-friendly interface.

## 🏗️ Project Overview

This application provides a hierarchical navigation system for browsing real estate properties across multiple developers. Users authenticate with their phone number and navigate through:
1. **Developers** - Select a real estate developer
2. **Projects** - Browse projects under that developer
3. **Categories** - Choose information type (Brochure, Availability, Financial Fact Sheet, Floor Plans, Sales Offers)
4. **Search** - For Floor Plans and Sales Offers, search specific items

## 🗄️ Database Structure

### Tables

**developers_new**
- `id` (uuid, primary key) - Auto-generated
- `name` (text, unique) - Developer name
- `created_at` (timestamptz) - Auto-generated

**projects**
- `id` (uuid, primary key) - Auto-generated
- `developer_id` (uuid, foreign key → developers_new.id) - Links to developer
- `name` (text) - Project name
- `created_at` (timestamptz) - Auto-generated

**items**
- `id` (uuid, primary key) - Auto-generated
- `project_id` (uuid, foreign key → projects.id) - Links to project
- `name` (text) - Item name (searchable)
- `type` (text) - Item type: `floor_plan`, `sales_offer`, `brochure`, `availability`, `financial_fact_sheet`
- `created_at` (timestamptz) - Auto-generated

### Cascade Deletes
- Deleting a developer → deletes all projects and items
- Deleting a project → deletes all items

## 🎨 UI Navigation Flow

```
Phone Authentication
    ↓
Developer Selection (5 buttons + All Availability button)
    ├── All Availability (SALMON) → Bulk webhook request
    └── [Developer buttons] (SAGE GREEN) → Navigate to projects
    ↓
Project Selection (buttons for each project + Company Profile button)
    ├── Company Profile (SALMON) → Direct webhook
    └── [Project buttons] (SAGE GREEN) → Navigate to categories
    ↓
Category Selection (5 buttons)
    ├── Brochure (SALMON) → Direct webhook
    ├── Availability (SALMON) → Direct webhook
    ├── Financial Fact Sheet (SALMON) → Direct webhook
    ├── Floor Plans (SAGE GREEN) → Search interface
    └── Sales Offers (SAGE GREEN) → Search interface
```

### Button Color Coding
- **Salmon (#E07A5F)** = Direct actions (stays on page, sends to webhook immediately)
- **Sage Green (#8B9A8D)** = Navigate deeper (goes to another level)
- **Gray (#999)** = Back navigation

### Loading States & Toast Notifications
All salmon action buttons provide visual feedback:
- Button shows "Sending..." during webhook call
- Toast notification appears at bottom after success: "✓ Request sent!"
- Toast appears in salmon color and auto-dismisses after 3 seconds
- Smooth slide-up animation

## 📊 Current Data

### Developers (5)
1. **Peace Homes**
   - Peace Lagoons 2

2. **SCC Vertex**
   - The Willows Residences

3. **Premier Choice**
   - Gateway

4. **Mill Hill**
   - Allegro Park
   - Allegro Residences

5. **Alaia Developments**
   - Chelsea Gardens

Each developer also has a "Company Information" project (hidden from UI, used for company profile storage).

## 🛠️ Tech Stack

- **Frontend**: React (via CDN, single HTML file)
- **Styling**: Vanilla CSS (embedded)
- **Database**: Supabase PostgreSQL
- **Authentication**: Phone number (localStorage)
- **Hosting**: Vercel
- **Webhook**: n8n
- **Version Control**: GitHub

## 🔗 Important Links

- **Production URL**: Check latest Vercel deployment
- **GitHub Repo**: https://github.com/playingmontes/monty-lookup-chat
- **Supabase Project ID**: jfimvpvwpyrttiitvdbr
- **n8n Webhook**: https://thomasmccone.app.n8n.cloud/webhook/montychat

## 📁 File Structure

```
monty_lookup_chat/
├── index.html                    # Main application (React + CSS)
├── manifest.json                 # PWA manifest
├── sw.js                         # Service worker
├── icon-192.png                  # App icon
├── icon-512.png                  # App icon (large)
├── README.md                     # This file
├── SQL_CHEAT_SHEET.md           # SQL quick reference
├── .vercelignore                # Vercel deployment ignore
├── .gitignore                   # Git ignore rules
└── .claude/
    └── PROJECT_CONTEXT.md       # Claude Code context

Excluded from Git/Vercel:
├── peace_lagoons_2_floors_plans_a/  # Floor plan images
├── peace_lagoons_2_floors_plans_b/  # Floor plan images
└── .venv/                           # Python virtual environment
```

## 🚀 Deployment

### Initial Setup
```bash
vercel --prod
```

### Subsequent Deployments
```bash
vercel --prod
```

Deployment automatically happens from the root directory. Files are excluded via `.vercelignore`.

## 💾 Database Management

### Supabase Dashboard Access
1. Go to https://supabase.com/dashboard
2. Select project `jfimvpvwpyrttiitvdbr`
3. Use "Table Editor" for visual editing or "SQL Editor" for queries

### Common Operations
See `SQL_CHEAT_SHEET.md` for detailed SQL queries for:
- Adding/deleting developers
- Adding/deleting projects
- Bulk adding items (floor plans, sales offers)
- Viewing data

## 📱 How Items Work

### Category Buttons (Always Visible)
Every project automatically shows 5 category buttons:
1. Brochure
2. Availability
3. Financial Fact Sheet
4. Floor Plans
5. Sales Offers

No database entries needed for buttons to appear.

### Direct Actions (Salmon Buttons)
Clicking Brochure, Availability, or Financial Fact Sheet:
- Sends webhook with format: `{project_name} - {category}`
- Example: "Gateway - Brochure"
- No items need to exist in database
- Button shows "Sending..." during request
- Success toast appears: "✓ Request sent!"

### Bulk Actions
Clicking "All Availability" on home page:
- Sends single webhook with `item_type: 'bulk_availability'`
- n8n compiles availability for all projects
- Button shows "Sending..." during request
- Success toast appears: "✓ Request sent!"

### Search Categories (Sage Green Buttons)
Clicking Floor Plans or Sales Offers:
- Navigates to search interface
- Users type to search items in database
- If no items exist, shows "No items available yet"
- Items must be added to database for search to return results

## 🔄 Typical Workflows

### Adding a New Developer + Project
1. Add developer to database (SQL)
2. Add project to database (SQL)
3. Project automatically appears in app with all 5 category buttons
4. Optionally add floor plans/sales offers for searchable items

### Adding Bulk Floor Plans
1. Prepare list of 200 floor plan names
2. Use Claude Code or SQL to bulk insert
3. Items immediately searchable in app

### Testing
1. Open app in browser
2. Enter phone number
3. Test "All Availability" bulk button (watch for "Sending..." and toast)
4. Navigate: Developer → Project → Category
5. Test salmon buttons (direct actions with loading/toast feedback)
6. Test sage green buttons (navigation to search)

## 🔐 Authentication Flow

1. User enters phone number
2. Stored in `localStorage` as `userPhoneNumber`
3. No actual SMS/OTP (simplified for internal use)
4. Developer/Project/Category selections also stored in `localStorage`
5. State persists across page reloads

## 📤 Webhook Payload

When user selects an item, sends POST to n8n webhook:

**Regular Item Selection:**
```json
{
  "item": "Peace Lagoons 2 - Floor Plan - B1001",
  "item_type": "floor_plan",
  "developer": "Peace Homes",
  "project": "Peace Lagoons 2",
  "phone_number": "+971501234567",
  "timestamp": "2025-01-31T12:34:56.789Z"
}
```

**Bulk Availability Request:**
```json
{
  "item": "All Availability",
  "item_type": "bulk_availability",
  "developer": null,
  "project": null,
  "phone_number": "+971501234567",
  "timestamp": "2025-01-31T12:34:56.789Z"
}
```

**Company Profile Request:**
```json
{
  "item": "SCC Vertex - Company Profile",
  "item_type": "company_profile",
  "developer": "SCC Vertex",
  "project": null,
  "phone_number": "+971501234567",
  "timestamp": "2025-01-31T12:34:56.789Z"
}
```

## 🎯 Future Enhancements

Potential improvements:
- Admin interface for adding developers/projects/items without SQL
- Actual SMS authentication via Supabase Auth
- Image uploads for floor plans
- Analytics dashboard
- Multi-language support
- Favorite/bookmark functionality

## 📝 Development Notes

### Key Design Decisions
- Single HTML file for simplicity (no build process)
- Phone auth without OTP (internal tool, trusted users)
- Category buttons always visible (better UX than conditional rendering)
- Search only for large collections (floor plans, sales offers)
- Salmon/sage green color coding for action clarity
- Loading states and toast notifications for all actions
- "All Availability" bulk request button on home page
- Company Profile at developer level (not project level)

### Why These Choices?
- **Single file**: Easy deployment, no build complexity
- **Simplified auth**: Internal tool, users are trusted
- **Always show categories**: Prevents user confusion about what's available
- **Salmon/sage color coding**: Contrasting colors - salmon for actions (stay), sage green for navigation (go deeper)
- **Loading feedback**: Users need to know when webhook requests are sent successfully
- **Bulk availability**: Single-click option for common use case (all project availability)
- **Supabase**: Managed PostgreSQL, good free tier, RLS support

## 🆘 Troubleshooting

**App not loading?**
- Check Vercel deployment status
- Verify Supabase project is not paused
- Check browser console for errors

**Items not appearing in search?**
- Verify items exist in database with correct `project_id`
- Check item `type` matches category (`floor_plan` or `sales_offer`)
- Try broader search terms

**Developers not showing?**
- Check `developers_new` table has entries
- Verify no JavaScript errors in console
- Clear localStorage and refresh

## 👥 Contributors

- Initial development: Claude Code + User
- Maintained by: McCone Properties team

## 📄 License

Proprietary - Internal use only for McCone Properties

---

For SQL quick reference, see `SQL_CHEAT_SHEET.md`
For Claude Code context, see `.claude/PROJECT_CONTEXT.md`
