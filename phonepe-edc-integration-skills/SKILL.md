---
name: phonepe-edc-sdk-skill
description: PhonePe EDC Android POS integration via Intents for offline card and QR payments
version: 1.0.0
---

# PhonePe EDC SDK Skill

## When to Apply

Use this skill when the user asks about:
- PhonePe EDC integration
- Android POS terminal payments
- Card payment on EDC terminal
- QR payment on EDC
- ResultReceiver callback
- `com.phonepe.edc.ACTION_WORKFLOW`
- Configure terminal / CONFIGURE workflow
- ONLINE_SALE workflow
- CHECK_STATUS workflow
- PRINT_RECEIPT workflow
- `ERR_PENDING_REVERSAL_FAILED` error
- EDC receipt printing

---

## Quick Reference

| Item | Value |
|------|-------|
| **Intent Action** | `com.phonepe.edc.ACTION_WORKFLOW` |
| **Permission** | `com.phonepe.edc.app.INVOKE_WORKFLOW` |
| **Package (Staging)** | `com.phonepe.edc.app.stage` |
| **Package (Production)** | `com.phonepe.edc.app` |

---

## Required AndroidManifest.xml

```xml
<uses-permission android:name="com.phonepe.edc.app.INVOKE_WORKFLOW"/>

<queries>
    <intent>
        <action android:name="com.phonepe.edc.ACTION_WORKFLOW" />
    </intent>
</queries>
```

---

## Available Workflows

| Workflow | Purpose | Key Parameters |
|----------|---------|----------------|
| `CONFIGURE` | Terminal setup (first launch) | `TERMINAL_MODE="EXTERNAL"` |
| `ONLINE_SALE` | Card/QR payment | `AMOUNT`, `ALLOWED_PAYMENT_INSTRUMENTS` |
| `CHECK_STATUS` | Transaction status | `TXN_ID`, `AMOUNT`, `PRINT_SALE_RECEIPT` |
| `PRINT_RECEIPT` | Custom receipt | `PRINT_DATA` (JSON template) |

---

## Complete Integration Guide

**See: [Offline Integrations/EDC.md](Offline%20Integrations/EDC.md)** for:
- Complete `PhonePePaymentManager` class (copy-paste ready)
- Activity usage example with callback handling
- All workflow parameters with descriptions
- Print receipt JSON template
- Error codes and solutions
- Environment configuration (staging vs production)

---

## Minimum Working Example

```kotlin
// 1. Initialize manager
val manager = PhonePePaymentManager(context) { success, type, response ->
    if (success) Log.d("EDC", "Success: $response")
    else Log.e("EDC", "Failed: $response")
}

// 2. Configure terminal (first launch only)
manager.startConfigure()

// 3. Accept payment (₹500)
manager.startOnlineSaleCard(50000)  // Card only
manager.startOnlineSaleDQR(50000)   // QR only
manager.startOnlineSale(50000)      // Card + QR
```

---

## Common Errors

| Error | Solution |
|-------|----------|
| `ERR_PENDING_REVERSAL_FAILED` | PhonePe App → Settings → Clear Reversals |
| `ERR_TXN_DECLINED` | Try another card |
| `APP_NOT_FOUND` | Install PhonePe EDC app |
| `PERMISSION_ERROR` | Add permission to AndroidManifest.xml |