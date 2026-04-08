# Removing Specific Reports from `site_report.php`

This document describes the steps to remove the following report sections from `admin/site_report.php`:

- **Ticket Distribution**
- **Location Report**
- **Personnel Workload**
- **Urgent Tickets**

## 🍽 What to remove

### 1) UI Tabs / Menu Items
- Delete/remove the entries for these reports from the tab/menu list that controls the available report views.
- Typically this is an array or object that defines the report buttons (name, id, icon, description).

### 2) Tab/view rendering logic
- Remove the code that renders the HTML/JS blocks for these tabs.
- This can include:
  - `<div id="...">` containers for each report
  - dynamic rendering logic that switches views based on selected tab id
  - any initialization code that loads or prepares data for these reports

### 3) Backend data endpoints / AJAX handlers
- Remove or disable any backend code inside `site_report.php` (or related endpoints) that generates data specifically for these reports.
- This may include SQL queries, API routes, or `fetch()`/`$.ajax` handlers that are only used by those tabs.

### 4) JS code related to data fetching / chart rendering
- Remove any JS functions that are only used by the removed reports (e.g., fetching ticket distribution data, drawing charts for urgent tickets, etc.).

### 5) CSS / styling (optional)
- If there are any CSS rules scoped only to these report sections, clean them up too.

## ✅ Suggested Implementation Steps
1. **Find and remove the report definitions**
   - Search for the report IDs/labels: `2_ticket_distribution`, `4_location_report`, `6_personnel_workload`, `10_urgent_tickets`.
   - Remove those entries from the report selector structure.

2. **Remove rendering blocks**
   - Locate and delete any `<section>` / `<div>` blocks that are only used for the removed reports.

3. **Remove unused JS functions**
   - Search for functions named like `renderTicketDistribution`, `renderLocationReport`, `renderWorkload`, `renderUrgentTickets` (or similar) and delete them.

4. **Remove backend/data code**
   - Look for `if ($action === '...')` conditions used to serve those report data sets.
   - Remove those cases or ensure they are no longer invoked.

5. **Test**
   - Reload `admin/site_report.php` and verify only the remaining report tabs are visible.
   - Confirm no JS errors appear in the browser console.

---

## 🧪 Notes
- If any of the removed reports are used elsewhere, double-check for references in other JS files or API routes.
- If you want something kept but hidden temporarily, consider commenting out rather than deleting.

---

If you want me to apply the changes directly to `site_report.php`, let me know and I can modify the file in place as per this plan.