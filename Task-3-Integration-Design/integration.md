# Task 3 – Integration Design

## Objective

Design an end-to-end integration for the OrthoNow consultation landing page with HubSpot CRM, Karix WhatsApp Business API, Google Tag Manager (GTM), Google Analytics 4 (GA4), and Google Ads. The solution should automatically create or update patient records, send a WhatsApp confirmation message within two minutes, and record a Google Ads conversion after every successful consultation form submission.

---

## 1. End-to-End Integration Architecture

When a patient submits the consultation form, the landing page first validates the form using JavaScript. Once validation is successful, the form data is sent to a backend API using an HTTP POST request.

The backend acts as the central integration layer and communicates with HubSpot CRM, Karix WhatsApp Business API, Google Analytics 4 (GA4), and Google Ads.

I would use the **HubSpot CRM API** instead of the native HubSpot form because the landing page is custom-built and requires complete control over validation, business logic, and third-party integrations.

The integration flow is:

Landing Page (HTML, CSS, JavaScript)
        │
        ▼
Backend API (Node.js / Express)
        │
        ├──► HubSpot CRM API
        │
        ├──► Karix WhatsApp Business API
        │
        └──► Google Analytics 4 (GA4)
                    │
                    ▼
        Google Ads Conversion Import

The backend first searches HubSpot using the patient's phone number before creating a contact. Since HubSpot does not deduplicate contacts by phone number by default (it primarily uses email), I would implement a custom phone-number lookup. If the phone number already exists, the existing contact is updated; otherwise, a new contact is created.

The contact will contain the following properties:

- Name
- Phone Number
- Clinic Preference
- Source = Google Ads – Consultation Landing Page
- Lead Status = New Enquiry

After the contact is successfully created or updated, the backend immediately calls the Karix WhatsApp Business API to send a confirmation message. Finally, the backend records the successful conversion in GA4 and imports it into Google Ads for campaign optimization.

---

## 2. Biggest Failure Point & Fallback Strategy

The biggest failure point in this integration is the HubSpot CRM API request. If the CRM is temporarily unavailable due to network issues, API rate limits, or server downtime, patient information could be lost before reaching the CRM.

To prevent data loss, I would implement a retry mechanism with exponential backoff on the backend. If the request still fails after multiple retries, the form submission would be stored in a temporary database or message queue for later processing. An alert would also be sent to the development team so the issue can be investigated immediately.

Another important consideration is HubSpot's default deduplication behaviour. HubSpot automatically deduplicates contacts based on email, not phone number. Since this landing page only collects a phone number, I would first search for an existing contact using the phone number before deciding whether to create a new contact or update an existing one. This prevents duplicate patient records and maintains CRM data quality.


---

## 3. WhatsApp SLA Monitoring

The WhatsApp confirmation message must be delivered within two minutes of a successful form submission. Delays can occur due to backend server failures, Karix API downtime, network latency, API rate limits, or message queue congestion.

To ensure the SLA is met, every form submission should generate a timestamp. The backend should log the time when the request is received, when the HubSpot operation completes, when the Karix API is called, and when the message delivery status is received.

If the total processing time exceeds two minutes or the WhatsApp API returns an error, an automated alert should be triggered using monitoring tools such as Slack, email notifications, or PagerDuty. A monitoring dashboard (for example, Grafana or Datadog) should continuously display API response times, delivery success rates, and failed message counts. This enables the team to quickly identify issues and maintain the required service-level agreement.


---

## Conclusion

This architecture provides a reliable, scalable, and production-ready integration between the landing page, HubSpot CRM, Karix WhatsApp Business API, Google Analytics 4, and Google Ads. It ensures accurate lead management, prevents duplicate contacts through custom phone-number lookup, delivers timely patient communication, and supports effective campaign optimization while maintaining high system reliability.