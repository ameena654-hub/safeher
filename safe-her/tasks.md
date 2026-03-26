# Implementation Plan: SafeHer

## Overview

Implement SafeHer as a single self-contained `safeher.html` file using vanilla HTML5, CSS3, and ES6+ JavaScript. All state persists to `localStorage`. Device APIs (Geolocation, DeviceMotionEvent, Web Notifications) are accessed directly from the browser. No build tools, no npm, no backend.

## Tasks

- [ ] 1. Scaffold the HTML file with screen structure and base styles
  - Create `safeher.html` with all five `<section>` screens: `#screen-main`, `#screen-contacts`, `#screen-fake-call`, `#screen-in-call`, `#screen-settings`
  - Add CSS reset, CSS custom properties for the color palette, and base layout styles
  - Render the four primary feature buttons on `#screen-main` with distinct colors, emoji icons, and text labels — each at least 64×64 px
  - Add a `<script>` block at the bottom for all JS modules
  - _Requirements: 7.1, 7.2, 7.3, 7.4, 7.5_

- [ ] 2. Implement StorageService and data model helpers
  - [ ] 2.1 Write `StorageService` with `save`, `load`, and `remove` wrapping `localStorage` with JSON serialization and try/catch for `QuotaExceededError`
    - Keys: `safeher_contacts`, `safeher_settings`, `safeher_last_location`, `safeher_alert_log`
    - _Requirements: 5.5, 5.9_

  - [ ]* 2.2 Write property test for contact list persistence round-trip (Property 9)
    - **Property 9: Contact list persistence round-trip**
    - **Validates: Requirements 5.5, 5.9**

- [ ] 3. Implement the Router / Screen Manager
  - Write `showScreen(id)` that hides all `<section>` elements and shows the target one
  - Wire navigation buttons (Contacts, Settings, Back) to `showScreen`
  - _Requirements: 7.1, 7.5_

- [ ] 4. Implement GeoService
  - [ ] 4.1 Write `GeoService` with `getCurrentPosition(timeoutMs)`, `getLastKnown()`, `watchPosition(cb)`, and `clearWatch(id)`
    - Cache last known position to `safeher_last_location` via `StorageService`
    - _Requirements: 1.2, 1.4, 2.2_

  - [ ]* 4.2 Write unit test for stale/null location fallback edge case
    - Simulate `getCurrentPosition` rejection and assert `getLastKnown()` is returned
    - _Requirements: 1.4_

- [ ] 5. Implement ContactsManager
  - [ ] 5.1 Write `ContactsManager` with `getContacts`, `addContact`, `updateContact`, `removeContact`, `setPrimary`, and `initiateVerification`
    - Enforce 1–10 contact limit; enforce single-primary invariant on `setPrimary`
    - Persist all mutations immediately via `StorageService`
    - Default `role` to `"Other"` and `notificationPrefs` to `['sms', 'push']` when not supplied
    - _Requirements: 5.3, 5.4, 5.5, 5.7, 5.9, 5.10, 5.12, 5.13, 5.14, 5.20_

  - [ ] 5.2 Write `validatePhone(input)` returning `{ valid: boolean, error?: string }`
    - Accept strings matching `^\+?[\d\s\-\(\)]+$` with at least one digit; reject all others
    - _Requirements: 5.8_

  - [ ]* 5.3 Write property test for contact capacity constraint (Property 10)
    - **Property 10: Contact capacity constraint**
    - **Validates: Requirements 5.4**

  - [ ]* 5.4 Write property test for phone number format validation (Property 11)
    - **Property 11: Phone number format validation**
    - **Validates: Requirements 5.8**

  - [ ]* 5.5 Write property test for exactly one primary contact invariant (Property 13)
    - **Property 13: Exactly one primary contact invariant**
    - **Validates: Requirements 5.13, 5.14**

  - [ ]* 5.6 Write property test for role default and constraint (Property 12)
    - **Property 12: Role default and constraint**
    - **Validates: Requirements 5.10, 5.12**

  - [ ]* 5.7 Write property test for contact list persistence round-trip (Property 9)
    - **Property 9: Contact list persistence round-trip**
    - **Validates: Requirements 5.5, 5.9**

