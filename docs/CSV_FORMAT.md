# CSV Import Format Guide

This document describes the required CSV format for importing household beneficiary data into the system.

## Required CSV Header

The CSV file must contain the following header row as the first line:

```csv
State,LGA,Ward,Community,nidhh,HouseHoldNo,HAddress,TrancheStatus,TotalAmount,FirstTrancheRecipient,FirstTrancheAccountNumber,FirstTrancheBankName,FirstTranchePaymentDate,FirstTranchePhone,FirstTrancheGender,FirstTrancheAge,FirstTrancheIDType,SecondTrancheRecipient,SecondTrancheAccountNumber,SecondTrancheBankName,SecondTranchePaymentDate,SecondTranchePhone,SecondTrancheGender,SecondTrancheAge,SecondTrancheIDType,ThirdTrancheRecipient,ThirdTrancheAccountNumber,ThirdTrancheBankName,ThirdTranchePaymentDate,ThirdTranchePhone,ThirdTrancheGender,ThirdTrancheAge,ThirdTrancheIDType
```

## Field Descriptions

### Household Information
- **State**: State name (required, string, max 100 chars)
- **LGA**: Local Government Area (required, string, max 100 chars)
- **Ward**: Ward name (required, string, max 100 chars)
- **Community**: Community name (required, string, max 100 chars)
- **nidhh**: National ID Household Number (required, unique, string, max 50 chars)
- **HouseHoldNo**: Household number (optional, string, max 50 chars)
- **HAddress**: Household address (optional, string, max 500 chars)
- **TrancheStatus**: Current tranche status (required, enum: NotStarted, Partial, Completed)
- **TotalAmount**: Total amount for all tranches (required, decimal >= 0)

### First Tranche Information
- **FirstTrancheRecipient**: Recipient name (optional, string, max 200 chars)
- **FirstTrancheAccountNumber**: Bank account number (optional, numeric string)
- **FirstTrancheBankName**: Bank name (optional, string, max 100 chars)
- **FirstTranchePaymentDate**: Payment date (optional, ISO 8601 format: YYYY-MM-DD)
- **FirstTranchePhone**: Phone number (optional, E.164 format preferred)
- **FirstTrancheGender**: Gender (optional, enum: Male, Female, Other, Unknown)
- **FirstTrancheAge**: Age (optional, integer 0-120)
- **FirstTrancheIDType**: ID document type (optional, string, max 50 chars)

### Second Tranche Information
- **SecondTrancheRecipient**: Recipient name (optional, string, max 200 chars)
- **SecondTrancheAccountNumber**: Bank account number (optional, numeric string)
- **SecondTrancheBankName**: Bank name (optional, string, max 100 chars)
- **SecondTranchePaymentDate**: Payment date (optional, ISO 8601 format: YYYY-MM-DD)
- **SecondTranchePhone**: Phone number (optional, E.164 format preferred)
- **SecondTrancheGender**: Gender (optional, enum: Male, Female, Other, Unknown)
- **SecondTrancheAge**: Age (optional, integer 0-120)
- **SecondTrancheIDType**: ID document type (optional, string, max 50 chars)

### Third Tranche Information
- **ThirdTrancheRecipient**: Recipient name (optional, string, max 200 chars)
- **ThirdTrancheAccountNumber**: Bank account number (optional, numeric string)
- **ThirdTrancheBankName**: Bank name (optional, string, max 100 chars)
- **ThirdTranchePaymentDate**: Payment date (optional, ISO 8601 format: YYYY-MM-DD)
- **ThirdTranchePhone**: Phone number (optional, E.164 format preferred)
- **ThirdTrancheGender**: Gender (optional, enum: Male, Female, Other, Unknown)
- **ThirdTrancheAge**: Age (optional, integer 0-120)
- **ThirdTrancheIDType**: ID document type (optional, string, max 50 chars)

## Validation Rules

### Required Fields
- State, LGA, Ward, Community, nidhh, TrancheStatus, TotalAmount

### Unique Constraints
- **nidhh** must be unique across all records

### Data Format Requirements

#### Phone Numbers
- Preferred format: E.164 (e.g., +2348012345678)
- Fallback: Numeric string 7-15 digits
- Invalid characters will be rejected

#### Dates
- Format: ISO 8601 (YYYY-MM-DD)
- Example: 2024-01-15
- Empty values are allowed

#### Account Numbers
- Numeric strings only
- Length validation based on bank requirements
- Luhn algorithm validation where applicable

#### Gender Values
- Accepted values: Male, Female, Other, Unknown
- Case-sensitive
- Empty values default to "Unknown"

#### Age Values
- Integer between 0 and 120
- Empty values are allowed

#### Amount Values
- Decimal numbers >= 0
- Up to 2 decimal places
- Example: 150000.00

## Sample CSV Data

```csv
State,LGA,Ward,Community,nidhh,HouseHoldNo,HAddress,TrancheStatus,TotalAmount,FirstTrancheRecipient,FirstTrancheAccountNumber,FirstTrancheBankName,FirstTranchePaymentDate,FirstTranchePhone,FirstTrancheGender,FirstTrancheAge,FirstTrancheIDType,SecondTrancheRecipient,SecondTrancheAccountNumber,SecondTrancheBankName,SecondTranchePaymentDate,SecondTranchePhone,SecondTrancheGender,SecondTrancheAge,SecondTrancheIDType,ThirdTrancheRecipient,ThirdTrancheAccountNumber,ThirdTrancheBankName,ThirdTranchePaymentDate,ThirdTranchePhone,ThirdTrancheGender,ThirdTrancheAge,ThirdTrancheIDType
Lagos,Ikeja,Ward 1,Community A,NID001,HH001,123 Main Street,Partial,450000.00,John Doe,1234567890,First Bank,2024-01-15,+2348012345678,Male,35,National ID,Jane Doe,0987654321,GTBank,2024-02-15,+2348087654321,Female,32,Voter Card,,,,,,,,,
Kano,Nassarawa,Ward 2,Community B,NID002,HH002,456 Second Avenue,NotStarted,300000.00,,,,,,,,,,,,,,,,,,,,,,,
```

## Import Process

1. **File Upload**: Select CSV file (max 10MB)
2. **Header Validation**: System checks for required headers
3. **Data Preview**: Review first 10 rows for accuracy
4. **Validation**: System validates all data according to rules
5. **Duplicate Check**: System checks for existing nidhh values
6. **Import**: Data is processed in background job
7. **Report**: Detailed success/error report generated

## Error Handling

### Common Errors
- **Missing Required Fields**: Row will be rejected
- **Invalid Data Format**: Specific field errors reported
- **Duplicate nidhh**: Existing records will be updated or rejected based on settings
- **Invalid Enum Values**: Must match exact case-sensitive values

### Error Report
- Line-by-line error details
- Field-specific validation messages
- Summary of successful vs failed imports
- Downloadable error report in CSV format

## File Size Limits
- Maximum file size: 10MB
- Maximum rows: 50,000 per import
- For larger datasets, split into multiple files

## Best Practices

1. **Data Preparation**
   - Clean data before import
   - Ensure consistent formatting
   - Validate phone numbers and dates

2. **Testing**
   - Test with small sample first
   - Verify data accuracy in preview
   - Check validation messages

3. **Backup**
   - Export existing data before large imports
   - Keep original CSV files for reference

4. **Performance**
   - Import during off-peak hours
   - Monitor import progress
   - Large files are processed in background

