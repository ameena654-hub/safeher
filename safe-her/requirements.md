# Requirements Document

## Introduction

SafeHer is a mobile application designed to improve women's safety during emergency situations. The app provides a single, clean main screen with large, easily accessible buttons that can be operated quickly under stress. Core features include an emergency alert system, live location sharing, a fake incoming call simulator, and a shake-to-alert mechanism that works without opening the app.

## Glossary

- **App**: The SafeHer mobile application
- **User**: The person using the SafeHer application
- **Trusted_Contact**: A phone contact previously designated by the User to receive emergency alerts and location data
- **Emergency_Alert**: A pre-composed SMS or push notification containing the User's current location and a distress message, sent to all Trusted_Contacts
- **Location_Service**: The device's GPS and network-based location provider
- **Shake_Detector**: The background service that monitors the device accelerometer for a shake gesture
- **Fake_Call_Screen**: A full-screen UI that simulates an incoming phone call
- **Live_Location**: A real-time or periodically updated GPS coordinate shared with Trusted_Contacts
- **Contact_Role**: A label assigned to a Trusted_Contact by the User that describes the contact's relationship (e.g., "Family", "Friend", "Colleague")
- **Notification_Preference**: A per-contact setting that controls which alert types (SMS, push notification, or both) are sent to a given Trusted_Contact
- **Contact_Verification**: A process by which the App confirms that a Trusted_Contact's phone number is reachable by sending a one-time test message and recording acknowledgement
- **Primary_Contact**: A Trusted_Contact designated by the User to receive all alert types and location updates with highest priority

---

## Requirements

### Requirement 1: Emergency Alert

**User Story:** As a User, I want to trigger an emergency alert with one tap, so that my Trusted_Contacts are immediately notified of my distress and location.

#### Acceptance Criteria

1. THE App SHALL display a prominent red Emergency Alert button on the main screen.
2. WHEN the User presses the Emergency Alert button, THE App SHALL retrieve the User's current coordinates from the Location_Service within 5 seconds.
3. WHEN the User presses the Emergency Alert button, THE App SHALL send an Emergency_Alert containing the User's current coordinates and a distress message to all Trusted_Contacts.
4. IF the Location_Service is unavailable when the Emergency Alert button is pressed, THEN THE App SHALL send the Emergency_Alert with the last known coordinates and indicate that the location may be outdated.
5. IF no Trusted_Contacts have been configured, THEN THE App SHALL display a prompt instructing the User to add at least one Trusted_Contact before the alert can be sent.
6. WHEN the Emergency_Alert is successfully sent to all Trusted_Contacts, THE App SHALL display a confirmation message to the User.

---

### Requirement 2: Live Location Sharing

**User Story:** As a User, I want to share my live location with trusted contacts, so that they can monitor my whereabouts in real time.

#### Acceptance Criteria

1. THE App SHALL display a Share Location button on the main screen.
2. WHEN the User presses the Share Location button, THE App SHALL begin transmitting the User's Live_Location to all Trusted_Contacts at intervals no greater than 30 seconds.
3. WHEN live location sharing is active, THE App SHALL display a visible indicator on the main screen showing that sharing is in progress.
4. WHEN the User presses the Share Location button while sharing is active, THE App SHALL stop transmitting the Live_Location and remove the active sharing indicator.
5. IF the Location_Service becomes unavailable during an active sharing session, THEN THE App SHALL notify the User that location updates have been paused and resume sharing automatically when the Location_Service becomes available again.
6. THE App SHALL allow the User to set a sharing duration between 15 minutes and 8 hours, after which sharing stops automatically.

---

### Requirement 3: Fake Call

**User Story:** As a User, I want to simulate an incoming phone call, so that I can exit uncomfortable or unsafe situations discreetly.

#### Acceptance Criteria

1. THE App SHALL display a Fake Call button on the main screen.
2. WHEN the User presses the Fake Call button, THE App SHALL display the Fake_Call_Screen within 3 seconds.
3. THE Fake_Call_Screen SHALL display a caller name and photo that the User has previously configured, defaulting to "Mom" if no configuration exists.
4. WHEN the Fake_Call_Screen is displayed, THE App SHALL play an incoming call ringtone using the device's current ringer volume setting.
5. WHEN the User interacts with the Fake_Call_Screen accept action, THE App SHALL transition to a simulated in-call screen showing an active call timer.
6. WHEN the User interacts with the Fake_Call_Screen decline or end action, THE App SHALL dismiss the Fake_Call_Screen and return to the main screen.
7. THE App SHALL allow the User to configure a delay between 0 and 60 seconds before the Fake_Call_Screen appears, so the User can pocket the phone first.

---

### Requirement 4: Shake-to-Alert

**User Story:** As a User, I want shaking my phone to automatically trigger an emergency alert, so that I can call for help without unlocking or opening the app.

#### Acceptance Criteria

1. THE App SHALL provide a Shake_Detector that operates as a background service when enabled by the User.
2. WHEN the Shake_Detector is enabled and the device is shaken with an acceleration exceeding 2.5g for at least 500 milliseconds, THE App SHALL trigger the same Emergency_Alert as defined in Requirement 1.
3. WHEN the Shake_Detector triggers an Emergency_Alert, THE App SHALL display a system notification confirming the alert was sent.
4. THE App SHALL allow the User to enable or disable the Shake_Detector from the main screen.
5. WHEN the Shake_Detector is enabled, THE App SHALL display a visible indicator on the main screen.
6. IF the device battery level falls below 10%, THEN THE App SHALL notify the User that the Shake_Detector may be restricted by the operating system's battery optimization policies.

---

### Requirement 5: Trusted Contacts Management

