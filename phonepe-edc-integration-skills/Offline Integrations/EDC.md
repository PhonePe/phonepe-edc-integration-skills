---
name: phonepe-edc-integration
description: PhonePe EDC Android POS integration via Intents for offline payments
version: 1.0.0
---

# PhonePe EDC SDK Integration Guide

## Quick Reference

| Item | Value |
|------|-------|
| **Intent Action** | `com.phonepe.edc.ACTION_WORKFLOW` |
| **Permission** | `com.phonepe.edc.app.INVOKE_WORKFLOW` |
| **Package (Staging)** | `com.phonepe.edc.app.stage` |
| **Package (Production)** | `com.phonepe.edc.app` |
| **Communication** | Android `ResultReceiver` (async IPC) |
| **Workflows** | `CONFIGURE`, `ONLINE_SALE`, `CHECK_STATUS`, `PRINT_RECEIPT` |

---

## Quick Start (Minimum Code)

```kotlin
// 1. Create intent with workflow
val intent = Intent("com.phonepe.edc.ACTION_WORKFLOW").apply {
    putExtra("TXN_ID", UUID.randomUUID().toString())
    putExtra("WORKFLOW", "ONLINE_SALE")
    putExtra("AMOUNT", 50000L)  // ₹500 in paise
    setPackage("com.phonepe.edc.app")  // Production package
}

// 2. Start the service
context.startForegroundService(intent)
```

---

## AndroidManifest.xml (Required)

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <!-- Permission to invoke PhonePe EDC workflows -->
    <uses-permission android:name="com.phonepe.edc.app.INVOKE_WORKFLOW"/>

    <!-- Android 11+ intent visibility (required for resolveService) -->
    <queries>
        <intent>
            <action android:name="com.phonepe.edc.ACTION_WORKFLOW" />
        </intent>
    </queries>

    <application>
        <!-- Your activities -->
    </application>
</manifest>
```

---

## Complete Integration Class

Copy this complete working class to integrate PhonePe EDC:

```kotlin
package com.yourpackage

import android.content.Context
import android.content.Intent
import android.os.Bundle
import android.os.Handler
import android.os.Looper
import android.os.Parcel
import android.os.ResultReceiver
import android.util.Log
import java.util.UUID

/**
 * PhonePe EDC Payment Manager
 * Handles all EDC workflows: CONFIGURE, ONLINE_SALE, CHECK_STATUS, PRINT_RECEIPT
 */
