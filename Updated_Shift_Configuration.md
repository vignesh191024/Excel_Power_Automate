# Updated Shift Configuration - Corrected Naming and Timings

## Overview

This document contains the updated configuration with the correct shift naming conventions and timings.

## Updated Shift Details

### Shift 1 (India Shift)
- **Timing**: 6:30 AM IST
- **File Name Format**: `{Month}_India_Shift_1_{DD-MMM-YYYY}.xlsx`
- **Example**: `May_India_Shift_1_05-May-2026.xlsx`

### Shift 2 (India Shift)
- **Timing**: 2:30 PM IST (14:30)
- **File Name Format**: `{Month}_India_Shift_2_{DD-MMM-YYYY}.xlsx`
- **Example**: `May_India_Shift_2_05-May-2026.xlsx`

### Shift 3 (US-Canada Shift) ⭐ UPDATED
- **Timing**: 10:30 PM IST (22:30)
- **File Name Format**: `US-CAN-SHIFT_{DD-MMM-YYYY}.xlsx`
- **Example**: `US-CAN-SHIFT_05-May-2026.xlsx`

## Updated Folder Structure

```
OneDrive/Shift_Files/
└── May/
    └── 05_May_2026/
        ├── May_India_Shift_1_05-May-2026.xlsx    (Created at 6:30 AM IST)
        ├── May_India_Shift_2_05-May-2026.xlsx    (Created at 2:30 PM IST)
        └── US-CAN-SHIFT_05-May-2026.xlsx         (Created at 10:30 PM IST)
```

## Updated Power Automate Flow Configurations

### Flow 1: Monthly Folder Creator
**No changes needed** - Same as original configuration

### Flow 2: Daily Folder Creator
**No changes needed** - Same as original configuration

### Flow 3: Shift 1 File Generator (India)

#### Updated Recurrence Trigger
```
Trigger: Recurrence
├── Interval: 1
├── Frequency: Day
├── Time zone: (UTC+05:30) Chennai, Kolkata, Mumbai, New Delhi
├── At these hours: 6
└── At these minutes: 30    ⭐ CHANGED from 0 to 30
```

#### File Name Variable (No change)
```
Action: Initialize variable
├── Name: varFileName
├── Type: String
└── Value: @{variables('varMonthName')}_India_Shift_1_@{variables('varDateString')}.xlsx
```
Example output: `May_India_Shift_1_05-May-2026.xlsx`

### Flow 4: Shift 2 File Generator (India)

#### Updated Recurrence Trigger
```
Trigger: Recurrence
├── Interval: 1
├── Frequency: Day
├── Time zone: (UTC+05:30) Chennai, Kolkata, Mumbai, New Delhi
├── At these hours: 14
└── At these minutes: 30    ⭐ CHANGED from 0 to 30
```

#### File Name Variable (No change)
```
Action: Initialize variable
├── Name: varFileName
├── Type: String
└── Value: @{variables('varMonthName')}_India_Shift_2_@{variables('varDateString')}.xlsx
```
Example output: `May_India_Shift_2_05-May-2026.xlsx`

### Flow 5: Shift 3 File Generator (US-Canada) ⭐ MAJOR CHANGES

#### Updated Recurrence Trigger
```
Trigger: Recurrence
├── Interval: 1
├── Frequency: Day
├── Time zone: (UTC+05:30) Chennai, Kolkata, Mumbai, New Delhi
├── At these hours: 22
└── At these minutes: 30    ⭐ CHANGED from 0 to 30
```

#### Updated File Name Variable ⭐ NEW FORMAT
```
Action: Initialize variable
├── Name: varFileName
├── Type: String
└── Value: US-CAN-SHIFT_@{variables('varDateString')}.xlsx
```
Example output: `US-CAN-SHIFT_05-May-2026.xlsx`

**Note**: The US-Canada shift file name does NOT include the month name or shift number, just the date.

## Complete Flow 5 Configuration (US-Canada Shift)

### Step-by-Step Configuration

#### 1. Create New Flow
- Name: `Generate US-Canada Shift File`
- Type: Scheduled cloud flow