**User Story:** As a User, I want to manage my list of trusted emergency contacts, so that the right people are notified and can track my location during an emergency.

#### Acceptance Criteria

**Adding and Removing Contacts**

1. THE App SHALL allow the User to add Trusted_Contacts by selecting from the device's native contact list.
2. WHERE the User denies contacts permission, THE App SHALL allow the User to add a Trusted_Contact by manually entering a name and phone number.
3. THE App SHALL allow the User to remove any previously added Trusted_Contact.
4. THE App SHALL support a minimum of 1 and a maximum of 10 Trusted_Contacts.
5. WHEN a Trusted_Contact is added or removed, THE App SHALL persist the updated list so it is available after the App is closed and reopened.
6. THE App SHALL display the current list of Trusted_Contacts on a dedicated Contacts screen accessible from the main screen.

**Editing Contact Details**

7. THE App SHALL allow the User to edit the name and phone number of any existing Trusted_Contact.
8. WHEN the User saves an edit to a Trusted_Contact, THE App SHALL validate that the phone number contains only digits, spaces, hyphens, parentheses, and an optional leading plus sign, and SHALL display an inline error if the format is invalid.
9. WHEN the User saves an edit to a Trusted_Contact, THE App SHALL persist the updated details immediately.

**Contact Roles**

10. THE App SHALL allow the User to assign a Contact_Role to each Trusted_Contact from a predefined list: "Family", "Friend", "Colleague", and "Other".
11. THE App SHALL display each Trusted_Contact's Contact_Role alongside the contact name on the Contacts screen.
12. WHEN no Contact_Role is assigned, THE App SHALL default the Contact_Role to "Other".

**Primary Contact Designation**

13. THE App SHALL allow the User to designate exactly one Trusted_Contact as the Primary_Contact.
14. WHEN the User designates a new Primary_Contact, THE App SHALL remove the Primary_Contact designation from the previously designated contact.
15. THE App SHALL display a distinct visual indicator next to the Primary_Contact on the Contacts screen.
16. IF the User attempts to remove the Primary_Contact while it is the only Trusted_Contact, THEN THE App SHALL display a warning and require the User to confirm the removal.

**Notification Preferences**

17. THE App SHALL allow the User to configure a Notification_Preference for each Trusted_Contact, selecting one or more of: SMS, push notification.
18. WHEN an Emergency_Alert is triggered, THE App SHALL deliver the alert to each Trusted_Contact using only the delivery methods specified in that contact's Notification_Preference.
19. WHEN Live_Location sharing is active, THE App SHALL send location updates to each Trusted_Contact using only the delivery methods specified in that contact's Notification_Preference.
20. IF a Trusted_Contact has no Notification_Preference configured, THEN THE App SHALL default to sending both SMS and push notification for that contact.

**Contact Verification**

21. THE App SHALL allow the User to initiate Contact_Verification for any Trusted_Contact by sending a one-time test message to that contact's phone number.
22. WHEN Contact_Verification is initiated, THE App SHALL send a test SMS to the Trusted_Contact's phone number containing a verification code and instructions to reply.
23. WHEN the Trusted_Contact replies with the correct verification code within 24 hours, THE App SHALL mark that contact as verified and display a verified indicator on the Contacts screen.
24. IF the verification reply is not received within 24 hours, THEN THE App SHALL mark the Contact_Verification as expired and allow the User to re-initiate verification.
25. THE App SHALL display the verification status (unverified, pending, verified, expired) for each Trusted_Contact on the Contacts screen.

**Alert and Location Update Delivery**

26. WHEN an Emergency_Alert is triggered, THE App SHALL send the alert to all Trusted_Contacts concurrently, not sequentially.
27. IF delivery of an Emergency_Alert to a Trusted_Contact fails, THEN THE App SHALL retry delivery up to 3 times at 30-second intervals before marking that contact's delivery as failed.
28. WHEN an Emergency_Alert delivery attempt completes, THE App SHALL display a per-contact delivery status (sent, failed) to the User on the alert confirmation screen.
29. WHEN Live_Location sharing is active, THE App SHALL include a link or mechanism in the initial alert that allows each Trusted_Contact to view the User's Live_Location updates.

---

### Requirement 6: Permissions and Privacy

**User Story:** As a User, I want the app to request only necessary permissions and be transparent about data use, so that I can trust it with sensitive information.

#### Acceptance Criteria

1. WHEN the App is launched for the first time, THE App SHALL request location permission, explaining that it is required for emergency alerts and location sharing.
2. WHEN the App is launched for the first time, THE App SHALL request contacts permission, explaining that it is required to select Trusted_Contacts.
3. IF the User denies location permission, THEN THE App SHALL disable the Emergency Alert, Share Location, and Shake-to-Alert features and display a message explaining that these features require location access.
4. IF the User denies contacts permission, THEN THE App SHALL allow the User to manually enter Trusted_Contact phone numbers as an alternative.
5. THE App SHALL store all Trusted_Contact data locally on the device and SHALL NOT transmit it to any external server.

---

### Requirement 7: Main Screen UI

**User Story:** As a User, I want a single, uncluttered main screen with large buttons, so that I can activate any feature quickly under stress.

#### Acceptance Criteria

1. THE App SHALL present all four primary feature buttons (Emergency Alert, Share Location, Fake Call, Shake-to-Alert toggle) on a single main screen without requiring scrolling.
2. THE App SHALL render each primary button with a minimum touch target size of 64×64 dp to ensure reliable activation under stress.
3. THE App SHALL use distinct colors for each primary button to allow recognition without reading labels.
4. THE App SHALL display each button with a recognizable icon and a short text label.
5. WHEN the App is launched, THE App SHALL display the main screen within 2 seconds on devices meeting the minimum hardware requirements.