class PhonePePaymentManager(
    private val context: Context,
    private val onResult: (success: Boolean, type: String?, response: String?) -> Unit
) {

    companion object {
        private const val TAG = "PhonePeEDC"
        private const val ACTION_WORKFLOW = "com.phonepe.edc.ACTION_WORKFLOW"
        
        // Change package based on environment
        private const val PACKAGE_STAGING = "com.phonepe.edc.app.stage"
        private const val PACKAGE_PRODUCTION = "com.phonepe.edc.app"
    }
    
    // Set to false for production
    private val isStaging = true
    private val packageName = if (isStaging) PACKAGE_STAGING else PACKAGE_PRODUCTION

    // ═══════════════════════════════════════════════════════════════════
    // ResultReceiver: Receives async response from PhonePe EDC App
    // ═══════════════════════════════════════════════════════════════════
    private val resultReceiver = object : ResultReceiver(Handler(Looper.getMainLooper())) {
        override fun onReceiveResult(resultCode: Int, resultData: Bundle?) {
            super.onReceiveResult(resultCode, resultData)

            // RESULT_OK is -1 in Android
            if (resultCode == -1) {
                resultData?.let {
                    val type = it.getString("type")         // CONFIGURE, ONLINE_SALE, CHECK_STATUS, PRINT_RECEIPT
                    val success = it.getBoolean("success", false)
                    val response = it.getString("response") // JSON string with details
                    onResult(success, type, response)
                }
            } else {
                onResult(false, "ERROR", "User cancelled or terminal error")
            }
        }
    }

    // ═══════════════════════════════════════════════════════════════════
    // MANDATORY: Serialize ResultReceiver for Inter-Process Communication
    // ═══════════════════════════════════════════════════════════════════
    private fun ResultReceiver.toParcelable(): ResultReceiver {
        val parcel = Parcel.obtain()
        this.writeToParcel(parcel, 0)
        parcel.setDataPosition(0)
        val ipcReceiver = ResultReceiver.CREATOR.createFromParcel(parcel)
        parcel.recycle()
        return ipcReceiver
    }

    // ═══════════════════════════════════════════════════════════════════
    // WORKFLOW 1: CONFIGURE - Terminal Setup (REQUIRED on first launch)
    // ═══════════════════════════════════════════════════════════════════
    fun startConfigure() {
        val bundle = Bundle().apply {
            putString("TXN_ID", UUID.randomUUID().toString())
            putString("WORKFLOW", "CONFIGURE")
            putString("TERMINAL_MODE", "EXTERNAL")              // MANDATORY on first run
            putLong("AUTO_FINISH_FAILURE_DELAY", 300000)        // 5 minutes timeout
            putParcelable("EXTERNAL_RESULT_RECEIVER", resultReceiver.toParcelable())
        }
        sendIntent(bundle)
    }

    // ═══════════════════════════════════════════════════════════════════
    // WORKFLOW 2: ONLINE_SALE - Card Payment
    // ═══════════════════════════════════════════════════════════════════
    fun startOnlineSaleCard(amountInPaise: Long) {
        val bundle = Bundle().apply {
            putString("TXN_ID", UUID.randomUUID().toString())
            putString("WORKFLOW", "ONLINE_SALE")
            putLong("AMOUNT", amountInPaise)                    // Amount in paise (₹1 = 100)
            putStringArrayList("ALLOWED_PAYMENT_INSTRUMENTS", arrayListOf("CARD"))
            putParcelable("EXTERNAL_RESULT_RECEIVER", resultReceiver.toParcelable())
        }
        sendIntent(bundle)
    }

    // ═══════════════════════════════════════════════════════════════════
    // WORKFLOW 2: ONLINE_SALE - Dynamic QR Payment
    // ═══════════════════════════════════════════════════════════════════
    fun startOnlineSaleDQR(amountInPaise: Long) {
        val bundle = Bundle().apply {
            putString("TXN_ID", UUID.randomUUID().toString())
            putString("WORKFLOW", "ONLINE_SALE")
            putLong("AMOUNT", amountInPaise)                    // Amount in paise (₹1 = 100)
            putStringArrayList("ALLOWED_PAYMENT_INSTRUMENTS", arrayListOf("DQR"))
            putParcelable("EXTERNAL_RESULT_RECEIVER", resultReceiver.toParcelable())
        }
        sendIntent(bundle)
    }

    // ═══════════════════════════════════════════════════════════════════
    // WORKFLOW 2: ONLINE_SALE - Both Card and DQR
    // ═══════════════════════════════════════════════════════════════════
    fun startOnlineSale(amountInPaise: Long) {
        val bundle = Bundle().apply {
            putString("TXN_ID", UUID.randomUUID().toString())
            putString("WORKFLOW", "ONLINE_SALE")
            putLong("AMOUNT", amountInPaise)
            putStringArrayList("ALLOWED_PAYMENT_INSTRUMENTS", arrayListOf("CARD", "DQR"))
            putParcelable("EXTERNAL_RESULT_RECEIVER", resultReceiver.toParcelable())
        }
        sendIntent(bundle)
    }

    // ═══════════════════════════════════════════════════════════════════
    // WORKFLOW 3: CHECK_STATUS - Check last transaction & print receipt
    // ═══════════════════════════════════════════════════════════════════
    fun checkStatus(
        txnId: String,
        amountInPaise: Long,
        printReceipt: Boolean = true,
        receiptType: String = "MERCHANT"  // "MERCHANT" or "CUSTOMER"
    ) {
        val bundle = Bundle().apply {
            putString("TXN_ID", txnId)
            putString("WORKFLOW", "CHECK_STATUS")
            putLong("AMOUNT", amountInPaise)
            putBoolean("PRINT_SALE_RECEIPT", printReceipt)
            putString("SALE_RECEIPT_TYPE", receiptType)
            putParcelable("EXTERNAL_RESULT_RECEIVER", resultReceiver.toParcelable())
        }
        sendIntent(bundle)
    }

    // ═══════════════════════════════════════════════════════════════════
    // WORKFLOW 4: PRINT_RECEIPT - Custom Receipt Printing
    // ═══════════════════════════════════════════════════════════════════
    fun printReceipt(printData: String) {
        val bundle = Bundle().apply {
            putString("TXN_ID", UUID.randomUUID().toString())
            putString("WORKFLOW", "PRINT_RECEIPT")
            putString("PRINT_DATA", printData)                  // JSON template string
            putParcelable("EXTERNAL_RESULT_RECEIVER", resultReceiver.toParcelable())
        }
        sendIntent(bundle)
    }

    // ═══════════════════════════════════════════════════════════════════
    // Common Intent Sender with Error Handling
    // ═══════════════════════════════════════════════════════════════════
    private fun sendIntent(bundle: Bundle) {
        val intent = Intent(ACTION_WORKFLOW).apply {
            putExtras(bundle)
            setPackage(packageName)  // IMPORTANT: Must set explicit package
        }

        try {
            // Verify service exists before calling
            if (context.packageManager.resolveService(intent, 0) == null) {
                val msg = "PhonePe EDC App not found. Please install the app."
                Log.e(TAG, msg)
                onResult(false, "APP_NOT_FOUND", msg)
                return
            }
            context.startForegroundService(intent)
        } catch (e: SecurityException) {
            Log.e(TAG, "Permission denied: ${e.message}", e)
            onResult(false, "PERMISSION_ERROR", "Missing INVOKE_WORKFLOW permission")
        } catch (e: Exception) {
            Log.e(TAG, "Error: ${e.message}", e)
            onResult(false, "INTENT_ERROR", e.message)
        }
    }
}
```

---

## Usage in Activity

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var phonePeManager: PhonePePaymentManager

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Initialize manager with callback
        phonePeManager = PhonePePaymentManager(this) { success, type, response ->
            runOnUiThread {
                handleResult(success, type, response)
            }
        }

        // Button: Configure Terminal (call ONCE on first launch)
        btnConfigure.setOnClickListener {
            phonePeManager.startConfigure()
        }

        // Button: Pay ₹500 via Card
        btnPayCard.setOnClickListener {
            phonePeManager.startOnlineSaleCard(50000)  // 50000 paise = ₹500
        }

        // Button: Pay ₹500 via QR
        btnPayQR.setOnClickListener {
            phonePeManager.startOnlineSaleDQR(50000)
        }

        // Button: Pay ₹500 (Card or QR)
        btnPay.setOnClickListener {
            phonePeManager.startOnlineSale(50000)
        }
    }

    private fun handleResult(success: Boolean, type: String?, response: String?) {
        when {
            success && type == "CONFIGURE" -> {
                Toast.makeText(this, "Terminal configured!", Toast.LENGTH_SHORT).show()
            }
            success && type == "ONLINE_SALE" -> {
                // Parse response JSON for transaction details
                Toast.makeText(this, "Payment successful!", Toast.LENGTH_SHORT).show()
                Log.d("EDC", "Response: $response")
            }
            !success -> {
                handleError(type, response)
            }
        }
    }

    private fun handleError(type: String?, response: String?) {
        val message = when {
            response?.contains("ERR_PENDING_REVERSAL_FAILED") == true -> 
                "Please clear pending reversals:\nPhonePe App → Settings → Clear Reversals"
            response?.contains("ERR_TXN_DECLINED") == true -> 
                "Card declined. Try another card."
            response?.contains("ERR_PAYMENT_CANCELLED") == true -> 
                "Payment cancelled by user"
            response?.contains("APP_NOT_FOUND") == true -> 
                "PhonePe EDC App not installed"
            response?.contains("PERMISSION_ERROR") == true -> 
                "Permission denied. Check AndroidManifest.xml"
            else -> "Error: $response"
        }
        Toast.makeText(this, message, Toast.LENGTH_LONG).show()
    }
}
```

