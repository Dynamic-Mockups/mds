# Cancellation Flow - Analytics Events

Events are captured via `useCaptureEvent` (PostHog) throughout the cancellation flow. All events are prefixed with `Cancellation Flow - ` for easy filtering in analytics dashboards, except for `User initiated subscription cancellation` which is a legacy event shared with other cancellation entry points.

## Event Reference

| Event | Payload | Description |
|-------|---------|-------------|
| `Cancellation Flow - Opened` | — | User clicked the "Downgrade to Free" button and the flow opened |
| `Cancellation Flow - Step Changed` | `from`, `to` | Fires on every step transition throughout the flow |
| `Cancellation Flow - Reason Selected` | `reason` | User selected a cancellation reason from the list |
| `Cancellation Flow - Kept Account` | `step`, `reason` | User clicked "Keep My Account" — fired before credits are awarded |
| `Cancellation Flow - Credits Awarded` | `credits` | Credits were successfully awarded to the user for keeping their account |
| `Cancellation Flow - Offer Accepted` | `offer_type`, `discount_percentage` | User accepted a discount or seasonal offer |
| `Cancellation Flow - Offer Declined` | `offerType` | User declined a presented offer and continued toward cancellation |
| `Cancellation Flow - Credits Claimed` | — | User claimed available credits from the offer screen |
| `Cancellation Flow - Feature Requested` | `feature` | User submitted a missing feature request |
| `Cancellation Flow - Feedback Submitted` | `feedback` | User submitted free-text feedback (Other reason path) |
| `Cancellation Flow - Support Booked` | — | User clicked to book a support call (Technical Issues path) |
| `Cancellation Flow - Try AI Clicked` | — | User clicked to try the AI mockup feature (Missing Mockups path) |
| `Cancellation Flow - Roadmap Viewed` | — | User clicked to view the product roadmap |
| `Cancellation Flow - Back to Dashboard` | — | User clicked "Back to Dashboard" from the completion screen |
| `Cancellation Flow - Subscription Cancelled` | `reason` | Cancellation API call succeeded |
| `Cancellation Flow - Abandoned` | `step`, `reason` | User dismissed the flow via X button or backdrop click without completing |
| `User initiated subscription cancellation` | `cycleEndDate`, `billingStatus`, `amount`, `amount_paid`, `period`, `reasons`, `additionalDetails`, `detailedReasons` | Legacy event fired alongside `Subscription Cancelled` for backwards compatibility with existing analytics |

## User Paths

| Path | Events Fired (in order) |
|------|------------------------|
| Opens → "Keep My Account" immediately | `Opened` → `Kept Account` → `Credits Awarded` |
| Opens → reason → accepts discount | `Opened` → `Reason Selected` → `Step Changed` → `Offer Accepted` |
| Opens → reason → declines discount → cancels | `Opened` → `Reason Selected` → `Step Changed` → `Offer Declined` → `Step Changed` → `Subscription Cancelled` + `User initiated subscription cancellation` |
| Opens → Missing Mockups → Try AI | `Opened` → `Reason Selected` → `Step Changed` → `Try AI Clicked` |
| Opens → Missing Features → submits request → cancels | `Opened` → `Reason Selected` → `Step Changed` → `Feature Requested` → `Subscription Cancelled` + `User initiated subscription cancellation` |
| Opens → Seasonal Business → accepts seasonal offer | `Opened` → `Reason Selected` → `Step Changed` → `Offer Accepted` |
| Opens → Technical Issues → books support | `Opened` → `Reason Selected` → `Step Changed` → `Support Booked` |
| Opens → Other → submits feedback → cancels | `Opened` → `Reason Selected` → `Step Changed` → `Feedback Submitted` → `Subscription Cancelled` + `User initiated subscription cancellation` |
| Opens → views roadmap | `Opened` → `Reason Selected` → `Step Changed` → `Roadmap Viewed` |
| Opens → claims credits | `Opened` → `Reason Selected` → `Step Changed` → `Credits Claimed` |
| Opens → back to dashboard | `Opened` → `Back to Dashboard` |
| Any step → X / backdrop | `Abandoned` (with `step` + `reason` if already selected) |

> **Note:** `Step Changed` fires on every transition between steps throughout any path. It is shown once per path above for brevity but may fire multiple times.

## Cancellation Reason Values

| Value | Displayed as |
|-------|-------------|
| `too-expensive` | Too Expensive |
| `missing-mockups` | Missing Mockups |
| `missing-features` | Missing Features |
| `seasonal-business` | Seasonal Business |
| `technical-issues` | Technical Issues |
| `other` | Other |

## Source

All events are fired from:
`src/features/screens/workspace/components/workspace-settings/components/view-wrapper/DowngradeToFreeButton.tsx`
