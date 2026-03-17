# Patient Queue System - Functional Analysis Breakdown

## Feature Overview
A system that manages patient appointments by detecting doctor availability and automatically assigning available doctors to patients waiting in a queue.

---

## Epic 1: Patient Queue Management

### Title
Add, remove, and manage patients in the queue

### Objective
Provide a centralized queue to track all patients waiting for doctor consultations and allow administrative management of the queue.

### Context
- Patients arrive either in-person or via scheduled appointments
- Queue operates during clinic operating hours
- Integration point: Patient registration system, appointment booking system
- Related epics: Doctor Availability Detection, Automatic Patient-Doctor Assignment

### Personas
- **Patient**: Needs to know their position in queue and estimated wait time
- **Receptionist**: Adds patients to queue, manages queue visibility
- **Doctor**: Views next patient in queue
- **Admin**: Monitors and manages queue health

### Description
The queue management system maintains a list of patients waiting for doctor consultation. Patients can be added manually by receptionists or automatically via appointment bookings. The system maintains queue order based on arrival time (FIFO) and priority rules (emergency cases). Each queue entry tracks patient information, arrival time, priority level, and assigned status.

**Key Workflows:**
- Add patient to queue (manual entry or automatic from booking)
- Remove patient from queue (assigned to doctor or canceled)
- View current queue with patient details
- Prioritize emergency cases
- Update patient queue position

### Acceptance Criteria
1. **Given** a patient arrives at the clinic, **When** the receptionist searches for the patient and clicks "Add to Queue", **Then** the patient appears at the end of the queue with current timestamp and priority level set to "Normal"

2. **Given** a patient is in the queue, **When** they are assigned to a doctor, **Then** they are removed from the general queue and moved to the assigned doctor's consultation queue

3. **Given** an emergency patient arrives, **When** the receptionist marks them as "Emergency", **Then** they appear at the front of the queue regardless of arrival time

4. **Given** a queued patient cancels, **When** the system receives cancellation, **Then** the patient is removed from queue and status is updated to "Canceled"

5. **Given** the queue has multiple patients, **When** a user views the queue, **Then** they see all patients ordered by priority then arrival time with position numbers (1, 2, 3...)

6. **Edge Case**: Given a patient is in queue for >4 hours, **When** this occurs, **Then** system flags for admin review and generates notification

### Non-Functional Requirements
- Queue operations must complete within 200ms
- System must support up to 500 patients in queue simultaneously
- Queue data must be persisted in real-time (no data loss on system failure)
- Queue history must be maintained for 90 days for audit purposes
- Access control: Only authenticated clinic staff can modify queue
- All queue operations must be logged with user ID and timestamp

---

## Epic 2: Doctor Availability Detection

### Title
Track and detect doctor availability status in real-time

### Objective
Maintain real-time status of each doctor's availability to enable intelligent patient-to-doctor assignment.

### Context
- Doctors have varying schedules and break times
- Status can change frequently as consultations complete
- Integration point: Doctor schedules, consultation duration tracking
- Related epics: Patient Queue Management, Automatic Patient-Doctor Assignment

### Personas
- **Doctor**: Updates their availability status (available, on-break, offline)
- **System**: Automatically updates status based on consultation activity
- **Admin**: Monitors doctor availability and work distribution

### Description
The availability detection system tracks each doctor's real-time status. Status can be manually set by the doctor (break, offline) or automatically updated by the system (busy during consultation, automatically available when consultation ends). The system tracks availability windows, consultation history, and break schedules to make intelligent availability predictions.

**Key Workflows:**
- Doctor marks self as "On Break" / "Available" / "Offline"
- System automatically marks doctor as "In Consultation" when patient assigned
- System automatically reverts to "Available" when consultation ends
- Admin can override doctor status if needed
- System tracks doctor availability history

### Acceptance Criteria
1. **Given** a doctor finishes a consultation, **When** the consultation is marked complete, **Then** the doctor status automatically changes to "Available"

2. **Given** a doctor clicks "On Break", **When** they set a break duration, **Then** their status shows "On Break" and they don't receive new patient assignments until break ends

3. **Given** a doctor is unavailable, **When** patients are being assigned, **Then** this doctor is NOT included in assignment candidates

4. **Given** a doctor's status is "Available", **When** a system check occurs, **Then** the UI displays a green indicator and the doctor appears in available doctor list

