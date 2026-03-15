# LLD: Meeting Scheduler

## 1. Requirements

- **Users** have **calendars**; **rooms** (optional) have availability.
- **Meeting:** title, organizer, attendees, start time, end time, room (optional); **recurring** (daily/weekly) optional.
- **Schedule:** Check **conflict** (no double-booking for same user or room); suggest **free slots** given duration and list of attendees (and optional room).
- **CRUD:** Create, update, cancel meeting; send invites/notifications.
- **Optional:** Time zones, working hours, buffer between meetings.

---

## 2. Core Classes

```text
User – userId, timeZone, workingHours (start, end)
Meeting – id, title, organizerId, attendeeIds[], startTime, endTime, roomId?, recurrence?
Calendar – userId; getMeetings(start, end) → List<Meeting>
MeetingService – createMeeting(organizerId, attendees, start, end, roomId?) → conflict check; add to all attendees’ calendars
  getFreeSlots(attendeeIds[], durationMinutes, dateRange) → List<Slot>
  suggestRooms(start, end, capacity?) → List<Room>
Room – roomId, capacity; Calendar (same as user)
```

---

## 3. Conflict Check

- For each attendee (and room): get meetings that **overlap** [start, end). Overlap: meeting1.start < meeting2.end && meeting2.start < meeting1.end. If any overlap, return "conflict" and list conflicting meetings.
- **Create:** Only if no conflict for any attendee and room.

---

## 4. Free Slots

- **Input:** attendeeIds[], duration, dateRange (e.g. one day).
- Get all meetings for all attendees in dateRange; merge and sort by start. **Busy intervals:** union of all (start, end). **Free slots:** gaps between busy intervals (and within working hours). Filter gaps where gap.end - gap.start >= duration; return list of (start, start+duration).
- **Optional:** Working hours: only consider 9–5 (or per user); merge working hours with free gaps.

---

## 5. Key Methods

```text
MeetingService.createMeeting(organizerId, attendeeIds, start, end, roomId?) → Meeting | ConflictError
MeetingService.getFreeSlots(attendeeIds, duration, dateRange) → List<Slot>
MeetingService.cancelMeeting(meetingId)
Calendar.getMeetings(start, end)
```

---

## 6. Design Patterns

- **Strategy:** Conflict resolution (reject vs suggest next slot). **Observer:** Notify attendees on create/update/cancel.
