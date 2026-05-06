# Shift Files Automation - Complete Documentation

## 📋 Overview

This project provides a complete solution for automating the creation of shift-based Excel files using Power Automate and SharePoint. The system automatically generates three shift files daily using a **chain-copy workflow**, organized in a hierarchical folder structure by month and date.

## 🎯 What This System Does

**Automated File Generation using Chain-Copy Workflow**:
- Creates yearly folders (e.g., "2026", "2027") at 5:30 AM on January 1st
- Creates monthly folders (e.g., "May", "June") at 6:00 AM on 1st of month inside year folder
- Creates daily folders with format `DD_Month_YYYY` (e.g., "06_May_2026") at 6:15 AM daily inside year/month
- Generates 3 shift files per day by **copying and renaming** the previous shift's file:
  - **6:30 AM**: India Shift 1 (copies yesterday's US-CAN file)
  - **2:30 PM**: India Shift 2 (copies today's Shift 1 file)
  - **10:30 PM**: US-CAN Shift (copies today's Shift 2 file)

**Example Output Structure**:
```
SharePoint/Daily Handover/
└── 2026/                           (Year folder - created Jan 1 at 5:30 AM)
    └── May/                        (Month folder - created May 1 at 6:00 AM)
        └── 02_May_2026/            (Daily folder - created daily at 6:15 AM)
            ├── India_Shift_1_02-May-2026.xlsx (6:30 AM - copied from US_CAN-Shift-01-May-2026.xlsx)
            ├── India_Shift_2_02-May-2026.xlsx (2:30 PM - copied from India_Shift_1_02-May-2026.xlsx)
            └── US_CAN-Shift-02-May-2026.xlsx (10:30 PM - copied from India_Shift_2_02-May-2026.xlsx)
```

**Key Feature**: Each file is created by copying the previous shift's file, maintaining data continuity across shifts.

## 📚 Documentation Files

This repository contains comprehensive documentation for implementing and maintaining the automation system:

### 1. **SharePoint_Chain_Copy_Workflow_Guide.md** ⭐ PRIMARY GUIDE
   - Complete chain-copy workflow explanation
   - Step-by-step SharePoint flow configurations
   - Detailed variable setup for date calculations
   - Copy file actions with proper paths
   - Testing procedures
   - Troubleshooting for chain-copy scenarios

### 2. **OneDrive_Shift_Files_Automation_Plan.md** (Legacy - Template Approach)
   - Original template-based system architecture
   - Alternative approach using static templates
   - Reference for understanding different approaches

### 3. **Power_Automate_Flow_Configurations.md** (Legacy - Template Approach)
   - Original template-based flow configurations
   - Reference for OneDrive connector usage

### 4. **Quick_Reference_Guide.md**
   - Visual diagrams and flowcharts
   - Daily timeline
   - Naming conventions
   - Common tasks
   - Troubleshooting quick fixes
   - Monitoring dashboard
   - Maintenance checklist

### 5. **Implementation_Checklist.md**
   - Phase-by-phase implementation guide
   - Checkboxes for tracking progress
   - Testing procedures
   - Deployment steps
   - Training and handover
   - Sign-off section

### 6. **Using_Existing_Template_Guide.md** (Legacy - Not needed for chain-copy)
   - Reference for template-based approach
   - Not applicable for chain-copy workflow

## 🚀 Quick Start

### Prerequisites
- Microsoft 365 account with SharePoint access
- Power Automate Premium license (or included in M365)
- Edit permissions for SharePoint site
- **One initial file to start the chain** (e.g., US_CAN-Shift-01-May-2026.xlsx)

### Implementation Steps

1. **Read the Primary Guide**
   - Start with [`SharePoint_Chain_Copy_Workflow_Guide.md`](SharePoint_Chain_Copy_Workflow_Guide.md) ⭐
   - Understand the chain-copy workflow logic

2. **Prepare Your SharePoint Environment**
   - Create folder structure in SharePoint
   - Upload one initial file (e.g., US_CAN-Shift-01-May-2026.xlsx)
   - Verify you have edit permissions

3. **Create the Flows**
   - Follow the step-by-step guide in [`SharePoint_Chain_Copy_Workflow_Guide.md`](SharePoint_Chain_Copy_Workflow_Guide.md)
   - Create all 5 flows in order:
     1. Monthly Folder Creator
     2. Daily Folder Creator
     3. India Shift 1 Generator (copies from yesterday)
     4. India Shift 2 Generator (copies from today's Shift 1)
     5. US-CAN Shift Generator (copies from today's Shift 2)

4. **Test Each Flow**
   - Test manually before enabling automation
   - Verify file copying and renaming works correctly
   - Check date calculations for month transitions

5. **Enable and Monitor**
   - Enable all flows for automated runs
   - Monitor for 24-48 hours
   - Check flow run history daily

## 🏗️ System Architecture

### Five Power Automate Flows

1. **Monthly Folder Creator**
   - Runs: 1st of each month at 6:00 AM IST
   - Creates: Month folder (e.g., "May")

2. **Daily Folder Creator**
   - Runs: Every day at 6:15 AM IST
   - Creates: Daily folder (e.g., "02_May_2026")

3. **India Shift 1 File Generator**
   - Runs: Every day at 6:30 AM IST
   - Copies: Yesterday's US-CAN file → Renames to India_Shift_1_[today].xlsx

4. **India Shift 2 File Generator**
   - Runs: Every day at 2:30 PM IST
   - Copies: Today's India Shift 1 file → Renames to India_Shift_2_[today].xlsx

5. **US-Canada Shift File Generator**
   - Runs: Every day at 10:30 PM IST
   - Copies: Today's India Shift 2 file → Renames to US_CAN-Shift-[today].xlsx

### Why This Approach?

✅ **Data Continuity**: Each file carries over data from previous shift
✅ **No Template Needed**: Uses actual working files as source
✅ **Reliability**: Each flow handles one specific task
✅ **Maintainability**: Easy to troubleshoot individual components
✅ **Flexibility**: Can modify or disable individual flows
✅ **No Coding Required**: Uses Power Automate's visual designer
✅ **SharePoint Integration**: Works with existing SharePoint structure

## 🔧 Key Features

- **Automatic Folder Creation**: Monthly and daily folders created automatically
- **Chain-Copy Workflow**: Each file created by copying previous shift's file
- **Data Continuity**: Maintains data flow across shifts
- **Duplicate Prevention**: Replaces existing files if needed
- **Error Handling**: Robust error handling with notifications
- **Time Zone Support**: Properly handles India Standard Time (IST)
- **Month Transition**: Handles copying from previous month automatically
- **SharePoint Native**: Uses SharePoint connector for reliability

## 📊 File Naming Convention

**Monthly Folder**: `{MonthName}`
- Example: `May`, `June`, `July`

**Daily Folder**: `{DD}_{MonthName}_{YYYY}`
- Example: `01_May_2026`, `15_June_2026`

**India Shift Files**: `India_Shift_{Number}_{DD-MMM-YYYY}.xlsx`
- Example: `India_Shift_1_01-May-2026.xlsx`

**US-Canada Shift File**: `US_CAN-Shift-{DD-MMM-YYYY}.xlsx`
- Example: `US_CAN-Shift-01-May-2026.xlsx`

## 🛠️ Technology Stack

- **Cloud Platform**: Microsoft 365
- **Storage**: SharePoint Online
- **Automation**: Power Automate (Cloud Flows)
- **File Format**: Microsoft Excel (.xlsx)
- **Time Zone**: India Standard Time (UTC+5:30)
- **Workflow**: Chain-copy (each file copies from previous)

## 📈 Expected Results

### Daily
- 3 Excel files created automatically via chain-copy
- Each file contains data from previous shift
- Files available at scheduled times
- Proper folder organization maintained

### Monthly
- ~90 files created (30 days × 3 shifts)
- New month folder created automatically inside year folder
- Consistent naming and structure

### Yearly
- ~1,095 files created (365 days × 3 shifts)
- 12 month folders organized inside year folder
- New year folder created automatically on January 1st
- Minimal manual intervention required

## 🔍 Monitoring

### Daily Checks
- Review flow run history in Power Automate
- Verify files created on time
- Check for any failures

### Weekly Checks
- Audit all flow runs
- Verify file consistency
- Review storage usage

### Monthly Checks
- Audit folder structure
- Update HO template if needed
- Optimize flow performance

## 🆘 Troubleshooting

Common issues and solutions are documented in:
- [`SharePoint_Chain_Copy_Workflow_Guide.md`](SharePoint_Chain_Copy_Workflow_Guide.md) - Comprehensive troubleshooting
- [`Quick_Reference_Guide.md`](Quick_Reference_Guide.md) - Quick fixes

### Quick Fixes

**Flow not running?**
- Check if flow is enabled
- Verify time zone setting
- Review trigger configuration

**Wrong date/time?**
- Use `convertFromUtc()` function
- Set time zone to 'India Standard Time'

**File not created?**
- Verify source file exists (previous shift's file)
- Check folder paths in SharePoint
- Review flow run history
- Ensure previous flow completed successfully

**Wrong file copied?**
- Check date calculations
- Verify time zone settings
- Test with Compose actions to debug dates

## 📞 Support

### Resources
- [Power Automate Documentation](https://docs.microsoft.com/power-automate/)
- [OneDrive Connector Reference](https://docs.microsoft.com/connectors/onedrive/)
- [Power Automate Community](https://powerusers.microsoft.com/t5/Power-Automate-Community/ct-p/MPACommunity)

### Getting Help
1. Check the documentation files in this repository
2. Review flow run history for error details
3. Consult Power Automate community forums
4. Contact Microsoft support if needed

## 🎓 Learning Path

### For Beginners
1. Start with [`SharePoint_Chain_Copy_Workflow_Guide.md`](SharePoint_Chain_Copy_Workflow_Guide.md) ⭐
2. Understand the chain-copy concept
3. Follow the step-by-step flow creation guide
4. Test each flow individually
5. Monitor automated runs

### For Experienced Users
1. Review [`SharePoint_Chain_Copy_Workflow_Guide.md`](SharePoint_Chain_Copy_Workflow_Guide.md)
2. Understand the date calculation logic
3. Customize for month transitions
4. Add error handling enhancements
5. Implement monitoring and alerts

## 🔐 Security Considerations

- Use organizational account for automation
- Enable MFA for automation account
- Apply principle of least privilege
- Regular permission audits
- Enable audit logging in SharePoint
- Backup important files regularly
- Monitor flow run history

## 💰 Cost Considerations

### Power Automate Licensing
- Microsoft 365 license includes Power Automate (limited runs)
- Power Automate Premium for unlimited runs
- Estimated: ~1,472 runs per year

### SharePoint Storage
- Each file: Size depends on your actual working files
- Daily files: 3 files per day
- Monthly: ~90 files (30 days × 3 shifts)
- Yearly: ~1,095 files
- Implement retention policy to manage storage

## 🔄 Maintenance Schedule

### Daily
- Monitor flow run history
- Check for failures

### Weekly
- Review all flow runs
- Verify file consistency

### Monthly
- Audit folder structure
- Review file sizes and storage
- Check for any anomalies in copied files

### Quarterly
- Comprehensive system review
- Update documentation
- Plan improvements

## 📝 Version History

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | May 2026 | Initial documentation - template-based approach |
| 1.1 | May 2026 | Added guide for using existing HO template |
| 2.0 | May 6, 2026 | Major update - SharePoint chain-copy workflow |
| 2.1 | May 6, 2026 | Added Year Folder Creator (Flow 0) at 5:30 AM |

## 🤝 Contributing

This is an internal documentation project. For improvements or suggestions:
1. Document the proposed change
2. Test thoroughly in a development environment
3. Update relevant documentation
4. Submit for review

## 📄 License

Internal use only. All rights reserved.

## ✅ Implementation Status

Use [`Implementation_Checklist.md`](Implementation_Checklist.md) to track your implementation progress.

## 🎯 Success Metrics

- ✅ 100% automated file creation
- ✅ Zero manual intervention required
- ✅ Files available on time
- ✅ Consistent naming and structure
- ✅ Data continuity across shifts maintained
- ✅ No duplicate files
- ✅ Reliable 24/7 operation

## 📧 Contact

For questions or support regarding this automation system, contact your IT team or Power Automate administrator.

---

**Last Updated**: May 2, 2026  
**Maintained By**: IT Team  
**Review Frequency**: Quarterly

---

## 🚦 Getting Started Now

Ready to implement? Follow these steps:

1. ✅ Read [`SharePoint_Chain_Copy_Workflow_Guide.md`](SharePoint_Chain_Copy_Workflow_Guide.md) ⭐
2. ✅ Understand the chain-copy workflow concept
3. ✅ Prepare your SharePoint environment
4. ✅ Create all 5 flows following the guide
5. ✅ Test each flow individually
6. ✅ Enable automated runs and monitor

**Good luck with your implementation!** 🎉

---

## 💡 Important Note About Chain-Copy Workflow

This system uses a **chain-copy approach** where:
- Each shift file is created by copying the previous shift's file
- Data flows continuously from one shift to the next
- No static template is needed
- The first file of each day copies from yesterday's last file
- Month transitions are handled automatically

**Key Benefit**: Maintains data continuity across shifts and days!