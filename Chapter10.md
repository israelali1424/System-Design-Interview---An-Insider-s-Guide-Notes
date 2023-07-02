# Chapter 10: Design a Notification System

## Introduction

- A notification system alerts users with essential information like breaking news, product updates, events, offerings, etc.
- It supports different formats such as mobile push notifications, SMS messages, and Emails.
- Building a scalable system that sends millions of notifications daily is a complex task.

## Interview Requirements

- Understanding the notification system's specifics includes:
  1. Types of notifications (push notification, SMS, and email)
  2. Is it a real-time system? (Soft real-time, slight delays are acceptable under high workload)
  3. Supported devices (iOS devices, android devices, and laptop/desktop)
  4. What triggers notifications? (Client applications and server-side schedules)
  5. Users' opt-out option availability (Yes)
  6. Daily notification volume (10 million mobile push notifications, 1 million SMS messages, and 5 million emails)

## High-Level Design

- Supports different notification types: iOS push notification, Android push notification, SMS message, and Email.
- Notification types:
  - iOS push notification: Requires a provider to build and send notification requests to Apple Push Notification Service (APNS). Data includes a unique device token and a payload (a JSON dictionary).
  - Android push notification: Similar to iOS but uses Firebase Cloud Messaging (FCM).
  - SMS message: Utilizes third-party SMS services like Twilio, Nexmo, etc.
  - Email: Companies use commercial email services like Sendgrid, Mailchimp for better delivery rate and data analytics.
- Contact info gathering flow: Collects mobile device tokens, phone numbers, or email addresses when a user installs the app or signs up for the first time.
- Notification sending/receiving flow: Triggers notification events, builds notification payloads for third-party services, and delivers notifications to users.

## Design Challenges

- Single point of failure (SPOF): A single notification server can lead to SPOF.
- Hard to scale: Challenging to independently scale databases, caches, and different notification processing components.
- Performance bottleneck: The system could become overloaded when processing and sending resource-intensive notifications, especially during peak hours.

## Improved High-Level Design

- Move the database and cache out of the notification server.
- Add more notification servers and set up automatic horizontal scaling.
- Introduce message queues to decouple the system components.
- Notification servers functionalities:
  1. Provide APIs for services to send notifications. (These APIs are accessible internally or by verified clients to prevent spams)
  2. Carry out basic validations to verify emails, phone numbers, etc.
  3. Query the database or cache to fetch data needed to render a notification.
  4. Put notification data to message queues for parallel processing.


## Understanding the Notification System Design 

### Flow Overview

- The flow moves from left to right:
  - **Service 1 to N**: Represent different services that send notifications via APIs provided by notification servers.
  - **Notification servers**: Provide the functionalities like APIs for services to send notifications (only accessible internally or by verified clients to prevent spams), carrying out basic validations, querying the database or cache, and putting notification data to message queues for parallel processing.
  - **Cache & DB**: User info, device info, notification templates are cached. The DB stores data about users, notifications, settings, etc.
  - **Message queues**: Remove dependencies between components and serve as buffers for high volumes of notifications. Each notification type is assigned a distinct message queue.
  - **Workers**: Pull notification events from message queues and send them to the corresponding third-party services.
  - **Third-party services**: Already explained in the initial design.
  - **iOS, Android, SMS, Email**: User end-points where notifications are delivered.

### Design Deep Dive

- Deep dive into:
  - **Reliability**: Preventing data loss, handling possible duplication.
  - **Additional components and considerations**: Notification template, notification settings, rate limiting, retry mechanism, security in push notifications, monitoring queued notifications, and event tracking.

#### Reliability

- How to prevent data loss?
  - Persist notification data in a database and implement a retry mechanism.
- Will recipients receive a notification exactly once?
  - Although delivered exactly once most of the time, the distributed nature could result in duplicate notifications. To reduce the duplication, introduce a dedupe mechanism and handle each failure case carefully.

#### Additional Components and Considerations

- **Notification template**: A preformatted notification to create a unique notification by customizing parameters, styling, tracking links, etc.
- **Notification setting**: Allows users to control which notifications they want to receive.
- **Rate limiting**: Limit the number of notifications a user can receive to avoid overwhelming users.
- **Retry mechanism**: If a third-party service fails to send a notification, the notification is added to the message queue for retrying.
- **Security in push notifications**: Use AppKey and AppSecret to secure push notification APIs. Only authenticated or verified clients are allowed to send push notifications using these APIs.
- **Monitor queued notifications**: Monitor the total number of queued notifications. If the number is large, the notification events are not processed fast enough by workers and more workers are needed.
- **Event tracking**: Track notification metrics, such as open rate, click rate, and engagement to understand customer behaviors.

## Wrap Up

- Notifications keep users posted with important information. It could be a push notification about a favorite movie on Netflix, an email about discounts on new products, or a message about an online shopping payment confirmation.
- In this chapter, the design of a scalable notification system that supports multiple notification formats was described. This includes: push notification, SMS message, and email. The system uses message queues to decouple system components.
- Additional system features and optimizations include reliability measures, security implementations, tracking and monitoring, respecting user settings, and rate limiting.
