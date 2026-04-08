# Add Notifications for Resolved and Closed Tickets

This document describes how to add notifications when a ticket is marked **Resolved** or **Closed**. The goal is to let users (ticket creator/assignee) receive a notification when their ticket status changes to `RESOLVED` or `CLOSED`.

---

## 🔍 Current Notification Flow (Existing System)

The app already supports notifications via:
- `notif/api.php` which handles notification actions (e.g., mark as read)
- `notif/notification.php` which provides `NotificationManager` and methods like `getUnreadNotifications()`
- `admin/notification.php` which returns notification HTML for the dropdown

Notifications are likely stored in a `notifications` table with fields like:
- `id`
- `user_id` (recipient)
- `title`
- `message`
- `type` (string like `ticket_created`)
- `is_read`
- `created_at`

> ✅ Confirm your schema by checking the `notifications` table definition in your database.

---

## 🎯 Goal: Notify When Ticket Becomes RESOLVED / CLOSED

### Trigger points
Notifications should be created whenever a ticket’s `status` is updated to either:
- `RESOLVED`
- `CLOSED`

### Which users should receive notifications?
Common choices:
- The user who created the ticket (`created_by`)
- The assigned technician / staff (if the ticket has an `assigned_to` field)

---

## 🧩 Implementation Plan

### 1) Locate the ticket update flow
Find where ticket status gets updated in the codebase:
- Likely locations:
  - `admin/edit_ticket.php`
  - `admin/ticket.php` (if it handles updates)
  - `users/ticket.php` (if users can update status)

Search for queries like:
- `UPDATE tickets SET status =`
- `WHERE id =` (ticket id)

### 2) Add notification creation logic after status change
After executing the status update query, add a block like:

```php
// After updating ticket status...
if ($newStatus === 'RESOLVED' || $newStatus === 'CLOSED') {
    $recipientId = intval($ticket['created_by']); // or assigned user
    $title = $newStatus === 'RESOLVED' ? 'Ticket Resolved' : 'Ticket Closed';
    $message = "Ticket #{$ticketId} is now {$newStatus}.";

    $stmt = $pdo->prepare("INSERT INTO notifications (user_id, title, message, type, is_read, created_at) VALUES (?, ?, ?, ?, 0, NOW())");
    $stmt->execute([$recipientId, $title, $message, 'ticket_status_update']);
}
```

> ✅ Use `htmlspecialchars()` when outputting the notification text to avoid XSS.

### 3) Ensure `NotificationManager` returns new notifications
If `NotificationManager::getUnreadNotifications()` already pulls notifications from the table, it will automatically return these new entries.

If you want to customize the notification label in the dropdown, use a new type (e.g. `ticket_resolved`, `ticket_closed`) and adjust the display logic in `admin/notification.php`.

### 4) Add notification display logic (optional enhancement)
Update `admin/notification.php` so the unread notification list shows the correct icon and text for resolved/closed tickets. Example:

```php
case 'ticket_resolved':
    $icon = 'bi-check-circle';
    $badgeClass = 'text-success';
    break;
case 'ticket_closed':
    $icon = 'bi-lock';
    $badgeClass = 'text-secondary';
    break;
```

### 5) Testing Checklist
- [ ] Update ticket status to `RESOLVED` → Notification appears for ticket owner
- [ ] Update ticket status to `CLOSED` → Notification appears for ticket owner
- [ ] Notification dropdown properly shows new items
- [ ] Clicking a notification marks it as read (existing behavior)
- [ ] Verify correct recipients (ticket creator vs assigned user)

---

## 🏁 Optional Enhancements
- Add a `ticket_id` foreign key column to `notifications` to link directly to the ticket
- Add a `link` column to let notifications open the related ticket page
- Aggregate notifications and show count in badge (already existing)
- Add a `resolved_by` or `closed_by` mention in the message

---

If you share the exact ticket update code path (file and query), I can provide the exact snippet to insert for resolved/closed notifications.