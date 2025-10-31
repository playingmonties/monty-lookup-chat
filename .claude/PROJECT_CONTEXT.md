# Project Context - Monty Lookup Chat

**For Claude Code AI Assistant**

This file helps maintain context across sessions. Read this when returning to the project after time away.

## Quick Summary

**What**: WhatsApp-integrated real estate property lookup system
**Who**: McCone Properties
**Tech**: React (CDN), Supabase PostgreSQL, Vercel hosting
**File**: Single `index.html` file with embedded React + CSS
**Database**: Hierarchical (developers → projects → items)

## Current Project State

### Database Schema
- **5 developers** currently active
- **8 projects** total (5 visible + 3 Company Information hidden)
- **898 items** (mostly Peace Lagoons 2 floor plans)

### Active Developers & Projects
1. Peace Homes
   - Peace Lagoons 2 (784 floor plans)
2. SCC Vertex
   - Willow Residences
3. Premier Choice
   - Gateway (NEW - recently added)
4. Mill Hill
   - Allegro Park
   - Allegro Residences
5. Alaia Developments
   - Chelsea Gardens

### Removed Developers
- **Emaar Properties** (removed with 24 projects, 47 items)
- **Ellington Properties** (removed with 39 projects, 77 items)

## Key Architectural Decisions

### Navigation Hierarchy
```
Level 1: Developer Selection
Level 2: Project Selection (+ Company Profile button)
Level 3: Category Selection (5 buttons always shown)
Level 4: Search (only for Floor Plans and Sales Offers)
```

### Category Types (5 total)
1. **Brochure** - Green button, direct webhook
2. **Availability** - Green button, direct webhook
3. **Financial Fact Sheet** - Green button, direct webhook (NEW)
4. **Floor Plans** - Purple button, goes to search
5. **Sales Offers** - Purple button, goes to search

### Color Coding System
- **Green** = Direct action (sends webhook, stays on page)
- **Purple** = Navigation (goes to next level)
- **Gray** = Back button

### Why This Design?
- All 5 category buttons ALWAYS visible (no database check needed)
- Green buttons work immediately without database items
- Purple buttons require items in database to return search results
- Company Profile moved to developer level (not project level)

## Important Technical Details

### Tables Structure
```
developers_new (id, name, created_at)
    ↓ (CASCADE DELETE)
projects (id, developer_id, name, created_at)
    ↓ (CASCADE DELETE)
items (id, project_id, name, type, created_at)
```

### Item Types
- `brochure` - Direct webhook item
- `availability` - Direct webhook item
- `financial_fact_sheet` - Direct webhook item (NEW)
- `floor_plan` - Searchable item
- `sales_offer` - Searchable item

### Authentication
- Phone number stored in localStorage (no OTP)
- Format: `+971501234567`
- Trusted users only (internal tool)

### State Persistence
- `userPhoneNumber` - Phone auth
- `selectedDeveloper` - Current developer (JSON)
- `selectedProject` - Current project (JSON)
- `selectedCategory` - Current category (string)

### Webhook Integration
- **Endpoint**: https://thomasmccone.app.n8n.cloud/webhook/montychat
- **Method**: POST
- **Payload**:
  ```json
  {
    "item": "Project Name - Category/Item",
    "item_type": "floor_plan",
    "developer": "Developer Name",
    "project": "Project Name",
    "phone_number": "+971501234567",
    "timestamp": "2025-01-31T12:34:56.789Z"
  }
  ```

## Recent Changes

### Latest Session (2025-01-31)
1. ✅ Moved Company Profile to developer level
2. ✅ Added Financial Fact Sheet category
3. ✅ Removed Emaar and Ellington developers
4. ✅ Added Premier Choice developer with Gateway project
5. ✅ Implemented color-coded buttons (green/purple/gray)
6. ✅ Restored search interface for Floor Plans/Sales Offers

### Evolution of Navigation
- **v1**: Search at all levels
- **v2**: Buttons for categories, search for items
- **v3**: Mixed - green buttons (direct), purple buttons (search)
- **Current**: Hierarchical with color-coded actions

## Common User Workflows

### User Journey (Typical)
1. Enter phone number → Stored
2. Select developer → Navigate to projects
3. Select project → Navigate to categories
4. Click category:
   - Green → Webhook sent, confirmation shown
   - Purple → Navigate to search screen
5. (If purple) Type to search → Select item → Webhook sent

### Admin Workflows
See `SQL_CHEAT_SHEET.md` for:
- Adding developers/projects
- Bulk adding floor plans (200+ items)
- Deleting projects
- Viewing database state

