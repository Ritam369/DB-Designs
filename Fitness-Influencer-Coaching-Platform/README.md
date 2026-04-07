# Fitness Influencer Coaching Platform - DB Schema Summary

> Note: In the image of the designed DB, if the attributes their details aren't visible clearly the please open it in a new tab by right clicking on it. Or you can do just clone it an open in VS code for further clear image.

## Entities & Attributes

| Entity | Primary Key | Key Attributes | Description |
|---|---|---|---|
| `Trainer` | `trainer_id` | `name`, `email`, `phone`, `bio`, `specialization`, `profile_photo_url`, `onboard_date` | Stores coach profile and specialization details. One trainer can coach many clients and deliver multiple coaching activities. |
| `Client` | `client_id` | `name`, `email`, `phone`, `date_of_birth`, `gender`, `onboard_date`, `profile_photo_url`, `assigned_trainer_id` (FK) | Stores client identity and onboarding details. Also links each client to an assigned trainer for personalized coaching. |
| `PlanType` | `plan_type_id` | `type_name`, `description` | List of plan categories offered by the platform. |
| `Enrollment` | `enrollment_id` | `client_id` (FK), `plan_type_id` (FK), `status` | Connects clients to selected plan types and tracks whether enrollment is active, paused, or completed. |
| `Consultation` | `consultation_id` | `enrollment_id` (FK), `trainer_id` (FK), `client_id` (FK), `scheduled_at`, `duration_minutes`, `platform`, `status`, `notes` | Represents one scheduled trainer-client consultation session with timing and status information. |
| `CoachingPlan` | `coaching_plan_id` | `enrollment_id` (FK), `trainer_id` (FK), `client_id` (FK), `plan_name`, `description`, `duration_weeks`, `price`, `created_at` | Stores customized paid coaching plans created by trainers for individual clients. |
| `LiveSession` | `session_id` | `enrollment_id` (FK), `trainer_id` (FK), `title`, `scheduled_at`, `duration_minutes`, `platform`, `batch_size`, `status` | Represents live group coaching sessions hosted by trainers for enrolled clients. |
| `LiveSessionAttendance` | `attendance_id` | `session_id` (FK), `client_id` (FK), `joined_at`, `attended` | Tracks participation of each client in each live session. One row equals one client attendance record. |
| `Guidance` | `guidance_id` | `enrollment_id` (FK), `trainer_id` (FK), `client_id` (FK), `workout_plan`, `diet_plan`, `notes`, `created_at`, `updated_at` | Stores trainer-issued workout and diet guidance for a specific enrolled client. |
| `CheckIn` | `checkin_id` | `client_id` (FK), `submitted_at`, `week_number`, `energy_level`, `sleep_quality`, `notes`, `photos_url` | Weekly self-report submitted by clients to share wellbeing updates and progress signals. |
| `ClientProgress` | `progress_id` | `client_id` (FK), `checkin_id` (FK), `recorded_at`, `weight_kg`, `body_fat_percent`, `chest_cm`, `waist_cm`, `hips_cm`, `arms_cm`, `notes` | Stores measurable body metrics and progress snapshots, usually tied to check-ins over time. |
| `TrainerFeedback` | `feedback_id` | `checkin_id` (FK), `trainer_id` (FK), `client_id` (FK), `feedback_text`, `given_at` | Records trainer feedback provided in response to client check-ins. |
| `Payment` | `payment_id` | `client_id` (FK), `enrollment_id` (FK), `amount`, `payment_date`, `payment_method`, `status`, `transaction_id` | Captures payment transactions made by clients for specific enrollments. |
| `Review` | `review_id` | `client_id` (FK), `trainer_id` (FK), `rating`, `review_text`, `created_at` | Stores client ratings and written reviews for trainers. |

## Table Relationships

| Parent Table | Child Table | Relationship | FK Column | Description |
|---|---|---|---|---|
| `Trainer` | `Client` | 1 : many | `Client.assigned_trainer_id` | One trainer can be assigned to many clients. |
| `Client` | `Enrollment` | 1 : many | `Enrollment.client_id` | A client can enroll in multiple plan types over time. |
| `PlanType` | `Enrollment` | 1 : many | `Enrollment.plan_type_id` | Enrollements can be done by the plantype only |
| `Enrollment` | `Consultation` | 1 : 1 | `Consultation.enrollment_id` | Enrollment links to its consultation plan flow in schema design. |
| `Enrollment` | `CoachingPlan` | 1 : 1 | `CoachingPlan.enrollment_id` | Enrollment links to a personalized coaching plan record. |
| `Enrollment` | `LiveSession` | 1 : 1 | `LiveSession.enrollment_id` | Enrollment links to live session scheduling context. |
| `Enrollment` | `Guidance` | 1 : 1 | `Guidance.enrollment_id` | Enrollment links to guidance documents for that service journey. |
| `Trainer` | `Consultation` | 1 : many | `Consultation.trainer_id` | One trainer can conduct many consultations. |
| `Trainer` | `CoachingPlan` | 1 : many | `CoachingPlan.trainer_id` | One trainer can create many personalized coaching plans. |
| `Trainer` | `LiveSession` | 1 : many | `LiveSession.trainer_id` | One trainer can host many live sessions. |
| `Trainer` | `Guidance` | 1 : many | `Guidance.trainer_id` | One trainer can issue many guidance records. |
| `Client` | `CoachingPlan` | 1 : 1 | `CoachingPlan.client_id` | A coaching plan is directly tied to a specific client. (customised plan)|
| `Client` | `Guidance` | 1 : 1 | `Guidance.client_id` | Guidance record is directly tied to one client. |
| `Client` | `Consultation` | 1 : 1 | `Consultation.client_id` | Consultation record is directly tied to one client. |
| `LiveSession` | `LiveSessionAttendance` | 1 : many | `LiveSessionAttendance.session_id` | One live session has many attendance rows. |
| `Client` | `LiveSessionAttendance` | 1 : many | `LiveSessionAttendance.client_id` | One client can attend multiple live sessions. |
| `Client` | `CheckIn` | 1 : many | `CheckIn.client_id` | Clients submit multiple weekly check-ins over time. |
| `CheckIn` | `ClientProgress` | 1 : 1 | `ClientProgress.checkin_id` | Each client's progress snapshot is linked to that cllient's check-in event. |
| `Client` | `ClientProgress` | 1 : many | `ClientProgress.client_id` | One client can have many progress records across weeks/months. |
| `CheckIn` | `TrainerFeedback` | 1 : 1 | `TrainerFeedback.checkin_id` | Each check-in receive trainer feedback. |
| `Trainer` | `TrainerFeedback` | 1 : many | `TrainerFeedback.trainer_id` | One trainer can provide many feedback entries. |
| `Client` | `TrainerFeedback` | 1 : many | `TrainerFeedback.client_id` | One client can receive many feedback entries over their journey. |
| `Client` | `Payment` | 1 : many | `Payment.client_id` | A client can make multiple payments. |
| `Enrollment` | `Payment` | 1 : 1 | `Payment.enrollment_id` | Enrollment maps to its payment transaction record in this schema. |
| `Client` | `Review` | 1 : many | `Review.client_id` | A client can submit multiple trainer reviews. |
| `Trainer` | `Review` | 1 : many | `Review.trainer_id` | A trainer can receive reviews from multiple clients. |
