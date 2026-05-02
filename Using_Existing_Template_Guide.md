# Using Your Existing HO Template File

## Overview

Since you already have an existing HO (Head Office) template file, this guide explains how to integrate it into the automated shift file generation system.

## Steps to Use Your Existing Template

### Step 1: Locate Your HO Template File

1. Find your existing HO template file on your computer or current location
2. Note the file name and location
3. Ensure the file is in Excel format (.xlsx or .xls)

### Step 2: Prepare the Template for Automation

Before uploading to OneDrive, verify your template has:

✅ **All Required Content**:
- Headers and column names
- Formulas and calculations
- Data validation rules
- Conditional formatting
- Any required sheets/tabs
- Company branding/logos

✅ **No Dynamic Date References**:
- Remove any "TODAY()" functions that should be static
- Keep formulas that should calculate based on data entry
- Ensure the template works as a standalone file

✅ **Proper Formatting**:
- Cell formatting (number formats, dates, currency)
- Row heights and column widths
- Print settings (if needed)
- Page layout and margins

### Step 3: Upload Template to OneDrive

1. **Access OneDrive for Business**
   - Go to your OneDrive for Business account
   - Navigate to the root directory

2. **Create Folder Structure**
   ```
   /Shift_Files/
     └── Templates/
   ```
   
   Steps:
   - Create a new folder named `Shift_Files`
   - Inside `Shift_Files`, create a subfolder named `Templates`

3. **Upload Your HO Template**
   - Navigate to `/Shift_Files/Templates/`
   - Click **Upload** → **Files**
   - Select your HO template file
   - Rename it to: `Shift_Template.xlsx` (for consistency with the automation)
   - Or keep your original name and update the flow configurations

### Step 4: Verify Template Upload

1. Open the uploaded file in OneDrive
2. Verify all formatting is preserved
3. Check that formulas work correctly
4. Test any data validation rules
5. Ensure all sheets/tabs are present

### Step 5: Update Flow Configurations (If Needed)

If you kept your original template name instead of renaming to `Shift_Template.xlsx`:

**In Each Shift Flow (Flows 3, 4, and 5)**:

Find this action:
```
Action: Get file content (OneDrive for Business)
File: /Shift_Files/Templates/Shift_Template.xlsx
```

Update to your template name:
```
Action: Get file content (OneDrive for Business)
File: /Shift_Files/Templates/YOUR_TEMPLATE_NAME.xlsx
```

Replace `YOUR_TEMPLATE_NAME.xlsx` with your actual file name.

## Template File Path Reference

### Standard Path (Recommended)
```
/Shift_Files/Templates/Shift_Template.xlsx
```

### If Using Original Name
```
/Shift_Files/Templates/[Your_Original_HO_Template_Name].xlsx
```

## Important Considerations

### 1. Template Backup
- **Always keep a backup** of your original HO template
- Store backup in a separate location
- Version control if template changes frequently

### 2. Template Updates
When you need to update the template:
1. Make changes to a copy first
2. Test the updated template
3. Upload to OneDrive (overwrite existing)
4. Test one shift flow to verify
5. Monitor for issues

### 3. File Size
- Keep template file size reasonable (<5 MB recommended)
- Large files may slow down the automation
- Remove unnecessary data or images if file is too large

### 4. Compatibility
- Use Excel format (.xlsx) for best compatibility
- Avoid Excel features that don't work in OneDrive:
  - ActiveX controls
  - VBA macros (unless OneDrive supports them)
  - External data connections

### 5. Formulas
- Test that all formulas work when file is copied
- Avoid formulas that reference external files
- Use relative references where appropriate

## Testing Your Template

### Test 1: Manual Copy Test
1. Manually copy your template in OneDrive
2. Rename the copy with a shift file name format
3. Open and verify everything works
4. Check formulas, formatting, and data validation

