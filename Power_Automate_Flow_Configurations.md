# Power Automate Flow Configurations - Step-by-Step Guide

## Table of Contents
1. [Prerequisites Setup](#prerequisites-setup)
2. [Flow 1: Monthly Folder Creator](#flow-1-monthly-folder-creator)
3. [Flow 2: Daily Folder Creator](#flow-2-daily-folder-creator)
4. [Flow 3: Shift 1 File Generator](#flow-3-shift-1-file-generator)
5. [Flow 4: Shift 2 File Generator](#flow-4-shift-2-file-generator)
6. [Flow 5: Shift 3 File Generator](#flow-5-shift-3-file-generator)
7. [Testing Procedures](#testing-procedures)

---

## Prerequisites Setup

### Step 1: Create OneDrive Folder Structure

1. Open OneDrive for Business
2. Create the following structure:
   ```
   /Shift_Files/
     └── Templates/
   ```

### Step 2: Create Master Template File

1. Create a new Excel file with your desired structure
2. Add headers, formulas, formatting, and any required sheets
3. Save as `Shift_Template.xlsx`
4. Upload to `/Shift_Files/Templates/Shift_Template.xlsx`

### Step 3: Access Power Automate

1. Go to [Power Automate Portal](https://make.powerautomate.com)
2. Sign in with your Microsoft 365 account
3. Ensure you have appropriate license

---

## Flow 1: Monthly Folder Creator

### Purpose
Creates a new folder for each month (e.g., "May", "June")

### Configuration Steps

#### 1. Create New Flow
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
├── At these hours: 6 (6 AM)
├── At these minutes: 0
└── On these days: 1
```

#### 3. Add Action: Initialize Variable - Month Name
```
Action: Initialize variable
├── Name: varMonthName
├── Type: String
└── Value: formatDateTime(convertFromUtc(utcNow(), 'India Standard Time'), 'MMMM')
```

#### 4. Add Action: Initialize Variable - Month Folder Path
```
Action: Initialize variable
├── Name: varMonthFolderPath
├── Type: String
└── Value: /Shift_Files/@{variables('varMonthName')}
```

#### 5. Add Action: Get Folder Metadata (with error handling)
```
Action: Get folder metadata using path (OneDrive for Business)
├── File Path: @{variables('varMonthFolderPath')}
└── Configure run after: (leave default)
```

#### 6. Add Scope: Error Handling
```
Action: Scope
└── Name: Try Create Folder
```

Inside the Scope, add:

**6a. Condition: Check if folder exists**
```
Action: Condition
├── Choose a value: outputs('Get_folder_metadata_using_path')?['body/IsFolder']
├── is equal to: true
└── If yes: Do nothing (folder already exists)
```

**6b. If no: Create folder**
```
Action: Create new folder (OneDrive for Business)
├── Folder Path: /Shift_Files
└── Name: @{variables('varMonthName')}
```

#### 7. Add Action: Send Email Notification (Optional)
```
Action: Send an email (V2)
├── To: your-email@company.com
├── Subject: Monthly Shift Folder Created - @{variables('varMonthName')}
└── Body: The monthly folder for @{variables('varMonthName')} has been created successfully.
```

#### 8. Configure Error Handling
- Click on the Scope action
- Click **...** → **Configure run after**
- Check: **has failed**, **has timed out**, **is skipped**
- Add action: Send error notification email

#### 9. Save and Test
- Click **Save**
- Click **Test** → **Manually**
- Verify folder created in OneDrive

---

## Flow 2: Daily Folder Creator

### Purpose
Creates a daily folder with format: `DD_Month_YYYY` (e.g., "01_May_2026")

### Configuration Steps

#### 1. Create New Flow
- Click **+ Create** → **Scheduled cloud flow**
- Name: `Create Daily Shift Folder`
- Starting: Today
- Repeat every: 1 Day
- Click **Create**

#### 2. Configure Recurrence Trigger
```
Trigger: Recurrence
├── Interval: 1
├── Frequency: Day
├── Time zone: (UTC+05:30) Chennai, Kolkata, Mumbai, New Delhi
├── At these hours: 6 (6 AM)
└── At these minutes: 15
```

#### 3. Add Action: Initialize Variable - Current Date IST
```
Action: Initialize variable
├── Name: varCurrentDateIST
├── Type: String
└── Value: convertFromUtc(utcNow(), 'India Standard Time')
```

#### 4. Add Action: Initialize Variable - Month Name
```
Action: Initialize variable
├── Name: varMonthName
├── Type: String
└── Value: formatDateTime(variables('varCurrentDateIST'), 'MMMM')
```

#### 5. Add Action: Initialize Variable - Day
```
Action: Initialize variable
├── Name: varDay
├── Type: String
└── Value: formatDateTime(variables('varCurrentDateIST'), 'dd')
```

#### 6. Add Action: Initialize Variable - Year
```
Action: Initialize variable
├── Name: varYear
├── Type: String
└── Value: formatDateTime(variables('varCurrentDateIST'), 'yyyy')
```

#### 7. Add Action: Initialize Variable - Daily Folder Name
```
Action: Initialize variable
├── Name: varDailyFolderName
├── Type: String
└── Value: @{variables('varDay')}_@{variables('varMonthName')}_@{variables('varYear')}
```
Example output: `01_May_2026`

#### 8. Add Action: Initialize Variable - Monthly Folder Path
```
Action: Initialize variable
├── Name: varMonthlyFolderPath
├── Type: String
└── Value: /Shift_Files/@{variables('varMonthName')}
```

#### 9. Add Action: Initialize Variable - Daily Folder Path
```
Action: Initialize variable
├── Name: varDailyFolderPath
├── Type: String
└── Value: @{variables('varMonthlyFolderPath')}/@{variables('varDailyFolderName')}
```

#### 10. Add Scope: Check and Create Monthly Folder
```
Action: Scope
└── Name: Ensure Monthly Folder Exists
```

Inside the Scope:

**10a. Get Monthly Folder Metadata**
```
Action: Get folder metadata using path (OneDrive for Business)
└── File Path: @{variables('varMonthlyFolderPath')}
```

**10b. Condition: Monthly Folder Exists**
```
Action: Condition
├── Choose a value: outputs('Get_folder_metadata_using_path')?['body/IsFolder']
└── is equal to: true
```

**10c. If no: Create Monthly Folder**
```
Action: Create new folder (OneDrive for Business)
├── Folder Path: /Shift_Files
└── Name: @{variables('varMonthName')}
```

Configure run after on Condition to continue even if Get Folder fails.

#### 11. Add Action: Create Daily Folder
```
Action: Create new folder (OneDrive for Business)
├── Folder Path: @{variables('varMonthlyFolderPath')}
└── Name: @{variables('varDailyFolderName')}
```

Configure run after to handle "folder already exists" error gracefully.

#### 12. Add Action: Send Success Notification (Optional)
```
Action: Send an email (V2)
├── To: your-email@company.com
├── Subject: Daily Shift Folder Created - @{variables('varDailyFolderName')}
└── Body: Daily folder created at: @{variables('varDailyFolderPath')}
```

#### 13. Save and Test
- Click **Save**
- Click **Test** → **Manually**
- Verify folder structure in OneDrive

---

## Flow 3: Shift 1 File Generator

### Purpose
Creates Shift 1 Excel file at 6:00 AM IST daily

### Configuration Steps

#### 1. Create New Flow
- Click **+ Create** → **Scheduled cloud flow**
- Name: `Generate Shift 1 File`
- Starting: Today
- Repeat every: 1 Day
- Click **Create**

#### 2. Configure Recurrence Trigger
```
Trigger: Recurrence
├── Interval: 1
├── Frequency: Day
├── Time zone: (UTC+05:30) Chennai, Kolkata, Mumbai, New Delhi
├── At these hours: 6 (6 AM)
└── At these minutes: 30
```

#### 3. Add Action: Initialize Variable - Current Date IST
```
Action: Initialize variable
├── Name: varCurrentDateIST
├── Type: String
└── Value: convertFromUtc(utcNow(), 'India Standard Time')
```

#### 4. Add Action: Initialize Variable - Month Name
```
Action: Initialize variable
├── Name: varMonthName
├── Type: String
└── Value: formatDateTime(variables('varCurrentDateIST'), 'MMMM')
```

#### 5. Add Action: Initialize Variable - Day
```
Action: Initialize variable
├── Name: varDay
├── Type: String
└── Value: formatDateTime(variables('varCurrentDateIST'), 'dd')
```

#### 6. Add Action: Initialize Variable - Year
```
Action: Initialize variable
├── Name: varYear
├── Type: String
└── Value: formatDateTime(variables('varCurrentDateIST'), 'yyyy')
```

#### 7. Add Action: Initialize Variable - Date String
```
Action: Initialize variable
├── Name: varDateString
├── Type: String
└── Value: formatDateTime(variables('varCurrentDateIST'), 'dd-MMM-yyyy')
```
Example output: `01-May-2026`

#### 8. Add Action: Initialize Variable - Daily Folder Name
```
Action: Initialize variable
├── Name: varDailyFolderName
├── Type: String
└── Value: @{variables('varDay')}_@{variables('varMonthName')}_@{variables('varYear')}
```

#### 9. Add Action: Initialize Variable - File Name
```
Action: Initialize variable
├── Name: varFileName
├── Type: String
└── Value: @{variables('varMonthName')}_India_Shift_1_@{variables('varDateString')}.xlsx
```
Example output: `May_India_Shift_1_01-May-2026.xlsx`

#### 10. Add Action: Initialize Variable - Destination Path
```
Action: Initialize variable
├── Name: varDestPath
├── Type: String
└── Value: /Shift_Files/@{variables('varMonthName')}/@{variables('varDailyFolderName')}
```

#### 11. Add Action: Initialize Variable - Full File Path
```
Action: Initialize variable
├── Name: varFullFilePath
├── Type: String
└── Value: @{variables('varDestPath')}/@{variables('varFileName')}
```

#### 12. Add Action: Get Template File Content
```
Action: Get file content (OneDrive for Business)
└── File: /Shift_Files/Templates/Shift_Template.xlsx
```

#### 13. Add Scope: Check if File Already Exists
```
Action: Scope
└── Name: Check File Existence
```

Inside the Scope:

**13a. Get File Metadata**
```
Action: Get file metadata using path (OneDrive for Business)
└── File Path: @{variables('varFullFilePath')}
```

**13b. Condition: File Exists**
```
Action: Condition
├── Choose a value: outputs('Get_file_metadata_using_path')?['body/IsFolder']
└── is equal to: false
```

Configure run after to continue even if Get File fails (file doesn't exist).

#### 14. Add Action: Create File (Only if doesn't exist)
```
Action: Create file (OneDrive for Business)
├── Folder Path: @{variables('varDestPath')}
├── File Name: @{variables('varFileName')}
└── File Content: body('Get_file_content')
```

Place this action in the "If no" branch of the condition (file doesn't exist).

#### 15. Add Action: Send Success Notification (Optional)
```
Action: Send an email (V2)
├── To: shift-manager@company.com
├── Subject: Shift 1 File Created - @{variables('varDateString')}
└── Body: 
    Shift 1 file has been created:
    File: @{variables('varFileName')}
    Location: @{variables('varDestPath')}
    Time: @{convertFromUtc(utcNow(), 'India Standard Time')}
```

#### 16. Add Error Handling
Add a parallel branch with "Configure run after" set to handle failures:

```
Action: Send an email (V2)
├── To: admin@company.com
├── Subject: ERROR - Shift 1 File Creation Failed
└── Body: 
    Failed to create Shift 1 file.
    Date: @{variables('varDateString')}
    Error: Check flow run history for details.
```

#### 17. Save and Test
- Click **Save**
- Click **Test** → **Manually**
- Verify file created in correct location with correct name
- Check that template formatting is preserved

---

## Flow 4: Shift 2 File Generator

### Purpose
Creates Shift 2 Excel file at 2:00 PM IST daily

### Configuration Steps

**Follow the exact same steps as Flow 3 (Shift 1), with these changes:**

#### Changes from Shift 1:

1. **Flow Name**: `Generate Shift 2 File`

2. **Recurrence Trigger**:
   ```
   At these hours: 14 (2 PM)
   At these minutes: 30
   ```

3. **File Name Variable** (Step 9):
   ```
   Value: @{variables('varMonthName')}_India_Shift_2_@{variables('varDateString')}.xlsx
   ```
   Example output: `May_India_Shift_2_01-May-2026.xlsx`

4. **Email Subject** (Step 15):
   ```
   Subject: Shift 2 File Created - @{variables('varDateString')}
   ```

All other steps remain identical to Shift 1 flow.

---

## Flow 5: Shift 3 File Generator

### Purpose
Creates Shift 3 Excel file at 10:00 PM IST daily

### Configuration Steps

**Follow the exact same steps as Flow 3 (Shift 1), with these changes:**

#### Changes from Shift 1:

1. **Flow Name**: `Generate Shift 3 File`

2. **Recurrence Trigger**:
   ```
   At these hours: 22 (10 PM)
   At these minutes: 30
   ```

3. **File Name Variable** (Step 9):
   ```
   Value: @{variables('varMonthName')}_India_Shift_3_@{variables('varDateString')}.xlsx
   ```
   Example output: `May_India_Shift_3_01-May-2026.xlsx`

4. **Email Subject** (Step 15):
   ```
   Subject: Shift 3 File Created - @{variables('varDateString')}
   ```

All other steps remain identical to Shift 1 flow.

---

## Testing Procedures

### Phase 1: Individual Flow Testing

#### Test 1: Monthly Folder Creator
1. Open the flow in Power Automate
2. Click **Test** → **Manually**
3. Click **Run flow**
4. Wait for completion
5. Verify in OneDrive:
   - Folder `/Shift_Files/May` exists
   - Check email notification received

#### Test 2: Daily Folder Creator
1. Open the flow in Power Automate
2. Click **Test** → **Manually**
3. Click **Run flow**
4. Wait for completion
5. Verify in OneDrive:
   - Folder `/Shift_Files/May/01_May_2026` exists
   - Monthly folder created if didn't exist
   - Check email notification received

#### Test 3: Shift 1 File Generator
1. Ensure daily folder exists (run Test 2 first)
2. Open the flow in Power Automate
3. Click **Test** → **Manually**
4. Click **Run flow**
5. Wait for completion
6. Verify in OneDrive:
   - File `May_India_Shift_1_01-May-2026.xlsx` exists
   - File is in correct location
   - Open file and verify template formatting preserved
   - Check email notification received

#### Test 4: Shift 2 File Generator
1. Follow same steps as Test 3
2. Verify file name: `May_India_Shift_2_01-May-2026.xlsx`

#### Test 5: Shift 3 File Generator
1. Follow same steps as Test 3
2. Verify file name: `May_India_Shift_3_01-May-2026.xlsx`

### Phase 2: Integration Testing

#### Test 6: Full Day Simulation
1. Manually trigger flows in sequence:
   - Daily Folder Creator (12:05 AM)
   - Shift 1 Generator (6:00 AM)
   - Shift 2 Generator (2:00 PM)
   - Shift 3 Generator (10:00 PM)
2. Verify complete folder structure:
   ```
   /Shift_Files/May/01_May_2026/
   ├── May_India_Shift_1_01-May-2026.xlsx
   ├── May_India_Shift_2_01-May-2026.xlsx
   └── May_India_Shift_3_01-May-2026.xlsx
   ```

#### Test 7: Duplicate Prevention
1. Run Shift 1 flow twice
2. Verify only one file exists (no duplicates)
3. Check flow run history for proper handling

#### Test 8: Month Transition
1. Set system date to last day of month (or wait)
2. Run Monthly Folder Creator
3. Run Daily Folder Creator
4. Verify new month folder created
5. Verify daily folder in new month

### Phase 3: Automated Testing

#### Test 9: 3-Day Monitoring
1. Enable all flows
2. Monitor for 3 consecutive days
3. Check flow run history daily
4. Verify files created at correct times
5. Document any issues

#### Test 10: Error Scenario Testing

**Scenario A: Template File Missing**
1. Temporarily rename template file
2. Trigger Shift 1 flow
3. Verify error handling works
4. Check error notification received
5. Restore template file

**Scenario B: Insufficient Permissions**
1. Remove write permissions (if possible in test environment)
2. Trigger Daily Folder flow
3. Verify error handling
4. Restore permissions

**Scenario C: OneDrive Offline**
1. Disconnect network (if testing locally)
2. Trigger any flow
3. Verify retry logic or error handling
4. Reconnect and verify recovery

### Testing Checklist

- [ ] Monthly folder created correctly
- [ ] Daily folder created with correct naming
- [ ] Shift 1 file created at 6:00 AM IST
- [ ] Shift 2 file created at 2:00 PM IST
- [ ] Shift 3 file created at 10:00 PM IST
- [ ] Template formatting preserved in all files
- [ ] No duplicate files created
- [ ] Email notifications working
- [ ] Error handling working correctly
- [ ] Month transition handled properly
- [ ] All flows enabled and scheduled
- [ ] Flow run history reviewed
- [ ] Documentation updated

---

## Quick Reference: Power Automate Expressions

### Date/Time Expressions

**Current time in IST**:
```
convertFromUtc(utcNow(), 'India Standard Time')
```

**Month name (May)**:
```
formatDateTime(convertFromUtc(utcNow(), 'India Standard Time'), 'MMMM')
```

**Day (01)**:
```
formatDateTime(convertFromUtc(utcNow(), 'India Standard Time'), 'dd')
```

**Year (2026)**:
```
formatDateTime(convertFromUtc(utcNow(), 'India Standard Time'), 'yyyy')
```

**Date string (01-May-2026)**:
```
formatDateTime(convertFromUtc(utcNow(), 'India Standard Time'), 'dd-MMM-yyyy')
```

### Path Construction

**Monthly folder path**:
```
/Shift_Files/@{variables('varMonthName')}
```

**Daily folder name**:
```
@{variables('varDay')}_@{variables('varMonthName')}_@{variables('varYear')}
```

**Full file path**:
```
@{variables('varDestPath')}/@{variables('varFileName')}
```

---

## Troubleshooting Tips

### Issue: Flow not triggering
- Check flow is enabled (toggle in flow details)
- Verify time zone setting
- Check license status

### Issue: Wrong date/time
- Always use `convertFromUtc()` with 'India Standard Time'
- Test expressions in flow checker
- Add logging variables to debug

### Issue: File not created
- Check template file exists
- Verify folder path is correct
- Review flow run history for errors
- Check OneDrive permissions

### Issue: Duplicate files
- Verify file existence check is working
- Check flow trigger history
- Ensure only one instance running

---

## Maintenance Schedule

### Daily
- Monitor flow run history
- Check for any failures

### Weekly
- Review all flow runs
- Verify file creation consistency
- Check storage usage

### Monthly
- Audit folder structure
- Update template if needed
- Review and optimize flows

---

## Support Contacts

- **Power Automate Support**: [Microsoft Support](https://support.microsoft.com)
- **Community Forums**: [Power Automate Community](https://powerusers.microsoft.com/t5/Power-Automate-Community/ct-p/MPACommunity)
- **Documentation**: [Power Automate Docs](https://docs.microsoft.com/power-automate/)

---

## Conclusion

This guide provides complete step-by-step instructions for implementing all five Power Automate flows. Follow each section carefully, test thoroughly, and refer to the troubleshooting section if issues arise.

Remember to:
- Test each flow individually before enabling all
- Monitor closely for the first week
- Keep template file backed up
- Document any customizations made
- Train team members on the system

Good luck with your implementation!