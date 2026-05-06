# SharePoint Chain-Copy Workflow Guide

## 🎯 Overview

This guide explains how to set up Power Automate flows that create shift files by **copying and renaming** the previous shift's file in a chain sequence.

## 📊 Workflow Logic

Each shift file is created by copying the previous shift's file:

```
Day 1 (May 1st):
└── Manual: Create US_CAN-Shift-01-May-2026.xlsx

Day 2 (May 2nd):
├── 6:30 AM  → Copy US_CAN-Shift-01-May-2026.xlsx → India_Shift_1_02-May-2026.xlsx
├── 2:30 PM  → Copy India_Shift_1_02-May-2026.xlsx → India_Shift_2_02-May-2026.xlsx
└── 10:30 PM → Copy India_Shift_2_02-May-2026.xlsx → US_CAN-Shift-02-May-2026.xlsx

Day 3 (May 3rd):
├── 6:30 AM  → Copy US_CAN-Shift-02-May-2026.xlsx → India_Shift_1_03-May-2026.xlsx
└── (continues...)
```

## 🏗️ System Architecture

### Five Power Automate Flows

1. **Monthly Folder Creator** - Runs 1st of month at 6:00 AM IST
2. **Daily Folder Creator** - Runs daily at 6:15 AM IST
3. **India Shift 1 Generator** - Runs daily at 6:30 AM IST (copies yesterday's US-CAN file)
4. **India Shift 2 Generator** - Runs daily at 2:30 PM IST (copies today's Shift 1)
5. **US-Canada Shift Generator** - Runs daily at 10:30 PM IST (copies today's Shift 2)

## 📋 Prerequisites

### SharePoint Setup

**Site Details:**
- Site Address: `https://ibm-my.sharepoint.com/personal/b_vignesh19_ibm_com`
- Library: `Documents`
- Base Path: `/OC HANDOVER CIBC INTRIA/Daily Handover`

**Initial Folder Structure:**
```
/OC HANDOVER CIBC INTRIA/
  └── Daily Handover/
      └── May/
          └── 01_May_2026/
              └── US_CAN-Shift-01-May-2026.xlsx (your existing file)
```

### Required Permissions

- Edit access to SharePoint site
- Power Automate Premium license (or included in M365)

---

## 🔧 Flow 1: Monthly Folder Creator

**Purpose**: Creates monthly folder (e.g., "May", "June")  
**Schedule**: 1st of every month at 6:00 AM IST

### Configuration Steps

#### 1. Create Flow
- Go to https://make.powerautomate.com
- Click **+ Create** → **Scheduled cloud flow**
- Name: `Create Monthly Shift Folder`
- Starting: First day of current month
- Repeat every: 1 Month
- Click **Create**

#### 2. Configure Recurrence Trigger
```
Trigger: Recurrence
├── Interval: 1
├── Frequency: Month
├── Time zone: (UTC+05:30) Chennai, Kolkata, Mumbai, New Delhi
├── At these hours: 6
├── At these minutes: 0
└── On these days: 1
```

#### 3. Initialize Variable - Month Name
```
Action: Initialize variable
├── Name: varMonthName
├── Type: String
└── Value: formatDateTime(convertFromUtc(utcNow(), 'India Standard Time'), 'MMMM')
```

#### 4. Initialize Variable - Folder Path
```
Action: Initialize variable
├── Name: varMonthFolderPath
├── Type: String
└── Value: /OC HANDOVER CIBC INTRIA/Daily Handover/@{variables('varMonthName')}
```

#### 5. Create Folder (SharePoint)
```
Action: Create new folder
├── Connector: SharePoint
├── Site Address: https://ibm-my.sharepoint.com/personal/b_vignesh19_ibm_com
├── Folder Path: /OC HANDOVER CIBC INTRIA/Daily Handover
└── Name: @{variables('varMonthName')}
```

**Configure run after**: Click 3 dots → Configure run after → Check all boxes (handles "already exists" error)

#### 6. Save and Test
- Click **Save**
- Click **Test** → **Manually** → **Run flow**
- Verify folder created in SharePoint

---

## 🔧 Flow 2: Daily Folder Creator

**Purpose**: Creates daily folder (e.g., "01_May_2026")  
**Schedule**: Every day at 6:15 AM IST

### Configuration Steps

#### 1. Create Flow
- Name: `Create Daily Shift Folder`
- Recurrence: Daily, 6:15 AM IST

#### 2. Configure Recurrence Trigger
```
Trigger: Recurrence
├── Interval: 1
├── Frequency: Day
├── Time zone: (UTC+05:30) Chennai, Kolkata, Mumbai, New Delhi
├── At these hours: 6
└── At these minutes: 15
```

#### 3. Initialize Variables

**Variable 1: Current Date IST**
```
Name: varCurrentDateIST
Type: String
Value: convertFromUtc(utcNow(), 'India Standard Time')
```

**Variable 2: Month Name**
```
Name: varMonthName
Type: String
Value: formatDateTime(variables('varCurrentDateIST'), 'MMMM')
```

**Variable 3: Day**
```
Name: varDay
Type: String
Value: formatDateTime(variables('varCurrentDateIST'), 'dd')
```

**Variable 4: Year**
```
Name: varYear
Type: String
Value: formatDateTime(variables('varCurrentDateIST'), 'yyyy')
```

**Variable 5: Daily Folder Name**
```
Name: varDailyFolderName
Type: String
Value: @{variables('varDay')}_@{variables('varMonthName')}_@{variables('varYear')}
```

#### 4. Create Daily Folder (SharePoint)
```
Action: Create new folder
├── Connector: SharePoint
├── Site Address: https://ibm-my.sharepoint.com/personal/b_vignesh19_ibm_com
├── Folder Path: /OC HANDOVER CIBC INTRIA/Daily Handover/@{variables('varMonthName')}
└── Name: @{variables('varDailyFolderName')}
```

**Configure run after**: Check all boxes

#### 5. Save and Test

---

## 🔧 Flow 3: India Shift 1 File Generator

**Purpose**: Creates India Shift 1 file by copying yesterday's US-CAN file  
**Schedule**: Every day at 6:30 AM IST

### Configuration Steps

#### 1. Create Flow
- Name: `Generate India Shift 1 File`
- Recurrence: Daily, 6:30 AM IST

#### 2. Configure Recurrence Trigger
```
Trigger: Recurrence
├── Interval: 1
├── Frequency: Day
├── Time zone: (UTC+05:30) Chennai, Kolkata, Mumbai, New Delhi
├── At these hours: 6
└── At these minutes: 30
```

#### 3. Initialize Variables for TODAY

```
varCurrentDateIST: convertFromUtc(utcNow(), 'India Standard Time')
varMonthName: formatDateTime(variables('varCurrentDateIST'), 'MMMM')
varDay: formatDateTime(variables('varCurrentDateIST'), 'dd')
varYear: formatDateTime(variables('varCurrentDateIST'), 'yyyy')
varDateString: formatDateTime(variables('varCurrentDateIST'), 'dd-MMM-yyyy')
varDailyFolderName: @{variables('varDay')}_@{variables('varMonthName')}_@{variables('varYear')}
```

#### 4. Initialize Variables for YESTERDAY

```
varYesterdayDate: addDays(variables('varCurrentDateIST'), -1)
varYesterdayMonth: formatDateTime(variables('varYesterdayDate'), 'MMMM')
varYesterdayDay: formatDateTime(variables('varYesterdayDate'), 'dd')
varYesterdayYear: formatDateTime(variables('varYesterdayDate'), 'yyyy')
varYesterdayDateString: formatDateTime(variables('varYesterdayDate'), 'dd-MMM-yyyy')
varYesterdayFolderName: @{variables('varYesterdayDay')}_@{variables('varYesterdayMonth')}_@{variables('varYesterdayYear')}
```

#### 5. Initialize File Names

```
varSourceFileName: US_CAN-Shift-@{variables('varYesterdayDateString')}.xlsx
varNewFileName: India_Shift_1_@{variables('varDateString')}.xlsx
```

#### 6. Initialize Paths

```
varSourcePath: /OC HANDOVER CIBC INTRIA/Daily Handover/@{variables('varYesterdayMonth')}/@{variables('varYesterdayFolderName')}/@{variables('varSourceFileName')}
varDestPath: /OC HANDOVER CIBC INTRIA/Daily Handover/@{variables('varMonthName')}/@{variables('varDailyFolderName')}
```

#### 7. Copy File (SharePoint)

```
Action: Copy file
├── Connector: SharePoint
├── Site Address: https://ibm-my.sharepoint.com/personal/b_vignesh19_ibm_com
├── Current File Path: @{variables('varSourcePath')}
├── Destination Site Address: https://ibm-my.sharepoint.com/personal/b_vignesh19_ibm_com
├── Destination Folder: @{variables('varDestPath')}
├── New Name: @{variables('varNewFileName')}
└── If another file exists: Replace
```

#### 8. Save and Test

---

## 🔧 Flow 4: India Shift 2 File Generator

**Purpose**: Creates India Shift 2 file by copying today's Shift 1 file  
**Schedule**: Every day at 2:30 PM IST

### Configuration Steps

**Same as Flow 3, with these key differences:**

#### 1. Recurrence Trigger
```
At these hours: 14 (2 PM)
At these minutes: 30
```

#### 2. Source File (from TODAY, not yesterday)

**No need for "yesterday" variables**

```
varSourceFileName: India_Shift_1_@{variables('varDateString')}.xlsx
varNewFileName: India_Shift_2_@{variables('varDateString')}.xlsx
varSourcePath: /OC HANDOVER CIBC INTRIA/Daily Handover/@{variables('varMonthName')}/@{variables('varDailyFolderName')}/@{variables('varSourceFileName')}
varDestPath: /OC HANDOVER CIBC INTRIA/Daily Handover/@{variables('varMonthName')}/@{variables('varDailyFolderName')}
```

**Note**: Source and destination are in the same folder!

---

## 🔧 Flow 5: US-Canada Shift File Generator

**Purpose**: Creates US-CAN file by copying today's Shift 2 file  
**Schedule**: Every day at 10:30 PM IST

### Configuration Steps

**Same as Flow 4, with these key differences:**

#### 1. Recurrence Trigger
```
At these hours: 22 (10 PM)
At these minutes: 30
```

#### 2. File Names

```
varSourceFileName: India_Shift_2_@{variables('varDateString')}.xlsx
varNewFileName: US_CAN-Shift-@{variables('varDateString')}.xlsx
```

---

## ✅ Testing Procedures

### Phase 1: Manual Setup (Day 1)

1. **Create Initial Structure**
   ```
   /Daily Handover/
     └── May/
         └── 01_May_2026/
             └── US_CAN-Shift-01-May-2026.xlsx
   ```

2. **Upload your existing file** to this location

### Phase 2: Test Flows Individually

#### Test Flow 1: Monthly Folder
- Manually trigger the flow
- Verify "May" folder created (or already exists)

#### Test Flow 2: Daily Folder
- Manually trigger the flow
- Verify today's folder created (e.g., "06_May_2026")

#### Test Flow 3: India Shift 1
- Manually trigger the flow
- Verify file copied from yesterday's US-CAN to today's India Shift 1
- Check file name: `India_Shift_1_06-May-2026.xlsx`

#### Test Flow 4: India Shift 2
- Manually trigger the flow
- Verify file copied from today's Shift 1 to Shift 2
- Check file name: `India_Shift_2_06-May-2026.xlsx`

#### Test Flow 5: US-CAN Shift
- Manually trigger the flow
- Verify file copied from today's Shift 2 to US-CAN
- Check file name: `US_CAN-Shift-06-May-2026.xlsx`

### Phase 3: Full Day Simulation

Run all flows in sequence:
1. Daily Folder Creator (6:15 AM)
2. India Shift 1 Generator (6:30 AM)
3. India Shift 2 Generator (2:30 PM)
4. US-CAN Shift Generator (10:30 PM)

**Expected Result:**
```
/Daily Handover/
  └── May/
      ├── 01_May_2026/
      │   └── US_CAN-Shift-01-May-2026.xlsx
      └── 06_May_2026/
          ├── India_Shift_1_06-May-2026.xlsx
          ├── India_Shift_2_06-May-2026.xlsx
          └── US_CAN-Shift-06-May-2026.xlsx
```

### Phase 4: Enable Automated Runs

1. Enable all 5 flows
2. Monitor for 24 hours
3. Verify files created at scheduled times
4. Check flow run history for any errors

---

## 🔍 Key Differences from Template Approach

| Aspect | Template Approach | Chain-Copy Approach |
|--------|------------------|---------------------|
| Source | Static template file | Previous shift's file |
| Data | Empty/template data | Carries over from previous shift |
| Storage | Needs template folder | No template needed |
| Continuity | Each file independent | Files linked in sequence |
| Month Transition | No special handling | Needs yesterday's month check |

---

## 🆘 Troubleshooting

### Issue: Flow 3 fails on 1st of month

**Problem**: Yesterday's file is in previous month's folder

**Solution**: Add condition to check if yesterday's month ≠ today's month
```
If varYesterdayMonth ≠ varMonthName:
  Use previous month's folder path
```

### Issue: File not found error

**Causes**:
- File name doesn't match exactly
- Folder path incorrect
- Previous flow didn't run

**Solutions**:
1. Check exact file name in SharePoint
2. Verify date format matches (dd-MMM-yyyy)
3. Ensure previous shift's flow completed successfully
4. Check flow run history for errors

### Issue: Permission denied

**Causes**:
- Insufficient SharePoint permissions
- Connection expired

**Solutions**:
1. Verify you have Edit access to SharePoint site
2. Reconnect SharePoint connector in Power Automate
3. Check connection status in Power Automate

### Issue: Wrong file copied

**Causes**:
- Date calculation error
- Time zone mismatch

**Solutions**:
1. Verify time zone is set to India Standard Time
2. Test date variables with Compose actions
3. Check if DST affects calculations

---

## 📊 Daily Timeline

```
6:00 AM  → Monthly Folder Created (1st of month only)
6:15 AM  → Daily Folder Created
6:30 AM  → India Shift 1 File Created (from yesterday's US-CAN)
2:30 PM  → India Shift 2 File Created (from today's Shift 1)
10:30 PM → US-CAN Shift File Created (from today's Shift 2)
```

---

## 🔐 Security Considerations

- Use organizational account for automation
- Enable MFA for automation account
- Regular permission audits
- Monitor flow run history
- Set up failure alerts

---

## 💡 Best Practices

1. **Test thoroughly** before enabling automated runs
2. **Monitor daily** for the first week
3. **Keep backups** of important files
4. **Document changes** to flows
5. **Set up email alerts** for flow failures
6. **Review monthly** for optimization opportunities

---

## 📞 Support Resources

- [Power Automate Documentation](https://docs.microsoft.com/power-automate/)
- [SharePoint Connector Reference](https://docs.microsoft.com/connectors/sharepointonline/)
- [Power Automate Community](https://powerusers.microsoft.com/t5/Power-Automate-Community/ct-p/MPACommunity)

---

## 📝 Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | May 6, 2026 | Initial SharePoint chain-copy workflow documentation |

---

**Last Updated**: May 6, 2026  
**Document Owner**: IT Team  
**Review Frequency**: Quarterly