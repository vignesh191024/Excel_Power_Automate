# OneDrive Shift Files Automation - Complete Documentation

## 📋 Overview

This project provides a complete solution for automating the creation of shift-based Excel files in OneDrive using Power Automate. The system automatically generates three shift files daily, organized in a hierarchical folder structure by month and date.

## 🎯 What This System Does

**Automated File Generation**:
- Creates monthly folders (e.g., "May", "June")
- Creates daily folders with format `DD_Month_YYYY` (e.g., "01_May_2026")
- Generates 3 shift files per day at scheduled times:
  - **Shift 1 (India)**: 6:30 AM IST
  - **Shift 2 (India)**: 2:30 PM IST
  - **Shift 3 (US-Canada)**: 10:30 PM IST

**Example Output Structure**:
```
OneDrive/Shift_Files/
└── May/
    └── 01_May_2026/
        ├── May_India_Shift_1_01-May-2026.xlsx
        ├── May_India_Shift_2_01-May-2026.xlsx
        └── US-CAN-SHIFT_01-May-2026.xlsx
```

## 📚 Documentation Files

This repository contains comprehensive documentation for implementing and maintaining the automation system:

### 1. **OneDrive_Shift_Files_Automation_Plan.md**
   - Complete system architecture and design
   - Detailed explanation of all 5 Power Automate flows
   - Power Automate expressions and formulas
   - Troubleshooting guide
   - Alternative approaches and recommendations
   - Cost considerations
   - Security and permissions

### 2. **Power_Automate_Flow_Configurations.md**
   - Step-by-step configuration for each flow
   - Detailed action-by-action instructions
   - Testing procedures
   - Error handling setup
   - Quick reference for expressions

### 3. **Quick_Reference_Guide.md**
   - Visual diagrams and flowcharts
   - Daily timeline
   - Naming conventions
   - Common tasks
   - Troubleshooting quick fixes
   - Monitoring dashboard
   - Maintenance checklist

### 4. **Implementation_Checklist.md**
   - Phase-by-phase implementation guide
   - Checkboxes for tracking progress
   - Testing procedures
   - Deployment steps
   - Training and handover
   - Sign-off section

### 5. **Using_Existing_Template_Guide.md** ⭐ NEW
   - How to use your existing HO template file
   - Template preparation steps
   - Upload instructions
   - Testing procedures
   - Troubleshooting template issues
   - Template maintenance guide

## 🚀 Quick Start

### Prerequisites
- Microsoft 365 account with OneDrive for Business
- Power Automate Premium license (or included in M365)
- Appropriate permissions for OneDrive and Power Automate
- **Your existing HO template file** ✅

### Implementation Steps

1. **Read the Documentation**
   - Start with [`OneDrive_Shift_Files_Automation_Plan.md`](OneDrive_Shift_Files_Automation_Plan.md)
   - Review the architecture and approach

2. **Prepare Your HO Template**
   - Follow [`Using_Existing_Template_Guide.md`](Using_Existing_Template_Guide.md)
   - Upload your HO template to OneDrive
   - Verify template works correctly

3. **Prepare Your Environment**
   - Create folder structure in OneDrive: `/Shift_Files/Templates/`
   - Upload your HO template file
   - Verify template is accessible

4. **Create the Flows**
   - Follow [`Power_Automate_Flow_Configurations.md`](Power_Automate_Flow_Configurations.md)
   - Create all 5 flows step-by-step
   - Test each flow individually

5. **Test and Deploy**
   - Use [`Implementation_Checklist.md`](Implementation_Checklist.md)
   - Complete all testing phases
   - Enable flows for production

6. **Monitor and Maintain**
   - Use [`Quick_Reference_Guide.md`](Quick_Reference_Guide.md)
   - Follow maintenance schedule
   - Address issues promptly

## 🏗️ System Architecture

### Five Power Automate Flows

1. **Monthly Folder Creator**
   - Runs: 1st of each month at 6:00 AM IST
   - Creates: Month folder (e.g., "May")

2. **Daily Folder Creator**
   - Runs: Every day at 6:15 AM IST
   - Creates: Daily folder (e.g., "01_May_2026")

3. **Shift 1 File Generator (India)**
   - Runs: Every day at 6:30 AM IST
   - Creates: India Shift 1 Excel file from your HO template

4. **Shift 2 File Generator (India)**
   - Runs: Every day at 2:30 PM IST
   - Creates: India Shift 2 Excel file from your HO template

5. **Shift 3 File Generator (US-Canada)**
   - Runs: Every day at 10:30 PM IST
   - Creates: US-Canada Shift Excel file from your HO template

### Why This Approach?

✅ **Reliability**: Each flow handles one specific task  
✅ **Maintainability**: Easy to troubleshoot individual components  
✅ **Flexibility**: Can modify or disable individual flows  
✅ **Scalability**: Easy to add more shifts or modify schedules  
✅ **No Coding Required**: Uses Power Automate's visual designer  
✅ **Uses Your Existing Template**: Preserves your HO file formatting and formulas

## 🔧 Key Features

