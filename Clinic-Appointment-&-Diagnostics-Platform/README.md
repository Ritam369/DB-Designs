# Clinic Appointment and Diagnostics Platform - DB Schema Summary

> Note: In the image of the designed DB, if the attributes & their details aren't clearly visible then please open it in a new tab by right clicking on it. Or you can do just clone it an open in VS code for further clear image.

## Entities & Attributes

| Entity | Primary Key | Key Attributes | Description |
|---|---|---|---|
| `Patient` | `patient_id` | `first_name`, `last_name`, `date_of_birth`, `gender`, `phone_number`, `email`, `address`, `created_at`, `updated_at` | Stores patient profile and contact information. This is the base entity for appointments, consultations, tests, and payments. |
| `Doctor` | `doctor_id` | `first_name`, `last_name`, `departments`, `bio`, `experience_years`, `qualification`, `phone_number`, `email`, `license_number`, `consultation_fee`, `created_at`, `updated_at` | Stores doctor professional details including department, qualifications, and current consultation fee. Doctors are linked to appointments, consultations, and report reviews. |
| `DoctorAvailability` | `availability_id` | `doctor_id` (FK), `day_of_week`, `start_time`, `end_time`, `is_available`, `created_at`, `updated_at` | Stores weekly doctor availability slots (day and time range). Used to manage when a doctor can accept appointments. |
| `Appointment` | `appointment_id` | `patient_id` (FK), `doctor_id` (FK), `appointment_datetime`, `status`, `reason_for_visit`, `booked_at`, `updated_at` | Represents a booked patient-doctor slot. It tracks booking lifecycle from scheduled to completion/cancellation. |
| `Consultation` | `consultation_id` | `appointment_id` (FK), `doctor_id` (FK), `consultation_datetime`, `doctor_fee`, `notes`, `diagnosis`, `follow_up_recommended`, `created_at`, `updated_at` | Captures actual clinical consultation details after appointment. `doctor_fee` is stored historically so past bills remain accurate even if fees change later. |
| `Test` | `test_id` | `consultation_id` (FK), `test_name`, `description`, `prescribed_date`, `status`, `fee`, `performed_in_clinic` (default `true`), `created_at`, `updated_at` | Stores diagnostic tests prescribed during consultation. `performed_in_clinic` indicates whether the test performed in this clinic and whether the test fee should be included in clinic billing. |
| `Report` | `report_id` | `test_id` (FK), `report_date`, `report_details`, `findings_summary`, `reviewed_by_doctor_id` (FK), `created_at`, `updated_at` | Stores diagnostic result output for a test and doctor review linkage. Used for clinical interpretation and patient follow-up. |
| `Payment` | `payment_id` | `consultation_id` (FK), `amount`, `payment_date`, `payment_method`, `status`, `notes`, `created_at` | Stores consultation payment transaction details. `amount` is calculated as doctor fee plus only in-clinic test fees for the consultation. |

## Table Relationships

| Parent Table | Child Table | Relationship | FK Column | Description |
|---|---|---|---|---|
| `Patient` | `Appointment` | 1 : many | `Appointment.patient_id` | One patient can book multiple appointments over time. |
| `Doctor` | `Appointment` | 1 : many | `Appointment.doctor_id` | One doctor can be associated with many appointments. |
| `Doctor` | `DoctorAvailability` | 1 : many | `DoctorAvailability.doctor_id` | One doctor can have multiple availability slots across different days/times. |
| `Appointment` | `Consultation` | 1 : 1 (optional) | `Consultation.appointment_id` | One appointment can result in at most one consultation record. |
| `Doctor` | `Consultation` | 1 : many | `Consultation.doctor_id` | A doctor can conduct many consultations. |
| `Consultation` | `Test` | 1 : many | `Test.consultation_id` | One consultation can prescribe one or more diagnostic tests. |
| `Test` | `Report` | 1 : 1 | `Report.test_id` | Each test has exactly one report record in this design. |
| `Consultation` | `Payment` | 1 : 1 | `Payment.consultation_id` | One consultation can have at most one payment record. |
| `Doctor` | `Report` | 1 : many | `Report.reviewed_by_doctor_id` | One doctor can review many diagnostic reports. |

> Payment Amount Formula
> `amount = Consultation.doctor_fee + SUM(Test.fee WHERE Test.consultation_id = Payment.consultation_id AND Test.performed_in_clinic = true)`