### Test 2: Flow Test
1. Create one shift flow (e.g., Shift 1)
2. Run it manually
3. Open the generated file
4. Verify template content is preserved
5. Test all functionality

### Test 3: Full Day Test
1. Run all three shift flows
2. Verify all three files are created correctly
3. Open each file and test
4. Confirm no issues with multiple copies

## Common Issues and Solutions

### Issue 1: Formulas Not Working
**Problem**: Formulas show errors in copied files

**Solutions**:
- Check for external references
- Use named ranges instead of cell references
- Test formulas in a manual copy first
- Update formulas to be self-contained

### Issue 2: Formatting Lost
**Problem**: Some formatting doesn't appear in copied files

**Solutions**:
- Avoid complex conditional formatting
- Use standard Excel formatting
- Test with simpler formatting first
- Check OneDrive compatibility

### Issue 3: File Size Too Large
**Problem**: Template file is very large, slowing automation

**Solutions**:
- Remove unused sheets
- Compress images
- Remove unnecessary formatting
- Clear unused cells

### Issue 4: Data Validation Not Working
**Problem**: Dropdown lists or validation rules don't work

**Solutions**:
- Use in-cell dropdowns (not external lists)
- Define validation rules within the file
- Test validation in copied file
- Simplify validation rules if needed

## Template Maintenance

### Monthly Review
- [ ] Check if template needs updates
- [ ] Review feedback from users
- [ ] Test any proposed changes
- [ ] Update template in OneDrive if needed

### Version Control
Keep track of template versions:

| Version | Date | Changes | Updated By |
|---------|------|---------|------------|
| 1.0 | May 2026 | Initial template | [Name] |
| 1.1 | [Date] | [Changes] | [Name] |

### Change Process
1. Document proposed changes
2. Create test version
3. Test thoroughly
4. Get approval
5. Update production template
6. Monitor for issues
7. Document changes

## Template Location Quick Reference

**OneDrive Path**:
```
/Shift_Files/Templates/Shift_Template.xlsx
```

**Full URL** (example):
```
https://[your-company].sharepoint.com/personal/[your-name]/Documents/Shift_Files/Templates/Shift_Template.xlsx
```

**Power Automate Reference**:
```
File Path: /Shift_Files/Templates/Shift_Template.xlsx
```

## Integration with Automation Flows

Your HO template will be used by:

1. **Shift 1 Flow** (6:00 AM IST)
   - Copies template
   - Names: `May_India_Shift_1_01-May-2026.xlsx`

2. **Shift 2 Flow** (2:00 PM IST)
   - Copies template
   - Names: `May_India_Shift_2_01-May-2026.xlsx`

3. **Shift 3 Flow** (10:00 PM IST)
   - Copies template
   - Names: `May_India_Shift_3_01-May-2026.xlsx`

Each flow:
- Reads your template file
- Creates a copy in the daily folder
- Renames with shift-specific name
- Preserves all formatting and formulas

## Next Steps

Now that you have your HO template ready:

1. ✅ Upload template to OneDrive at `/Shift_Files/Templates/`
2. ✅ Verify template works correctly in OneDrive
3. ✅ Proceed with creating the Power Automate flows
4. ✅ Test with your actual template
5. ✅ Deploy to production

## Questions to Consider

Before proceeding, confirm:

- [ ] Is the template file in Excel format (.xlsx)?
- [ ] Does it contain all necessary sheets and data?
- [ ] Are all formulas working correctly?
- [ ] Is the file size reasonable (<5 MB)?
- [ ] Have you created a backup of the original?
- [ ] Do you want to rename it to `Shift_Template.xlsx` or keep the original name?

## Support

If you encounter issues with your template:
1. Check the Common Issues section above
2. Test manual copy in OneDrive first
3. Simplify template if needed
4. Contact IT support for assistance

---

**Remember**: Your existing HO template is the foundation of this automation. Taking time to properly prepare and test it will ensure smooth operation of the entire system.