- [ ] 6. Implement Contacts screen UI
  - [ ] 6.1 Render the contacts list on `#screen-contacts`: each card shows name, phone, role, primary indicator, and verification status badge
    - Wire the Add Contact form with inline phone validation error display
    - Wire Edit and Remove buttons; show warning modal when removing the sole primary contact
    - _Requirements: 5.1, 5.2, 5.3, 5.6, 5.7, 5.8, 5.11, 5.15, 5.16, 5.25_

  - [ ] 6.2 Wire role selector (Family / Friend / Colleague / Other) and notification preference checkboxes (SMS, Push) per contact
    - _Requirements: 5.10, 5.11, 5.17, 5.18, 5.19, 5.20_

  - [ ] 6.3 Wire "Verify" button to `ContactsManager.initiateVerification` and display pending/verified/expired badge
    - _Requirements: 5.21, 5.22, 5.23, 5.24, 5.25_

  - [ ]* 6.4 Write property test for contact card renders all required fields (Property 14)
    - **Property 14: Contact card renders all required fields**
    - **Validates: Requirements 5.11, 5.15, 5.25**

  - [ ]* 6.5 Write property test for verification state machine (Property 17)
    - **Property 17: Verification state machine**
    - **Validates: Requirements 5.21, 5.23, 5.24**

  - [ ]* 6.6 Write unit test for contacts permission denied → manual entry shown
    - _Requirements: 5.2_

  - [ ]* 6.7 Write unit test for remove last primary contact warning
    - _Requirements: 5.16_

  - [ ]* 6.8 Write unit test for verification code expired after 24h
    - _Requirements: 5.24_

- [ ] 7. Checkpoint — Ensure all tests pass, ask the user if questions arise.

- [ ] 8. Implement NotificationService
  - [ ] 8.1 Write `NotificationService` with `requestPermission()` and `send(contact, message)`
    - SMS: display a toast/modal showing the SMS body that would be sent
    - Push: fire `new Notification(...)` via the Web Notifications API
    - Respect `contact.notificationPrefs` — only invoke methods present in the array
    - _Requirements: 5.18, 5.19, 5.20_

  - [ ]* 8.2 Write property test for delivery respects notification preferences (Property 15)
    - **Property 15: Delivery respects notification preferences**
    - **Validates: Requirements 5.18, 5.19**

  - [ ]* 8.3 Write property test for default notification preference (Property 16)
    - **Property 16: Default notification preference**
    - **Validates: Requirements 5.20**

- [ ] 9. Implement EmergencyAlert module
  - [ ] 9.1 Write `EmergencyAlert.triggerAlert()` returning `Promise<AlertResult>`
    - Call `GeoService.getCurrentPosition(5000)`; on timeout/rejection fall back to `getLastKnown()` and set `locationStale = true`
    - Dispatch to all contacts concurrently via `Promise.allSettled`, respecting each contact's `notificationPrefs`
    - Retry failed deliveries up to 3 times at 30-second intervals via recursive `setTimeout`
    - Persist result to `safeher_alert_log` via `StorageService`
    - _Requirements: 1.2, 1.3, 1.4, 5.26, 5.27, 5.28_

  - [ ] 9.2 Wire the Emergency Alert button on `#screen-main` to `triggerAlert()`
    - Show "no contacts" prompt if contact list is empty
    - Show per-contact delivery status (✓ sent / ✗ failed) on confirmation screen
    - _Requirements: 1.1, 1.5, 1.6, 5.28_

  - [ ]* 9.3 Write property test for alert reaches every contact (Property 1)
    - **Property 1: Alert reaches every contact**
    - **Validates: Requirements 1.3, 5.26**

  - [ ]* 9.4 Write property test for stale-location fallback sets the staleness flag (Property 2)
    - **Property 2: Stale-location fallback sets the staleness flag**
    - **Validates: Requirements 1.4**

  - [ ]* 9.5 Write property test for retry count invariant (Property 18)
    - **Property 18: Retry count invariant**
    - **Validates: Requirements 5.27**

  - [ ]* 9.6 Write unit test for empty contacts guard
    - `triggerAlert()` with zero contacts must return an error state without calling `NotificationService`
    - _Requirements: 1.5_

