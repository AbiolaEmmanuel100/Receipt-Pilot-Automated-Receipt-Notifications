# Receipt Pilot — Automated Receipt Notifications

**Event-driven automation | n8n · Supabase · REST API · SMTP · HTML Email**

Automating receipt notifications for business owners and customers, from database insertion to personalized email delivery.

## Project Overview

Receipt Pilot is an online receipt-generation application. I developed an n8n automation that connects its Supabase database to a notification workflow, enabling business owners to receive email alerts when new receipts are created.

I subsequently redesigned the owner notification email and extended the workflow to prepare personalized customer notifications through email, WhatsApp, or SMS, depending on the contact details available.

The project demonstrates practical workflow automation, API integration, database lookups, conditional logic, data transformation, and HTML email design.

## 1. The Business Problem

The original reporting system delivered a scheduled summary to a single email address. It did not provide immediate, personalized receipt notifications to individual business owners.

The goal was to create a more responsive system that would:

- Notify the correct business owner whenever a new receipt was recorded.
- Include useful transaction information in a professionally designed email.
- Prepare a separate receipt notification for the customer.
- Handle missing contact information without interrupting the owner's notification.

## 2. Solution Architecture

**Workflow:** Supabase → n8n → Business lookup → Owner notification → Customer notification

1. **Database trigger:** A Supabase Database Webhook fires when a new record is inserted into `public.receipts`.
2. **Authenticated webhook:** n8n receives the POST request through a webhook configured with header authentication.
3. **Data extraction:** The workflow extracts the receipt ID, owner ID, customer details, payment information, and timestamp.
4. **Validation:** Conditional nodes check that the required identifiers are present.
5. **Business lookup:** The workflow retrieves the business name using the receipt owner's `user_id`.
6. **Owner email lookup:** An HTTP Request node uses the Supabase Auth Admin API to retrieve the account's login email.
7. **Owner notification:** An SMTP node sends a branded HTML email to the correct owner.
8. **Customer notification:** The workflow checks the customer's contact details and chooses the available notification channel.

## 3. Redesigned Owner Email

I redesigned the owner notification using a visual reference, replacing the original plain white-and-teal layout with a softer, more polished design.

The updated email features:

- A soft teal background and rounded receipt card.
- A business initial and receipt number.
- Customer information and the transaction date.
- Date and time converted to `Africa/Lagos`.
- Payment method and payment status.
- Total amount, amount paid, and outstanding balance.

**Verification:** The redesigned email was tested on real receipt executions 98 and 101. Both executions succeeded.

## 4. Customer Receipt Notifications

I extended the workflow to support a separate customer notification path.

The workflow reads the customer's email address and phone number from the receipt record and applies the following rules:

| Customer contact details | Intended action |
|---|---|
| Email available | Send a branded receipt email containing item lines. |
| No email, phone available | Attempt WhatsApp, with SMS as a fallback. |
| No email or phone | Skip the customer notification while allowing the owner notification to proceed. |

The customer email branch has been published but still requires confirmation using a real receipt.

The Twilio WhatsApp and SMS steps have been built into the workflow structure but remain unconfigured. They are not yet available for production delivery.

## 5. Phone Number Normalization

Customer phone numbers may be stored in Nigerian local format, such as `0906…`, while messaging services generally require international formatting.

I implemented a transformation that converts a leading `0` to `+234` and leaves numbers already beginning with `+` unchanged.

This prepares the number for the messaging integration. It does not, by itself, verify that the number is valid or that a WhatsApp sender is approved.

## 6. Technology Stack

- **n8n:** Workflow orchestration, conditional routing, and automation logic.
- **Supabase Database Webhooks:** Event-driven triggers when receipts are inserted.
- **Supabase PostgreSQL:** Receipt records and business information.
- **Supabase Auth Admin API:** Retrieval of business owners' login email addresses.
- **HTTP Request node:** API integration.
- **SMTP and Gmail:** Transactional email delivery.
- **HTML and CSS:** Branded email layout and receipt presentation.
- **JavaScript expressions:** Field extraction, data transformation, and timezone handling.
- **Twilio integration steps:** Planned WhatsApp and SMS delivery, pending account and sender configuration.

## 7. Challenges and Solutions

### Webhook authentication and connectivity

I encountered a 404 caused by using the test URL when the workflow was not listening, followed by a 500 error because the webhook had no authentication credential attached. I corrected the configuration and verified a successful HTTP response from Supabase.

### Large database payloads

The webhook payload was approximately 340 KB because it included an embedded base64 business logo. I reduced the fields passed through the workflow to avoid carrying unnecessary data into later steps.

### Retrieving the correct owner email

The email field in `business_information` was often empty. I used the Supabase Auth Admin API to retrieve the account's login email instead.

### Timestamp consistency

The website's receipt time differed from the database creation time. I used the database's `created_at` value and converted it to Lagos time for the owner email.

### Draft and production management

I separated workflow editing from publishing, reviewed changes before publication, and distinguished simulated test data from real receipt executions.

### Independent notification paths

I designed the customer-notification logic so that missing customer contact details would not prevent the owner from receiving their notification.

## 8. Results and Verification

The project has demonstrated the following verified outcomes:

- **Real receipt processing:** Execution 92 processed receipt `RCT-000017`, associated with Damil Enterprise.
- **Owner email delivery:** The SMTP server accepted the email, and the owner confirmed receipt. The first message landed in spam.
- **Redesigned email:** Real receipt executions 98 and 101 completed successfully with the updated owner email design.
- **Webhook connectivity:** Supabase successfully reached the n8n endpoint after the initial configuration errors were corrected.

The customer email branch has been published but has not yet been verified with a real customer receipt. WhatsApp and SMS delivery remain pending Twilio configuration.

## 9. Security and Reliability

The workflow uses the receipt's owner identifier to retrieve the associated business and account details. Credentials are stored in n8n rather than embedded in the application.

Further security and reliability work is required:

- Rotate the webhook secret because it appeared in an earlier execution record.
- Securely remove or restrict access to affected execution records.
- Keep the elevated Supabase credential protected and limit its permissions where practical.
- Test missing business records, missing recipient emails, and API failures.
- Configure a custom sending domain with SPF and DKIM to improve email deliverability.
- Configure and test the Twilio sender before enabling customer WhatsApp or SMS notifications.

These are outstanding actions, not completed improvements.

## 10. Skills Demonstrated

- Event-driven workflow automation
- Webhook integration and authentication
- Supabase database operations
- REST API integration
- Authentication Admin API usage
- Conditional logic and workflow branching
- Data extraction and payload reduction
- Nigerian phone number normalization
- HTML email design and SMTP delivery
- Timezone conversion and timestamp handling
- Debugging and execution-log analysis
- Testing real transactions and documenting limitations

## Conclusion

This project demonstrates how I translate a business problem into a practical automation using n8n, Supabase, API integrations, and email delivery.

The result is a receipt-notification workflow designed to give business owners timely transaction information while establishing a foundation for customer notifications across multiple channels.

The project also reflects an important engineering practice: separating verified behavior from simulated tests and documenting the remaining work required before production use.