#### 2. Configure Recurrence Trigger
```
Trigger: Recurrence
├── Interval: 1
├── Frequency: Day
├── Time zone: (UTC+05:30) Chennai, Kolkata, Mumbai, New Delhi
├── At these hours: 22 (10 PM)
└── At these minutes: 30
```

#### 3. Initialize Variable - Current Date IST
```
Action: Initialize variable
├── Name: varCurrentDateIST
├── Type: String
└── Value: convertFromUtc(utcNow(), 'India Standard Time')
```

#### 4. Initialize Variable - Month Name
```
Action: Initialize variable
├── Name: varMonthName
├── Type: String
└── Value: formatDateTime(variables('varCurrentDateIST'), 'MMMM')
```

#### 5. Initialize Variable - Day
```
Action: Initialize variable
├── Name: varDay
├── Type: String
└── Value: formatDateTime(variables('varCurrentDateIST'), 'dd')
```

#### 6. Initialize Variable - Year
```
Action: Initialize variable
├── Name: varYear
├── Type: String
└── Value: formatDateTime(variables('varCurrentDateIST'), 'yyyy')
```

#### 7. Initialize Variable - Date String
```
Action: Initialize variable
├── Name: varDateString
├── Type: String
└── Value: formatDateTime(variables('varCurrentDateIST'), 'dd-MMM-yyyy')
```
Example output: `05-May-2026`

#### 8. Initialize Variable - Daily Folder Name
```
Action: Initialize variable
├── Name: varDailyFolderName
├── Type: String
└── Value: @{variables('varDay')}_@{variables('varMonthName')}_@{variables('varYear')}
```

#### 9. Initialize Variable - File Name ⭐ UPDATED FORMAT
```
Action: Initialize variable
├── Name: varFileName
├── Type: String
└── Value: US-CAN-SHIFT_@{variables('varDateString')}.xlsx
```
Example output: `US-CAN-SHIFT_05-May-2026.xlsx`

#### 10. Initialize Variable - Destination Path
```
Action: Initialize variable
├── Name: varDestPath
├── Type: String
└── Value: /Shift_Files/@{variables('varMonthName')}/@{variables('varDailyFolderName')}
```

#### 11. Initialize Variable - Full File Path
```
Action: Initialize variable
├── Name: varFullFilePath
├── Type: String
└── Value: @{variables('varDestPath')}/@{variables('varFileName')}
```

#### 12-17. Remaining Actions
Same as Shift 1 and Shift 2 flows:
- Get template file content
- Check if file exists
- Create file if doesn't exist
- Send notifications
- Error handling

## Updated Daily Timeline

```
Time (IST)    | Action
--------------|--------------------------------------------------
12:01 AM      | Create monthly folder (1st of month only)
12:05 AM      | Create daily folder
6:30 AM       | Create India Shift 1 file
2:30 PM       | Create India Shift 2 file
10:30 PM      | Create US-Canada Shift file
```

## Updated Naming Convention Summary

| Shift | Region | File Name Format | Example |
|-------|--------|------------------|---------|
| 1 | India | `{Month}_India_Shift_1_{Date}.xlsx` | `May_India_Shift_1_05-May-2026.xlsx` |
| 2 | India | `{Month}_India_Shift_2_{Date}.xlsx` | `May_India_Shift_2_05-May-2026.xlsx` |
| 3 | US-Canada | `US-CAN-SHIFT_{Date}.xlsx` | `US-CAN-SHIFT_05-May-2026.xlsx` |

## Key Differences for Shift 3 (US-Canada)

1. **No Month Name**: File name starts with `US-CAN-SHIFT_` instead of month name
2. **No Shift Number**: No "Shift_3" in the name
3. **Region Identifier**: Uses `US-CAN` to identify the region
4. **Timing**: 10:30 PM IST (30 minutes later than originally planned)

## Power Automate Expression for Shift 3 File Name

```
US-CAN-SHIFT_@{formatDateTime(convertFromUtc(utcNow(), 'India Standard Time'), 'dd-MMM-yyyy')}.xlsx
```

Or using variables:
```
US-CAN-SHIFT_@{variables('varDateString')}.xlsx
```

## Updated Testing Checklist

