# E2E Test: Export Functionality

## Test Overview
This test validates the one-click export functionality for both tables and query results.

## Prerequisites
- Application is running (server and client)
- Browser automation tools available
- Sample CSV file for testing

## Test Steps

### 1. Setup and Upload Data
1. Navigate to http://localhost:5173
2. Take screenshot: `initial_state.png`
3. Upload sample CSV file (e.g., sales_data.csv)
4. Verify table appears in Available Tables
5. Take screenshot: `table_uploaded.png`

### 2. Test Table Export
1. Locate the download button to the left of the X button for the uploaded table
2. Click the download button
3. Verify CSV file downloads with the table name (e.g., sales_data.csv)
4. Open downloaded file and verify:
   - Headers match column names
   - Data matches original upload
   - Special characters are properly escaped
5. Take screenshot: `table_export_button.png`

### 3. Run Query and Test Query Export
1. Enter a natural language query: "Show all sales from last month"
2. Click Submit
3. Wait for results to display
4. Take screenshot: `query_results.png`
5. Locate the download button to the left of the Hide button
6. Click the download button
7. Verify CSV file downloads with timestamp in name (e.g., query_results_20240115_143022.csv)
8. Open downloaded file and verify:
   - Headers match displayed columns
   - Data matches displayed results
   - Row count matches displayed count
9. Take screenshot: `query_export_button.png`

### 4. Edge Cases Testing
1. **Empty Results:**
   - Run query that returns no results
   - Click export button
   - Verify CSV downloads with headers only

2. **Special Characters:**
   - Upload data with special characters (commas, quotes, newlines)
   - Export table and verify proper escaping in CSV

3. **Large Dataset:**
   - If available, upload larger dataset (1000+ rows)
   - Export and verify all data is included

### 5. Error Handling
1. Delete the table
2. Attempt to navigate directly to export URL
3. Verify appropriate error response

## Success Criteria
- ✅ Download buttons appear in correct positions
- ✅ Table export downloads complete table data
- ✅ Query export downloads displayed results only
- ✅ CSV format is valid with proper headers
- ✅ Special characters are properly escaped
- ✅ File names are descriptive and include timestamps where appropriate
- ✅ No console errors during export process
- ✅ Downloads work in Chrome, Firefox, and Safari

## Screenshots to Capture
1. `initial_state.png` - Application before data upload
2. `table_uploaded.png` - Available Tables with data
3. `table_export_button.png` - Table with export button visible
4. `query_results.png` - Query results displayed
5. `query_export_button.png` - Query results with export button visible

## Test Data Requirements
Prepare a CSV file with:
- At least 10 rows of data
- Mix of data types (text, numbers, dates)
- Some special characters for edge case testing
- Column names that are SQL-friendly

## Validation Commands
After manual testing, run:
```bash
cd app/server && uv run pytest tests/test_export.py -v
cd app/client && bun test export.test.ts
```

## Notes
- Ensure downloads folder is accessible for verification
- Clear downloads before testing for clean validation
- Test in incognito/private mode to avoid caching issues