---

## Workflow Parameters Reference

### CONFIGURE Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `TXN_ID` | String | Yes | Unique transaction ID |
| `WORKFLOW` | String | Yes | `"CONFIGURE"` |
| `TERMINAL_MODE` | String | First run | `"EXTERNAL"` (mandatory on first configure) |
| `AUTO_FINISH_FAILURE_DELAY` | Long | No | Auto-close delay in ms (default: 300000) |
| `EXTERNAL_RESULT_RECEIVER` | ResultReceiver | Yes | Callback receiver |

### ONLINE_SALE Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `TXN_ID` | String | Yes | Unique transaction ID |
| `WORKFLOW` | String | Yes | `"ONLINE_SALE"` |
| `AMOUNT` | Long | Yes | Amount in paise (₹1 = 100 paise) |
| `ALLOWED_PAYMENT_INSTRUMENTS` | ArrayList<String> | No | `["CARD"]`, `["DQR"]`, or `["CARD", "DQR"]` |
| `EXTERNAL_RESULT_RECEIVER` | ResultReceiver | Yes | Callback receiver |

### CHECK_STATUS Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `TXN_ID` | String | Yes | Original transaction ID |
| `WORKFLOW` | String | Yes | `"CHECK_STATUS"` |
| `AMOUNT` | Long | Yes | Original amount in paise |
| `PRINT_SALE_RECEIPT` | Boolean | No | Print duplicate receipt |
| `SALE_RECEIPT_TYPE` | String | No | `"MERCHANT"` or `"CUSTOMER"` |
| `EXTERNAL_RESULT_RECEIVER` | ResultReceiver | Yes | Callback receiver |

