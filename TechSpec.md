# Technical Specification

## 1. Product Overview
Academic Control Center is a local-first web app for students to manage class schedules and homework in one frictionless interface. The app runs entirely in the browser and stores all data in `localStorage`.

## 2. Scope
### In Scope
- Task management for homework and exams
- Weekly fixed schedule display
- Today view and weekly toggle for schedules
- Kanban board with 3 statuses
- Add/Edit/Delete/Move task actions
- Validation, toast feedback, and confirmation dialogs

### Out of Scope
- Backend or server API
- Authentication
- Drag and drop interaction
- Cross-device sync
- Push notifications
- Complex calendar or recurring schedule engine

## 3. Data Model

### Task
```json
{
  "id": 1,
  "title": "อ่านบทที่ 3",
  "subject": "Math",
  "dueDate": "2026-04-24",
  "priority": "high",
  "status": "todo",
  "createdAt": 1713840000000
}
```

### Schedule
```json
{
  "id": 1,
  "subject": "Physics",
  "dayOfWeek": 1,
  "startTime": "09:00",
  "room": "B204"
}
```

## 4. Storage Design
- Use one `localStorage` key for app state, for example `academic-control-center-state`
- Store tasks and schedules as JSON arrays inside a single state object
- Read on app start, write after every mutation
- If the stored payload is missing or invalid, initialize from defaults

## 5. UI Requirements

### Schedule Module
- Show today's classes based on current weekday
- Provide a toggle to switch between Today and Weekly views
- Weekly view shows all fixed classes grouped by day

### Kanban Board
- Three columns: To Do, Doing, Done
- Sort tasks by `dueDate` ascending inside each column
- Show status controls on each card

### Task Card
- Display subject, task title, due date, priority, and status action buttons
- Highlight tasks due today or tomorrow with a soft red background and warning icon

### FAB
- A floating circular `+` button fixed to the bottom-right
- Opens the input modal for new task entry

### Input Modal
- Fields: subject, title, due date, priority
- Use a simple modal overlay
- Support submit and cancel

## 6. Business Logic

### Auto-Sort
- Re-render the Kanban board after every state change
- Sort by `dueDate` before rendering cards

### Validation
- `dueDate` must be today or in the future
- Required fields must not be empty
- Invalid data must not be saved
- On validation failure, call `add_error_case` in console and show a toast message

### Delete Safety
- Single delete requires one confirmation modal
- Batch delete requires a second confirmation step before execution

### Persistence
- Every Add/Edit/Delete/Move operation must immediately update `localStorage`

## 7. Suggested Module Breakdown
- `state.js` for load/save and state transitions
- `dateUtils.js` for due-date checks and formatting
- `renderSchedule.js` for today and weekly schedule views
- `renderKanban.js` for board and card rendering
- `modal.js` for create/edit dialog behavior
- `notifications.js` for toast and error logging

## 8. Non-Functional Constraints
- Must work offline in a single browser session
- Must remain simple and stable, with minimal moving parts
- Must not depend on frameworks unless later explicitly added

## 9. Acceptance Rules
- The app starts without a backend
- Data survives refresh through `localStorage`
- Task ordering is deterministic and auto-sorted
- Dangerous actions are gated by confirmation dialogs
- Invalid inputs are blocked and reported
