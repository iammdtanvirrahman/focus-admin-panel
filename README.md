# Focus Admin Studio

A responsive, Bengali-first static prototype for managing the Focus education platform.

## Included in this prototype
- Dashboard overview and sample metrics
- Course catalog editor: name, category, level, cover preview, description, instructor, price/discount and draft/published label
- Class/lesson manager: recorded/live type, course assignment, duration, date/time, video/meeting URL and status
- Notes/materials library UI for PDF metadata and resource links
- Demo orders/purchases table, status labels, CSV export
- Demo student list/search, JSON export/import, light/dark mode
- Local browser persistence through `localStorage`

## Important: prototype only
This repository currently contains a frontend-only demo. Changes are stored in the current browser and do not sync across administrators or appear on the student site. PDF selection records file metadata only; it does not upload files to cloud storage. Order status changes are not payment processing. The live class section schedules links; it does not broadcast video.

**Do not use this static demo for real administrator accounts or real student/payment data.** It has no authentication/authorization, database, server-side file access control or payment verification. Do not put API secrets or payment keys into frontend code.

## Production integration plan
1. Add authenticated admin and student roles (e.g. Supabase Auth or Firebase Auth).
2. Store courses, modules, lessons, enrollments and orders in a database with row-level authorization.
3. Upload course covers and PDFs to private cloud storage; use signed/authorized downloads for paid content.
4. Connect recorded video hosting and a live-class provider (Zoom/Google Meet or a dedicated streaming provider).
5. Connect a supported payment gateway (e.g. SSLCommerz or bKash merchant integration) with server-side signature/status verification and idempotent webhook handling before enabling course access.
6. Add audit logs, backups, validation, rate limits and an admin-only deployment.

## Preview
Open `index.html` locally or serve this folder from an HTTPS static host. Before making it reachable from the internet, connect authentication and protect the admin route; GitHub Pages alone cannot secure an administrator panel.