### PRINT_RECEIPT Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `TXN_ID` | String | Yes | Unique transaction ID |
| `WORKFLOW` | String | Yes | `"PRINT_RECEIPT"` |
| `PRINT_DATA` | String | Yes | JSON template string |
| `EXTERNAL_RESULT_RECEIVER` | ResultReceiver | Yes | Callback receiver |

---

## Print Receipt Template

```kotlin
fun createReceiptTemplate(merchantName: String, amount: String): String {
    return """
    {
        "version": 1,
        "grayscale": 2000,
        "defaultFontSize": 16,
        "printElements": [
            {"type": "TEXT", "content": "$merchantName", "isBold": true, "fontSize": 24, "align": "CENTER"},
            {"type": "SEPARATOR"},
            {"type": "LEFT_RIGHT_TEXT", "leftContent": "Amount:", "rightContent": "$amount"},
            {"type": "SEPARATOR"},
            {"type": "TEXT", "content": "Thank You!", "align": "CENTER"},
            {"type": "NEW_LINE", "numLines": 3}
        ]
    }
    """.trimIndent()
}

// Usage
val template = createReceiptTemplate("MY STORE", "₹500.00")
phonePeManager.printReceipt(template)
```

### Print Element Types

| Type | Properties | Description |
|------|------------|-------------|
| `TEXT` | content, isBold, fontSize, align | Single line text |
| `LEFT_RIGHT_TEXT` | leftContent, rightContent | Two-column text |
| `SEPARATOR` | - | Dashed line separator |
| `NEW_LINE` | numLines | Blank lines |
| `BITMAP` | key, align | Image (requires PRINT_URI_MAP) |
| `TABLE` | columns, itemsData | Tabular data |

---

## Error Codes

| Error Code | Meaning | Solution |
|------------|---------|----------|
| `ERR_PENDING_REVERSAL_FAILED` | Stuck reversal | PhonePe App → Settings → Clear Reversals |
| `ERR_TXN_DECLINED` | Bank declined card | Try another card |
| `ERR_PAYMENT_CANCELLED_BY_USER` | User cancelled | Retry payment |
| `ERR_WORKFLOW_ALREADY_RUNNING` | Another workflow active | Wait for completion |
| `ERR_NO_INTERNET` | No network | Check connectivity |
| `ERR_APP_NOT_INITIALIZED` | Terminal not configured | Run CONFIGURE first |
| `APP_NOT_FOUND` | PhonePe app missing | Install PhonePe EDC app |
| `PERMISSION_ERROR` | Missing permission | Add permission to AndroidManifest.xml |

---

## Environment Configuration

```kotlin
// In PhonePePaymentManager class
companion object {
    // Staging environment (for testing)
    private const val PACKAGE_STAGING = "com.phonepe.edc.app.stage"
    
    // Production environment (for live)
    private const val PACKAGE_PRODUCTION = "com.phonepe.edc.app"
}

// Set based on your build variant
private val isStaging = BuildConfig.DEBUG
private val packageName = if (isStaging) PACKAGE_STAGING else PACKAGE_PRODUCTION
```

---

## Integration Checklist

- [ ] Add `INVOKE_WORKFLOW` permission to AndroidManifest.xml
- [ ] Add `<queries>` block for Android 11+ compatibility
- [ ] Create `PhonePePaymentManager` class
- [ ] Initialize manager with result callback
- [ ] Call `startConfigure()` on first app launch
- [ ] Set correct package name (staging vs production)
- [ ] Handle all error cases in callback
- [ ] Test on actual EDC terminal (not emulator)

---

## Response JSON Examples

### Successful ONLINE_SALE Response
```json
{
  "type": "ONLINE_SALE",
  "success": true,
  "response": {
    "amount": "500.00",
    "rrn": "412345678901",
    "invoiceNo": "000123",
    "cardLastFourDigits": "1234",
    "cardType": "VISA",
    "tranId": "uuid-string"
  }
}
```

### Failed Response
```json
{
  "type": "ONLINE_SALE",
  "success": false,
  "response": {
    "code": "ERR_TXN_DECLINED",
    "message": "Transaction declined by bank",
    "tranId": "uuid-string"
  }
}
```