5. **Given** system tracks doctor schedules, **When** scheduled work hours end, **Then** doctor status automatically changes to "Offline" and cannot accept new patients

6. **Edge Case**: Given a consultation exceeds expected duration by 50%, **When** this occurs, **Then** system alerts admin and doctor remains "Busy" until manually cleared

### Non-Functional Requirements
- Availability status updates must propagate within 500ms
- System must handle up to 200 doctors with concurrent status changes
- Availability history must be maintained for 30 days
- Real-time status queries must respond within 100ms
- System must detect stale status (no update >5 min) and flag for admin
- All status changes must be audited with timestamp and actor

---

## Epic 3: Automatic Patient-Doctor Assignment

### Title
Intelligently assign queued patients to available doctors based on criteria

### Objective
Automatically match patients from the queue to available doctors, optimizing utilization and reducing wait times.

### Context
- Assignment happens when doctors become available
- Rules consider: doctor specialization, patient condition, priority, doctor workload
- Integration point: Patient Queue Management, Doctor Availability Detection, Patient Medical Records
- Related epics: Queue Notifications

### Personas
- **Patient**: Receives assignment to a doctor
- **Doctor**: Receives next patient to consult
- **Admin**: Monitors assignment efficiency and can manually override assignments

### Description
The assignment engine continuously monitors for matches between queued patients and available doctors. When a doctor becomes available, the system searches the queue for the best matching patient based on configurable rules (specialization, priority, complexity). Once matched, the patient is assigned and both parties are notified. Assignment history is tracked for analytics.

**Key Workflows:**
- Monitor for available doctors
- Query patient queue with matching criteria
- Perform assignment operation
- Notify doctor and patient of assignment
- Track assignment in history

### Acceptance Criteria
1. **Given** a doctor's status changes to "Available" and queue has patients, **When** the assignment check runs, **Then** the highest-priority queued patient is assigned to this doctor within 1 second

2. **Given** multiple doctors are available and multiple patients in queue, **When** assignment occurs, **Then** patients are matched to doctors based on priority rules without assigning the same patient twice

3. **Given** a doctor specializes only in "Cardiology" and a patient needs "Neurology", **When** assignment is attempted, **Then** the patient is NOT assigned to this doctor

4. **Given** a patient is assigned to a doctor, **When** the assignment completes, **Then** patient status changes from "Queued" to "Assigned" and doctor status changes to "With Patient"

5. **Given** an assignment occurs, **When** the operation completes, **Then** both doctor and patient receive notification with meeting location/details

6. **Edge Case**: Given no available doctors when patient arrives, **When** a doctor becomes available later, **Then** the longest-waiting patient is prioritized for assignment

### Non-Functional Requirements
- Assignment algorithm must complete within 500ms
- System must handle up to 1000+ concurrent assignments per hour
- Assignment rules must be configurable without code changes
- System must ensure 100% accuracy (no duplicate or missed assignments)
- Assignment decision must be logged for audit and performance analysis
- System must support priority-based assignment (emergency > urgent > normal)
- Load balancing: Assign patients to balance doctor workload

---

## Epic 4: Queue Notifications

### Title
Notify patients and doctors of queue status changes and assignments

### Objective
Keep all stakeholders informed of queue position changes, assignments, and wait time updates.

### Context
- Multiple notification channels: SMS, In-app, Email, Display screens
- Timing is critical (real-time notifications for assignments)
- Integration point: Patient Queue Management, Doctor Availability Detection, Automatic Assignment
- Related epics: All other epics depend on notifications

### Personas
- **Patient**: Receives notifications about queue position, wait time, assignment
- **Doctor**: Receives notifications about new patient assignments
- **Admin**: Can configure notification preferences and channels

### Description
The notification system sends real-time alerts to patients and doctors based on queue events. Patients can receive notifications about their queue position, estimated wait time, and when they're assigned. Doctors receive notifications when a new patient is assigned. Notification channels can be SMS, email, in-app push, or clinic display screens. Preferences are configurable per user.

**Key Workflows:**
- Send patient added to queue notification
- Send queue position update (every 5 patients ahead removed)
- Send assignment notification to patient and doctor
- Send estimated wait time update
- Send cancellation notification

### Acceptance Criteria
1. **Given** a patient is added to queue, **When** the operation completes, **Then** patient receives notification (SMS or in-app) with queue position and estimated wait time