## Common Patterns to Remember

### Adding Projects
```sql
INSERT INTO projects (developer_id, name)
SELECT id, 'Project Name'
FROM developers_new
WHERE name = 'Developer Name';
```

### Bulk Adding Items
```sql
INSERT INTO items (project_id, name, type)
SELECT p.id, v.name, 'floor_plan'
FROM projects p,
(VALUES ('Floor Plan - A101'), ('Floor Plan - A102')) AS v(name)
WHERE p.name = 'Project Name';
```

### Pattern Recognition
- Floor plans typically: 100-500+ items per project
- Sales offers typically: 100-400 items per project
- Brochures/availability: No items needed in database
- Naming convention: `{Type} - {Unit} - {Project}`

## Files & Locations

### Main Application
- **Path**: `/Users/montyai/PycharmProjects/monty_lookup_chat/index.html`
- **Size**: ~26KB
- **Lines**: ~900 lines
- **Structure**: HTML → CSS → React JSX (all in one file)

### Components in index.html
1. `PhoneAuth` - Phone number entry
2. `DeveloperSelection` - Developer buttons
3. `ProjectSelection` - Project buttons + Company Profile
4. `ItemCategorySelection` - 5 category buttons
5. `ChatApp` - Main orchestrator

### Key State Management
```javascript
// In ChatApp component
const [isAuthenticated, setIsAuthenticated] = useState(false);
const [userPhone, setUserPhone] = useState('');
const [selectedDeveloper, setSelectedDeveloper] = useState(null);
const [selectedProject, setSelectedProject] = useState(null);
const [selectedCategory, setSelectedCategory] = useState(null);
const [searchQuery, setSearchQuery] = useState('');
const [results, setResults] = useState([]);
```

## Deployment

### Production
- **Platform**: Vercel
- **Command**: `vercel --prod`
- **URL**: Changes with each deployment
- **Exclusions**: See `.vercelignore` (floor plan images, .venv, etc.)

### Git Repository
- **Remote**: https://github.com/playingmontes/monty-lookup-chat
- **Branch**: master
- **Commit Convention**: Descriptive + Claude Code signature

## Known Quirks & Gotchas

1. **Company Information projects** exist in DB but hidden from UI (`WHERE p.name != 'Company Information'`)
2. **Floor plan folders** are excluded from Git (too large)
3. **No actual OTP** - phone auth is simulated
4. **localStorage persistence** - Users may need to clear cache when testing
5. **Search is debounced** - 300ms delay before querying
6. **Item type must match exactly** - `floor_plan` not `floorplan` or `floor-plan`

## Future Considerations

### Likely Next Requests
- Add more developers (Damac, Nakheel, etc.)
- Bulk upload 200+ floor plans per project
- Custom branding/styling (McCone Properties colors/logo)
- Admin interface for non-technical users

### Scalability Notes
- Current design supports unlimited developers/projects
- Search is limited to 10 results (configurable)
- No pagination implemented yet
- No image hosting (floor plans are just text references)

## Questions to Ask When Resuming

1. "Which developer/project are we working with?"
2. "Are you adding data or modifying the UI?"
3. "Do you need to bulk add items (and how many)?"
4. "Is this for testing or production?"

## Emergency References

### Supabase Dashboard
- **URL**: https://supabase.com/dashboard
- **Project ID**: jfimvpvwpyrttiitvdbr
- **Tables**: developers_new, projects, items

### Vercel Dashboard
- **Deployment URL**: Check latest in Vercel dashboard
- **Logs**: Available in Vercel UI

### GitHub Repo
- **URL**: https://github.com/playingmontes/monty-lookup-chat
- **Access**: User has push access

## Claude Code Hints

When user says:
- "Add 200 floor plans" → Use bulk INSERT with generate_series
- "New developer" → Ask for developer name, then project name
- "Delete Emaar" → Warn about CASCADE, verify first
- "UI changes" → Work in index.html, test locally before deploy
- "Not working" → Check Supabase status, check browser console
- "Make it match our company" → Ask for brand colors, logo

## Session Summary Template

Use this when documenting work:
```
## Session: [Date]

**Objective**: [What user wanted]

**Changes Made**:
- Added: [List]
- Modified: [List]
- Removed: [List]
- Deployed: [Yes/No]

**Database State After**:
- X developers
- Y projects
- Z items

**Next Steps**:
- [Pending tasks]
```

---

**Last Updated**: 2025-01-31
**Project Status**: Active development
**Ready for**: Adding more developers, bulk item uploads, UI customization
