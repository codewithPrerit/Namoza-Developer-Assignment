# Task 1 – GTM Event Schema

## Objective

The objective of this implementation is to establish a comprehensive event tracking framework for the OrthoNow website using Google Tag Manager (GTM) and Google Analytics 4 (GA4).

Currently, the website only tracks page views, making it difficult for the marketing team to understand user behaviour, identify booking funnel drop-offs, measure lead generation performance, and optimize paid advertising campaigns.

This implementation focuses on tracking all critical user interactions, including appointment bookings, phone call clicks, WhatsApp engagement, patient guide downloads, clinic page visits, and blog engagement. The collected data will support funnel analysis, audience creation, conversion tracking, and Google Ads optimization.

---

## 1. Complete GTM Event Schema 
| Event Name | Trigger Type | Key Parameters | GA4 Report / Audience |
|------------|--------------|----------------|-----------------------|
| booking_step_complete | Custom Event (dataLayer) | step_number, step_name, clinic_location, specialty | Funnel Exploration |
| booking_completed | Custom Event (dataLayer) | booking_id, clinic_location, specialty | Conversions Report |
| call_now_click | Click Trigger (`tel:` link) | phone_number, clinic_location, page_name | High Intent Users Audience |
| whatsapp_click | Click Trigger (`wa.me` link) | whatsapp_number, clinic_location, page_name | WhatsApp Engagement Audience |
| patient_guide_form_submit | Form Submission Trigger | patient_name, phone_number, guide_name | Lead Generation Report |
| patient_guide_download | Link Click Trigger (PDF) | guide_name, file_name, download_source | Downloads Report |
| clinic_page_view | Page View (Clinic Pages Only) | clinic_name, city, page_url | Clinic Performance Report |
| blog_scroll_25 | Scroll Depth Trigger (25%) | article_title, category, scroll_percentage | Content Engagement Report |
| blog_scroll_50 | Scroll Depth Trigger (50%) | article_title, category, scroll_percentage | Engaged Readers Audience |
| blog_scroll_75 | Scroll Depth Trigger (75%) | article_title, category, scroll_percentage | High Engagement Audience |
| blog_scroll_100 | Scroll Depth Trigger (100%) | article_title, category, scroll_percentage | Fully Engaged Readers Audience |

---

# 2. Booking Funnel Tracking

The appointment booking process consists of the following three steps:

1. **Select Clinic Location & Specialty**
2. **Enter Patient Details (Name, Phone Number, Preferred Date)**
3. **Confirm Booking**

Since the booking process is implemented as a custom multi-step form, Google Tag Manager (GTM) cannot automatically detect internal step transitions.

To accurately measure funnel progression and user drop-offs, the front-end developer should trigger a `window.dataLayer.push()` event immediately after a user successfully completes each booking step.

Google Tag Manager will listen for these custom events using **Custom Event Triggers** and send them to Google Analytics 4 (GA4). This enables Funnel Exploration reports to measure conversion rates and identify the exact step where users abandon the booking process.

---

# 3. dataLayer Push for Booking Form

The following JSON objects represent the custom events that the front-end developer should push into the `window.dataLayer` after each successful booking step.

## Step 1 – Location & Specialty Selected

```json
{
  "event": "booking_step_complete",
  "step_number": 1,
  "step_name": "location_specialty_selected",
  "clinic_location": "{{clinic_name}}",
  "specialty": "{{specialty_selected}}"
}
```

**GTM Trigger**

- Trigger Type: **Custom Event**
- Event Name: `booking_step_complete`
- Trigger Condition: `step_number equals 1`

## Step 2 – Patient Details Entered

```json
{
  "event": "booking_step_complete",
  "step_number": 2,
  "step_name": "patient_details_entered",
  "patient_name": "{{patient_name}}",
  "phone_number": "{{phone_number}}",
  "preferred_date": "{{preferred_date}}"
}
```

**GTM Trigger**

- Trigger Type: **Custom Event**
- Event Name: `booking_step_complete`
- Trigger Condition: `step_number equals 2`

## Step 3 – Booking Confirmed

```json
{
  "event": "booking_completed",
  "step_number": 3,
  "step_name": "booking_confirmed",
  "booking_id": "{{booking_id}}",
  "clinic_location": "{{clinic_name}}",
  "specialty": "{{specialty_selected}}"
}
```

**GTM Trigger**

- Trigger Type: **Custom Event**
- Event Name: `booking_completed`

---

# 4. GA4 Funnel Exploration

The booking funnel in Google Analytics 4 (GA4) should be configured using the following sequence of events:

| Funnel Step | GA4 Event |
|-------------|------------|
| Step 1 | booking_step_complete (step_number = 1) |
| Step 2 | booking_step_complete (step_number = 2) |
| Step 3 | booking_completed |

This Funnel Exploration helps identify the exact booking step where users abandon the appointment process. The marketing team can use this data to optimize the booking experience and improve conversion rates.

### Example Funnel

| Booking Stage | Users | Drop-off |
|--------------|------:|---------:|
| Step 1 Completed | 1000 | - |
| Step 2 Completed | 720 | 28% |
| Booking Completed | 430 | 40.3% |

---

# 5. Google Ads Conversion

## Conversion Action

**booking_completed**

### Why this conversion?

I would import **booking_completed** as the primary Google Ads conversion because it represents a successfully completed appointment booking, which is the main business objective.

Events such as **call_now_click**, **whatsapp_click**, and **patient_guide_download** indicate user interest but do not guarantee an appointment. Therefore, they should be used for audience creation and performance analysis rather than campaign optimization.

---

# 6. Developer Implementation Notes

The `window.dataLayer.push()` events for each booking step should be implemented by the **front-end developer** within the booking form's JavaScript logic.

Google Tag Manager (GTM) does not automatically detect internal state changes in a custom multi-step form. Instead, GTM listens for custom events that are pushed into the `window.dataLayer`.

### Implementation Flow

```text
User Completes Booking Step
        │
        ▼
Frontend JavaScript
(window.dataLayer.push)
        │
        ▼
Google Tag Manager
(Custom Event Trigger)
        │
        ▼
Google Analytics 4
(Event Recorded)
        │
        ▼
GA4 Funnel Exploration
        │
        ▼
Google Ads Conversion (booking_completed)
```

This implementation provides accurate funnel tracking, reliable conversion measurement, and enables the marketing team to identify user drop-offs between booking steps.