2. **Given** a doctor is assigned a patient, **When** assignment completes, **Then** doctor receives notification with patient name, reason for visit, and meeting location

3. **Given** a patient's queue position improves by 5 positions, **When** this occurs, **Then** patient receives notification with new position and updated wait time

4. **Given** patient has opted in to SMS notifications, **When** assignment occurs, **Then** patient receives SMS in addition to in-app notification

5. **Given** system cannot deliver notification (invalid number, offline), **When** this occurs, **Then** notification is queued for retry and logged as failed delivery

6. **Edge Case**: Given patient has not been notified in 30 minutes, **When** their wait time is known to be >1 hour, **Then** proactive wait time update notification is sent

### Non-Functional Requirements
- Notifications must be delivered within 1 second of triggering event
- System must support 10,000+ concurrent notification deliveries per hour
- SMS delivery must have 95%+ success rate within 5 minutes
- In-app notifications must be real-time (WebSocket-based or polling)
- Notification history must be maintained for 60 days
- Failed notifications must be retried with exponential backoff
- System must handle notification throttling (don't spam users)
- Support multiple languages for notification content

---

## Epic 5: Queue Reporting & Analytics

### Title
Generate reports and analytics on queue performance and doctor utilization

### Objective
Provide visibility into queue operations, wait times, doctor efficiency, and system performance for continuous improvement.

### Context
- Collect data from all other epics
- Support trend analysis and forecasting
- Integration point: All other epics (data source)
- Related epics: All other epics

### Personas
- **Admin**: Views dashboards and generates reports
- **Clinic Manager**: Uses analytics for staffing decisions
- **Doctor**: Can view their personal performance metrics

### Description
The reporting system collects metrics from all queue operations and provides dashboards and reports. Key metrics include: average wait times, doctor utilization rates, assignment accuracy, patient throughput, peak hours, and system reliability. Reports can be customized by date range, doctor, time period, and filtered by various criteria. Trend analysis helps identify patterns and improve operations.

**Key Workflows:**
- Collect queue and assignment metrics
- Generate real-time dashboard
- Generate periodic reports (daily, weekly, monthly)
- Export reports in PDF/Excel format
- Display performance trends

### Acceptance Criteria
1. **Given** admin accesses the dashboard, **When** the page loads, **Then** they see: average wait time today, total patients seen, average doctor utilization %, and current queue size

2. **Given** a date range is selected, **When** the report is generated, **Then** it shows: average wait times, peak hours, doctor performance (consultations/hour), patient satisfaction trends

3. **Given** admin wants to see individual doctor metrics, **When** they select a doctor, **Then** they see: consultations completed, average consultation duration, patient ratings, on-time arrivals

4. **Given** data for a report exists, **When** admin clicks "Export", **Then** report is generated in PDF or Excel format with all selected metrics and charts

5. **Given** historical data exists, **When** dashboard loads, **Then** trend graphs show 7-day and 30-day trends for key metrics

6. **Edge Case**: Given report requires data from 1+ million queue operations, **When** export is requested, **Then** report generates asynchronously and user receives download link via email

### Non-Functional Requirements
- Dashboard must load within 2 seconds
- Reports must generate within 30 seconds for standard date ranges
- System must maintain 365 days of historical data
- Analytics queries must not impact real-time queue operations
- Support up to 100 concurrent dashboard users
- Export must support PDF, Excel, CSV formats
- Data aggregation must be accurate to 99.99%
- Dashboard must be mobile-responsive

---

## Epic Dependencies

```
Patient Queue Management (Epic 1)
    ↓
Doctor Availability Detection (Epic 2) ← Required by Epics 3, 4
    ↓
Automatic Patient-Doctor Assignment (Epic 3)
    ↓
Queue Notifications (Epic 4) ← Depends on all above
    ↓
Queue Reporting & Analytics (Epic 5) ← Depends on all above
```

**Development Sequence Recommendation:**
1. Epic 1 (Patient Queue Management) - Foundation
2. Epic 2 (Doctor Availability) - Parallel with Epic 1
3. Epic 3 (Automatic Assignment) - After Epics 1 & 2
4. Epic 4 (Notifications) - After Epic 3
5. Epic 5 (Reporting) - After Epics 1-4 (can start early for data collection)