### Test Shift 1 (India)
- [ ] Trigger at 6:30 AM IST
- [ ] File name: `May_India_Shift_1_05-May-2026.xlsx`
- [ ] File created in correct location
- [ ] Template formatting preserved

### Test Shift 2 (India)
- [ ] Trigger at 2:30 PM IST
- [ ] File name: `May_India_Shift_2_05-May-2026.xlsx`
- [ ] File created in correct location
- [ ] Template formatting preserved

### Test Shift 3 (US-Canada) ⭐
- [ ] Trigger at 10:30 PM IST
- [ ] File name: `US-CAN-SHIFT_05-May-2026.xlsx` (correct format)
- [ ] File created in correct location
- [ ] Template formatting preserved
- [ ] No month name in file name
- [ ] No shift number in file name

## Updated Email Notification for Shift 3

```
Action: Send an email (V2)
├── To: shift-manager@company.com
├── Subject: US-Canada Shift File Created - @{variables('varDateString')}
└── Body: 
    US-Canada shift file has been created:
    File: @{variables('varFileName')}
    Location: @{variables('varDestPath')}
    Time: @{convertFromUtc(utcNow(), 'India Standard Time')}
```

## Migration Notes

If you've already created flows with the old configuration:

### For Shift 1 and Shift 2
1. Edit the flow
2. Update the Recurrence trigger
3. Change "At these minutes" from `0` to `30`
4. Save and test

### For Shift 3 (US-Canada)
1. Edit the flow
2. Update the Recurrence trigger - change minutes to `30`
3. Update the `varFileName` variable:
   - Old: `@{variables('varMonthName')}_India_Shift_3_@{variables('varDateString')}.xlsx`
   - New: `US-CAN-SHIFT_@{variables('varDateString')}.xlsx`
4. Update email notification subject and body
5. Save and test thoroughly

## Quick Reference Card

### Shift Timings (IST)
- **Shift 1 (India)**: 6:30 AM
- **Shift 2 (India)**: 2:30 PM
- **Shift 3 (US-Canada)**: 10:30 PM

### File Name Patterns
- **Shift 1**: `May_India_Shift_1_05-May-2026.xlsx`
- **Shift 2**: `May_India_Shift_2_05-May-2026.xlsx`
- **Shift 3**: `US-CAN-SHIFT_05-May-2026.xlsx`

### Power Automate Trigger Times
- **Shift 1**: Hour: 6, Minute: 30
- **Shift 2**: Hour: 14, Minute: 30
- **Shift 3**: Hour: 22, Minute: 30

## Important Notes

1. **All times are in IST** (India Standard Time, UTC+5:30)
2. **Shift 3 has a different naming convention** - no month name, no shift number
3. **All shifts trigger 30 minutes past the hour** (6:30, 14:30, 22:30)
4. **US-CAN-SHIFT represents the US-Canada region** shift
5. **Date format remains consistent** across all shifts: `DD-MMM-YYYY`

## Verification Steps

After implementing the changes:

1. **Check Flow Names**:
   - Flow 3: "Generate Shift 1 File" or "Generate India Shift 1 File"
   - Flow 4: "Generate Shift 2 File" or "Generate India Shift 2 File"
   - Flow 5: "Generate US-Canada Shift File" ⭐

2. **Verify Trigger Times**:
   - All should show `:30` minutes
   - Hours: 6, 14, 22

3. **Test File Names**:
   - Manually trigger each flow
   - Verify file names match the new format
   - Especially check Shift 3 has no month name

4. **Check Folder Structure**:
   - All three files should be in the same daily folder
   - Example: `/Shift_Files/May/05_May_2026/`

## Summary of Changes

| Item | Old Value | New Value |
|------|-----------|-----------|
| Shift 1 Time | 6:00 AM | 6:30 AM |
| Shift 2 Time | 2:00 PM | 2:30 PM |
| Shift 3 Time | 10:00 PM | 10:30 PM |
| Shift 3 Name | `May_India_Shift_3_05-May-2026.xlsx` | `US-CAN-SHIFT_05-May-2026.xlsx` |

---

**Last Updated**: May 2, 2026  
**Version**: 1.1 (Updated Configuration)