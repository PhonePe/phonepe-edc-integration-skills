# PhonePe Offline (EDC) Skills

This repository contains **Agent Skills** for integrating Android applications with **PhonePe EDC (Electronic Data Capture)** terminals for offline card and QR payments. These skills enable GitHub Copilot CLI, Copilot coding agent, and VS Code to assist you in implementing PhonePe EDC integration with the correct Intent contracts, workflows, callbacks, and error handling.

---

## 📋 Table of Contents

- [About These Skills](#-about-these-skills)
- [Prerequisites](#-prerequisites)
- [Setup Instructions](#-setup-instructions)
- [Quick Reference](#-quick-reference)
- [Supported Workflows](#-supported-workflows)
- [Quick Start](#-quick-start)
- [Usage Examples](#-usage-examples)
- [Environment Configuration](#️-environment-configuration)
- [Troubleshooting](#-troubleshooting)
- [Additional Resources](#-additional-resources)

---

## 🎯 About These Skills

This skill collection provides AI-powered assistance for:

- ✅ **Terminal Configuration** – One-time `CONFIGURE` workflow required on first app launch
- ✅ **Card Payments** – EMV chip and magstripe payments on the EDC terminal
- ✅ **Dynamic QR (DQR) Payments** – On-screen QR generation and collection
- ✅ **Transaction Status** – Query transaction state with optional duplicate receipt
- ✅ **Receipt Printing** – Custom receipt templates using built-in print elements
- ✅ **Error Handling** – Complete error-code reference including pending reversal recovery

> **Note:** PhonePe EDC integration works over **Android Intents + `ResultReceiver` (IPC)** — not REST APIs.

---

## 📦 Prerequisites

Before using these skills, ensure you have:

1. **PhonePe EDC Terminal & Merchant Onboarding**
   - A PhonePe EDC device provisioned for your merchant account
   - PhonePe EDC app installed (staging or production build)

2. **GitHub Copilot CLI** (or one of the supported tools)
   - Install: `brew install copilot-cli` or [other installation methods](https://docs.github.com/en/copilot/how-tos/set-up/install-copilot-cli)
   - Active Copilot subscription

3. **Development Environment**
   - Android Studio with Kotlin/Java support
   - Bash shell (for the setup script)
   - Git

---

## 🚀 Setup Instructions

### Quick Setup (One Command)

Install directly with a single command from your project root:

```bash
# Using curl
curl -fsSL https://raw.githubusercontent.com/PhonePe/phonepe-offline-skills/main/setup.sh | bash

# Or using wget
wget -qO- https://raw.githubusercontent.com/PhonePe/phonepe-offline-skills/main/setup.sh | bash
```

The script will:
- Check prerequisites (Git, GitHub Copilot CLI)
- Clone the repository automatically
- Guide you through setup for new or existing projects
- Install the skills into `.github/skills/`
- Update `.gitignore` for security

### Alternative: Clone and Run

```bash
git clone https://github.com/PhonePe/phonepe-offline-skills.git
cd phonepe-offline-skills
./setup.sh
```

### Manual Setup

<details>
<summary>Click to expand manual setup instructions</summary>

```bash
# From the cloned repository, copy skills into your Android project
mkdir -p /path/to/your/project/.github/skills
cp -r phonepe-offline-skill /path/to/your/project/.github/skills/

# Navigate to your project and start Copilot CLI
cd /path/to/your/project
copilot
```

Verify the skill is detected:

```bash
/skills list
```

</details>

---

## 📌 Quick Reference

| Item | Value |
|------|-------|
| **Intent Action** | `com.phonepe.edc.ACTION_WORKFLOW` |
| **Permission** | `com.phonepe.edc.app.INVOKE_WORKFLOW` |
| **Package (Staging)** | `com.phonepe.edc.app.stage` |
| **Package (Production)** | `com.phonepe.edc.app` |
| **Amount Unit** | Paise (₹1 = 100) |

---

## 🔁 Supported Workflows

| Workflow | Purpose | Key Parameters |
|----------|---------|----------------|
| `CONFIGURE` | Terminal setup (required on first launch) | `TERMINAL_MODE="EXTERNAL"` |
| `ONLINE_SALE` | Card & Dynamic QR payments | `AMOUNT`, `ALLOWED_PAYMENT_INSTRUMENTS` |
| `CHECK_STATUS` | Transaction status lookup | `TXN_ID`, `AMOUNT`, `PRINT_SALE_RECEIPT` |
| `PRINT_RECEIPT` | Custom receipt printing | `PRINT_DATA` (JSON template) |

---

## ⚡ Quick Start

### 1. Add to `AndroidManifest.xml`

```xml
<uses-permission android:name="com.phonepe.edc.app.INVOKE_WORKFLOW"/>

<queries>
    <intent>
        <action android:name="com.phonepe.edc.ACTION_WORKFLOW" />
    </intent>
</queries>
```

### 2. Minimum Code Example

```kotlin
val intent = Intent("com.phonepe.edc.ACTION_WORKFLOW").apply {
    putExtra("TXN_ID", UUID.randomUUID().toString())
    putExtra("WORKFLOW", "ONLINE_SALE")
    putExtra("AMOUNT", 50000L)  // ₹500 in paise
    setPackage("com.phonepe.edc.app")
}
context.startForegroundService(intent)
```

### 3. Integration Steps

1. **Configure terminal** – Call the `CONFIGURE` workflow on first app launch
2. **Process payments** – Use `ONLINE_SALE` with the amount in paise (₹1 = 100)
3. **Handle response** – Implement a `ResultReceiver` for async callbacks

---

## 💡 Usage Examples

Start Copilot CLI in your project directory:

```bash
copilot
```

### Example 1: Accept a Card Payment

```
Integrate PhonePe EDC card payment in my Android app for ₹500 with a ResultReceiver callback
```

Copilot will:
1. Add the required manifest permission and `<queries>` block
2. Generate a reusable `PhonePePaymentManager` class
3. Wire up the `ResultReceiver` callback handling

### Example 2: Configure the Terminal

```
Add the PhonePe EDC CONFIGURE workflow to my app's first-launch flow
```

### Example 3: Print a Custom Receipt

```
Generate a PhonePe EDC PRINT_RECEIPT JSON template with merchant name, amount, and a separator
```

### Example 4: Debug an Integration Issue

```
I'm getting ERR_PENDING_REVERSAL_FAILED on my PhonePe EDC terminal. Help me fix it.
```

---

## ⚙️ Environment Configuration

**Staging (Testing):**
```kotlin
const val PHONEPE_PACKAGE = "com.phonepe.edc.app.stage"
```

**Production:**
```kotlin
const val PHONEPE_PACKAGE = "com.phonepe.edc.app"
```

Switch via build variants so the correct package is targeted per environment:

```kotlin
buildTypes {
    debug   { buildConfigField("String", "PHONEPE_PACKAGE", "\"com.phonepe.edc.app.stage\"") }
    release { buildConfigField("String", "PHONEPE_PACKAGE", "\"com.phonepe.edc.app\"") }
}
```

---

## 🔧 Troubleshooting

### Skills Not Loading

**Problem:** `/skills list` returns empty

**Solutions:**
1. Check file structure: `.github/skills/phonepe-offline-skill/SKILL.md` must exist
2. Verify `SKILL.md` frontmatter has a lowercase `name` field
3. Run `/skills reload`
4. Restart Copilot CLI

### Common Integration Errors

| Error | Solution |
|-------|----------|
| `ERR_PENDING_REVERSAL_FAILED` | PhonePe App → Settings → Clear Reversals |
| `ERR_TXN_DECLINED` | Try another card |
| `ERR_APP_NOT_INITIALIZED` | Run the `CONFIGURE` workflow first |
| `APP_NOT_FOUND` | Install the PhonePe EDC app on the terminal |
| `PERMISSION_ERROR` | Add `INVOKE_WORKFLOW` permission to `AndroidManifest.xml` |

### Intent Not Resolving on Android 11+

Ensure the `<queries>` block is present in `AndroidManifest.xml` — package visibility restrictions will otherwise block the Intent.

---

## 📚 Additional Resources

| Document | Description |
|----------|-------------|
| [EDC.md](phonepe-offline-skill/Offline%20Integrations/EDC.md) | Complete integration guide with copy-paste code |
| [SKILL.md](phonepe-offline-skill/SKILL.md) | Quick reference for all workflows |
| [CHANGELOG.md](CHANGELOG.md) | Version history and migration notes |

- [PhonePe Official Documentation](https://developer.phonepe.com/)
- [GitHub Copilot CLI Documentation](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/use-copilot-cli)
- [Agent Skills Standard](https://github.com/agentskills/agentskills)

---

## Contributing

Contributions to PhonePe Offline Skills are welcome! Here's how you can contribute:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

Please ensure your changes follow the project's documentation standards.

---

## License

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.

```
Copyright 2026 PhonePe Private Limited

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```

---

**Version:** 1.0.0 | **Copyright:** PhonePe Private Limited