- [ ] 10. Implement LocationSharing module
  - [ ] 10.1 Write `LocationSharing` with `startSharing(durationMs)`, `stopSharing()`, and `isActive()`
    - Use `navigator.geolocation.watchPosition` for continuous updates
    - Broadcast to contacts every ≤30 seconds via `NotificationService`
    - Include a viewer link/mechanism in the initial alert message body
    - Auto-stop after `durationMs` via `setTimeout`; clamp/reject durations outside [15, 480] minutes
    - _Requirements: 2.1, 2.2, 2.4, 2.6, 5.29_

  - [ ] 10.2 Wire the Share Location button to `startSharing` / `stopSharing`
    - Toggle `active` CSS class on `#btn-share` and update aria attributes to reflect sharing state
    - Show paused notification when `GeoService` becomes unavailable mid-session; resume automatically
    - _Requirements: 2.1, 2.3, 2.4, 2.5_

  - [ ]* 10.3 Write property test for location sharing interval invariant (Property 3)
    - **Property 3: Location sharing interval invariant**
    - **Validates: Requirements 2.2**

  - [ ]* 10.4 Write property test for sharing active indicator mirrors sharing state (Property 4)
    - **Property 4: Sharing active indicator mirrors sharing state**
    - **Validates: Requirements 2.3, 2.4**

  - [ ]* 10.5 Write property test for sharing duration validation (Property 5)
    - **Property 5: Sharing duration validation**
    - **Validates: Requirements 2.6**

  - [ ]* 10.6 Write property test for location sharing message contains viewer link (Property 19)
    - **Property 19: Location sharing message contains viewer link**
    - **Validates: Requirements 5.29**

  - [ ]* 10.7 Write unit test for location service unavailable during sharing
    - _Requirements: 2.5_

- [ ] 11. Implement FakeCall module and screens
  - [ ] 11.1 Write `FakeCall` with `scheduleFakeCall(delayMs)`, `acceptCall()`, `declineCall()`, and `endCall()`
    - Use `setTimeout` for configurable delay (0–60 s); reject values outside range
    - Play ringtone via Web Audio API (generated tone); fall back silently if audio fails
    - Manage call timer with `setInterval` on `#screen-in-call`
    - _Requirements: 3.2, 3.4, 3.5, 3.6, 3.7_

  - [ ] 11.2 Render `#screen-fake-call` with caller name/photo from settings (default "Mom") and wire accept/decline buttons
    - Render `#screen-in-call` with live call timer and end-call button
    - _Requirements: 3.1, 3.3, 3.5, 3.6_

  - [ ] 11.3 Add fake call configuration to `#screen-settings`: caller name, caller photo, and delay slider (0–60 s)
    - Persist settings to `safeher_settings` via `StorageService`
    - _Requirements: 3.3, 3.7_

  - [ ]* 11.4 Write property test for fake call delay validation (Property 6)
    - **Property 6: Fake call delay validation**
    - **Validates: Requirements 3.7**

  - [ ]* 11.5 Write unit test for fake call default caller name "Mom"
    - _Requirements: 3.3_

  - [ ]* 11.6 Write unit tests for accept/decline screen transitions
    - _Requirements: 3.5, 3.6_