- **Automatic Folder Creation**: Monthly and daily folders created automatically
- **Template-Based Files**: All files copied from your existing HO template
- **Duplicate Prevention**: Checks if files exist before creating
- **Error Handling**: Robust error handling with notifications
- **Time Zone Support**: Properly handles India Standard Time (IST)
- **Email Notifications**: Optional success/failure notifications
- **Formatting Preservation**: Your HO template formatting maintained in all files

## 📊 File Naming Convention

**Monthly Folder**: `{MonthName}`
- Example: `May`, `June`, `July`

**Daily Folder**: `{DD}_{MonthName}_{YYYY}`
- Example: `01_May_2026`, `15_June_2026`

**Shift Files**: `{Month}_India_Shift_{Number}_{DD-MMM-YYYY}.xlsx`
- Example: `May_India_Shift_1_01-May-2026.xlsx`

## 🛠️ Technology Stack

- **Cloud Platform**: Microsoft 365
- **Storage**: OneDrive for Business
- **Automation**: Power Automate (Cloud Flows)
- **File Format**: Microsoft Excel (.xlsx)
- **Time Zone**: India Standard Time (UTC+5:30)
- **Template**: Your existing HO template file

## 📈 Expected Results

### Daily
- 3 Excel files created automatically from your HO template
- Files available at scheduled times
- Proper folder organization maintained

### Monthly
- ~90 files created (30 days × 3 shifts)
- New month folder created automatically
- Consistent naming and structure

### Yearly
- ~1,095 files created (365 days × 3 shifts)
- 12 month folders organized
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
- [`OneDrive_Shift_Files_Automation_Plan.md`](OneDrive_Shift_Files_Automation_Plan.md) - Detailed troubleshooting section
- [`Quick_Reference_Guide.md`](Quick_Reference_Guide.md) - Quick fixes
- [`Using_Existing_Template_Guide.md`](Using_Existing_Template_Guide.md) - Template-specific issues

### Quick Fixes

**Flow not running?**
- Check if flow is enabled
- Verify time zone setting
- Review trigger configuration

**Wrong date/time?**
- Use `convertFromUtc()` function
- Set time zone to 'India Standard Time'

**File not created?**
- Verify HO template file exists at `/Shift_Files/Templates/`
- Check folder path
- Review flow run history

**Template formatting lost?**
- See [`Using_Existing_Template_Guide.md`](Using_Existing_Template_Guide.md)
- Test manual copy first
- Check template compatibility

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
1. Start with [`Quick_Reference_Guide.md`](Quick_Reference_Guide.md)
2. Read [`Using_Existing_Template_Guide.md`](Using_Existing_Template_Guide.md)
3. Understand the folder structure and naming conventions
4. Learn basic Power Automate concepts
5. Follow [`Power_Automate_Flow_Configurations.md`](Power_Automate_Flow_Configurations.md) step-by-step

### For Experienced Users
1. Review [`OneDrive_Shift_Files_Automation_Plan.md`](OneDrive_Shift_Files_Automation_Plan.md)
2. Understand the architecture decisions
3. Prepare your HO template using the guide
4. Customize flows for your specific needs
5. Implement advanced features

## 🔐 Security Considerations

- Use organizational account for automation
- Enable MFA for automation account
- Apply principle of least privilege
- Regular permission audits
- Enable audit logging
- Backup HO template file regularly
- Version control for template changes

## 💰 Cost Considerations

### Power Automate Licensing
- Microsoft 365 license includes Power Automate (limited runs)
- Power Automate Premium for unlimited runs
- Estimated: ~1,472 runs per year

### OneDrive Storage
- Each file: ~50 KB (depends on your HO template size)
- Daily files: 150 KB (3 shifts)
- Monthly: ~4.5 MB (30 days)
- Yearly: ~54 MB
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
- Update HO template if needed
- Review performance

### Quarterly
- Comprehensive system review
- Update documentation
- Plan improvements

## 📝 Version History

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | May 2026 | Initial documentation and implementation plan |
| 1.1 | May 2026 | Added guide for using existing HO template |

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
- ✅ HO template formatting preserved
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

1. ✅ Read [`OneDrive_Shift_Files_Automation_Plan.md`](OneDrive_Shift_Files_Automation_Plan.md)
2. ✅ Prepare your HO template using [`Using_Existing_Template_Guide.md`](Using_Existing_Template_Guide.md)
3. ✅ Open [`Implementation_Checklist.md`](Implementation_Checklist.md)
4. ✅ Start with Phase 1: Prerequisites and Setup
5. ✅ Follow [`Power_Automate_Flow_Configurations.md`](Power_Automate_Flow_Configurations.md) for each flow
6. ✅ Use [`Quick_Reference_Guide.md`](Quick_Reference_Guide.md) as your daily reference

**Good luck with your implementation!** 🎉

---

## 💡 Important Note About Your HO Template

Since you already have an existing HO template file, make sure to:
- Read [`Using_Existing_Template_Guide.md`](Using_Existing_Template_Guide.md) first
- Upload your template to `/Shift_Files/Templates/` in OneDrive
- Test that the template works correctly when copied
- Keep a backup of your original template file

The automation will use your HO template to create all shift files, preserving all your formatting, formulas, and data structures.