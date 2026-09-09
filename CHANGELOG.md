# Changelog

All notable changes to the PhonePe EDC SDK Skills are documented in this file.

## [1.0.0-20260216] - 11 May 2026

### Added
- **Initial Release** - PhonePe EDC SDK Skills for Android POS terminals

#### Skills Added

- **phonepe-pg-skill/SKILL.md**: Master skill index file
  - Frontmatter with skill metadata (name, description, version)
  - "When to Apply" trigger scenarios for AI matching
  - Pre-requisites and AndroidManifest.xml configuration
  - Quick reference table for all workflows
  - Links to complete EDC integration guide

- **phonepe-pg-skill/Offline Integrations/EDC.md**: Complete EDC Integration Guide
  - `CONFIGURE_TERMINAL` — One-time mandatory terminal setup with AUTO_PRINT, AUTO_FINISH, TERMINAL_MODE options
  - `PROCESS_SALE` — Card payment (EMV chip/magstripe) and Dynamic QR (DQR) processing
  - `PRINT_CUSTOM_RECEIPT` — 7 print element types (TEXT, TEXT_KEY_VALUE, MULTI_COLUMN_TEXT, SEPARATOR, TABLE, BITMAP, FEED_LINE)
  - `CHECK_TRANSACTION_STATUS` — Last transaction status query with duplicate receipt option
  - Complete error code reference (40+ codes)
  - Manual reversal clearing instructions for `ERR_PENDING_REVERSAL_FAILED`
  - ResultReceiver serialization patterns
  - FileProvider and ClipData patterns for bitmap sharing
  - Full integration example (Retail POS scenario)

#### Setup & Documentation

- **setup.sh**: Automated installation script
  - Supports standalone execution via `curl | bash` or `wget | bash`
  - Supports running from cloned repository
  - Interactive setup for new or existing projects
  - Creates `.env.template` with EDC configuration
  - Handles Git and Copilot CLI prerequisite checks
  - 60-second timeout on git clone operations
  - Input validation for all user prompts

- **README.md**: Project documentation
  - Prerequisites for Android EDC development
  - Quick setup and manual installation instructions
  - Available skills overview with field tables
  - Usage examples for all 4 workflows
  - Troubleshooting guide with common error codes

### Technical Details

- **Communication Pattern**: Android Intents with ResultReceiver (IPC)
- **Intent Action**: `com.phonepe.edc.ACTION_WORKFLOW`
- **Required Permission**: `com.phonepe.edc.app.INVOKE_WORKFLOW`
- **Supported Workflows**: CONFIGURE, SALE, ONLINE_SALE, PRINT_RECEIPT, CHECK_STATUS
- **Payment Instruments**: CARD (EMV chip/magstripe), DQR (Dynamic QR)

---

## Migration Note

This repository was restructured from online Payment Gateway skills to offline EDC SDK skills. The EDC integration uses:

| Aspect | Previous (PG) | Current (EDC) |
|--------|---------------|---------------|
| Platform | Web/Server | Android POS Terminal |
| Communication | REST APIs + OAuth | Android Intents + ResultReceiver |
| Authentication | OAuth 2.0 tokens | Android permission model |
| Callbacks | Webhooks (HTTP) | ResultReceiver (IPC) |
| Use Case | Online payments | Offline card/DQR payments |

---

**Version:** 1.0.0-20260216  
**Created By:** Saurabh Jaiswal & Kusumakar Dwivedi  
**Approved By:** Suhail Mehta  
**License:** Apache License 2.0