- [ ] 12. Implement ShakeDetector module
  - [ ] 12.1 Write `ShakeDetector` with `enable()`, `disable()`, `isEnabled()`, and `onShake(callback)`
    - Listen to `DeviceMotionEvent`; compute `√(x²+y²+z²)`; trigger when > 24.5 m/s² for ≥500 ms
    - Call `EmergencyAlert.triggerAlert()` on shake; fire a system notification confirming the alert
    - Persist enabled state to `safeher_settings` via `StorageService`
    - Hide toggle and show tooltip if `DeviceMotionEvent` is unsupported; disable with message if iOS permission denied
    - _Requirements: 4.1, 4.2, 4.3, 4.4, 4.5_

  - [ ] 12.2 Wire the Shake-to-Alert toggle on `#screen-main` to `enable()` / `disable()`
    - Show active indicator when enabled
    - Show battery warning toast when battery level < 10%
    - _Requirements: 4.4, 4.5, 4.6_

  - [ ]* 12.3 Write property test for shake threshold triggers alert (Property 7)
    - **Property 7: Shake threshold triggers alert**
    - **Validates: Requirements 4.2**

  - [ ]* 12.4 Write property test for shake detector toggle round-trip (Property 8)
    - **Property 8: Shake detector toggle round-trip**
    - **Validates: Requirements 4.4, 4.5**

  - [ ]* 12.5 Write unit test for shake detector `enable()` registers `devicemotion` listener
    - _Requirements: 4.1_

  - [ ]* 12.6 Write unit test for battery below 10% warning
    - _Requirements: 4.6_

- [ ] 13. Implement permissions flow and permission-gated UI
  - [ ] 13.1 On first launch, request location permission with explanatory prompt; request contacts permission with explanatory prompt
    - _Requirements: 6.1, 6.2_

  - [ ] 13.2 When location permission is denied, visually disable Emergency Alert, Share Location, and Shake-to-Alert; show explanatory banner; prevent `GeoService` calls from those buttons
    - _Requirements: 6.3_

  - [ ] 13.3 When contacts permission is denied, show manual name/phone entry form instead of native contact picker
    - _Requirements: 6.4_

  - [ ]* 13.4 Write property test for location permission denial disables dependent features (Property 20)
    - **Property 20: Location permission denial disables dependent features**
    - **Validates: Requirements 6.3**

  - [ ]* 13.5 Write unit test for first-launch permission requests
    - _Requirements: 6.1, 6.2_

- [ ] 14. Checkpoint — Ensure all tests pass, ask the user if questions arise.

- [ ] 15. Validate main screen UI properties
  - [ ] 15.1 Verify all four primary buttons are present on `#screen-main` without scrolling; confirm each has an icon child and a non-empty label child
    - _Requirements: 7.1, 7.4_

  - [ ]* 15.2 Write property test for primary button touch targets ≥ 64 px (Property 21)
    - **Property 21: Primary button touch targets**
    - **Validates: Requirements 7.2**

  - [ ]* 15.3 Write property test for primary buttons have distinct colors (Property 22)
    - **Property 22: Primary buttons have distinct colors**
    - **Validates: Requirements 7.3**

  - [ ]* 15.4 Write property test for primary buttons contain icon and label (Property 23)
    - **Property 23: Primary buttons contain icon and label**
    - **Validates: Requirements 7.4**

  - [ ]* 15.5 Write unit test for main screen renders all 4 buttons with correct icons and labels
    - _Requirements: 7.1, 7.4_

  - [ ]* 15.6 Write unit test for app loads and displays main screen within 2 seconds
    - _Requirements: 7.5_

- [ ] 16. Wire everything together and handle error states
  - [ ] 16.1 Connect all modules: `ShakeDetector` → `EmergencyAlert`, `LocationSharing` → `NotificationService`, `FakeCall` → screen transitions
    - Ensure `StorageService` quota-exceeded toast is wired globally
    - Ensure all audio failures degrade silently without blocking UI
    - _Requirements: 1.3, 2.2, 4.2, 4.3_

  - [ ] 16.2 Load persisted `safeher_settings` and `safeher_contacts` on startup and initialize all module states accordingly
    - _Requirements: 5.5, 5.9_

- [ ] 17. Final checkpoint — Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional and can be skipped for a faster MVP
- Property tests use **fast-check** loaded from CDN; each must run ≥ 100 iterations and be tagged `// Feature: safe-her, Property N: <text>`
- All 23 correctness properties from the design document are covered by property test sub-tasks above
- Checkpoints at tasks 7, 14, and 17 ensure incremental validation across the build
