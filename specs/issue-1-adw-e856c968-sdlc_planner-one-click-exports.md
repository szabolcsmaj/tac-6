# Feature: One Click Table Exports

## Metadata
issue_number: 1
adw_id: e856c968
issue_json: {"number":1,"title":"One click table exports","body":"Using adw_plan_build_review add one click table exports and one click result export feature to get results as csv files.\n\nCreate 2 new endpoints to support these features. One for exporting tables, one for exporting query results.\n\nPlace a download button directly to the left of the 'x' icon for available tables.\nPlace a download button directly to the left of the 'hide' button for query results.\n\nUse the appropriate download icon."}

## Feature Description
This feature adds the ability to export database tables and query results as CSV files with a single click. Users will have download buttons conveniently placed in the UI: one for each table in the Available Tables section (to the left of the remove button) and one for query results (to the left of the hide button). This enables users to easily extract and share data from the application without needing to manually copy or reformat results.

## User Story
As a data analyst
I want to export tables and query results as CSV files with one click
So that I can easily share data, perform further analysis in spreadsheet applications, or archive query results

## Problem Statement
Currently, users cannot easily export their data from the application. They must manually copy and paste results, which is error-prone and time-consuming, especially for large datasets. Users need a quick way to download both entire tables and specific query results in a standard format (CSV) that can be opened in Excel, Google Sheets, or other data analysis tools.

## Solution Statement
Implement two new API endpoints that generate CSV exports: one for complete table data and another for query results. Add download buttons with clear icons to the UI that trigger immediate CSV downloads when clicked. The solution will handle proper CSV formatting, including escaping special characters, handling null values, and setting appropriate HTTP headers for file downloads.

## Relevant Files
Use these files to implement the feature:

- `app/server/server.py` - Add new export endpoints for table and query result exports
- `app/server/core/data_models.py` - Add request model for query export endpoint
- `app/server/core/sql_processor.py` - Reuse existing SQL execution logic for exports
- `app/server/core/sql_security.py` - Use for validating table names and SQL queries
- `app/client/src/main.ts` - Add download buttons and click handlers
- `app/client/src/api/client.ts` - Add API methods for export endpoints
- `app/client/src/style.css` - Add styles for download buttons
- `app/client/src/types.d.ts` - Add TypeScript interface for export request
- `.claude/commands/test_e2e.md` - Reference for E2E test structure
- `.claude/commands/e2e/test_basic_query.md` - Reference for E2E test example

### New Files
- `.claude/commands/e2e/test_export_functionality.md` - E2E test file for validating export functionality

## Implementation Plan
### Phase 1: Foundation
Set up the backend infrastructure for CSV generation and export functionality, including data models and utility functions.

### Phase 2: Core Implementation
Implement the export endpoints in the backend and add the download buttons with full functionality in the frontend.

### Phase 3: Integration
Create comprehensive E2E tests and ensure all components work together seamlessly with proper error handling.

## Step by Step Tasks
IMPORTANT: Execute every step in order, top to bottom.

### Task 1: Add Export Request Model
- Add `ExportQueryRequest` model to `app/server/core/data_models.py` with fields for SQL query and columns
- This model will be used by the query export endpoint to reproduce exact query results

### Task 2: Implement Table Export Endpoint
- Add GET endpoint `/api/export/table/{table_name}` in `app/server/server.py`
- Validate table name using `sql_security.validate_identifier()`
- Query all data from the table using `sql_processor.execute_sql_safely()`
- Generate CSV using Python's csv module with proper escaping
- Return Response with CSV content, Content-Type: text/csv, and Content-Disposition header

### Task 3: Implement Query Export Endpoint
- Add POST endpoint `/api/export/query` in `app/server/server.py`
- Accept `ExportQueryRequest` with SQL query and columns
- Validate SQL query using `sql_security.validate_sql_query()`
- Execute query using `sql_processor.execute_sql_safely()`
- Generate CSV from results
- Return Response with CSV content and appropriate headers

### Task 4: Add TypeScript Types
- Add `ExportQueryRequest` interface to `app/client/src/types.d.ts`
- Interface should match the Pydantic model exactly with sql and columns fields

### Task 5: Add API Client Methods
- Add `exportTable(tableName: string)` method to `app/client/src/api/client.ts`
- Add `exportQueryResults(sql: string, columns: string[])` method
- Both methods should handle blob responses and trigger downloads

### Task 6: Create E2E Test File
- Create `.claude/commands/e2e/test_export_functionality.md`
- Include test steps for uploading data, exporting table, running query, exporting results
- Specify screenshot capture points
- Define success criteria for download functionality

### Task 7: Add Download Button Styles
- Add `.download-button` class to `app/client/src/style.css`
- Style should match existing button patterns but be smaller/icon-focused
- Add hover effects and disabled states

### Task 8: Add Table Export Buttons
- Modify `displayTables()` function in `app/client/src/main.ts`
- Add download button element to the left of remove button in table-header
- Use download icon (⬇ or SVG)
- Add click handler that calls `api.exportTable()`
- Implement blob download logic with descriptive filename

### Task 9: Add Query Result Export Button
- Modify `displayResults()` function in `app/client/src/main.ts`
- Add download button to the left of Hide button in results-header
- Store current SQL and columns in closure or data attributes
- Add click handler that calls `api.exportQueryResults()`
- Implement blob download with timestamp in filename

### Task 10: Test Backend Endpoints
- Manually test table export endpoint with curl or browser
- Verify CSV format is correct with proper headers and escaping
- Test query export endpoint with sample SQL
- Verify error handling for invalid table names or SQL

### Task 11: Test Frontend Integration
- Upload sample data
- Verify download button appears for table
- Click download and verify CSV file downloads
- Run a query
- Verify download button appears for results
- Click download and verify CSV file downloads with correct data

### Task 12: Run Validation Commands
- Execute all validation commands to ensure no regressions
- Run the new E2E test to validate export functionality
- Fix any issues that arise

## Testing Strategy
### Unit Tests
- Test CSV generation with various data types (strings, numbers, nulls, special characters)
- Test filename generation with sanitization
- Test error handling for invalid table names
- Test SQL validation for export endpoints
- Test response headers are set correctly

### Edge Cases
- Empty tables or query results (should download empty CSV with headers)
- Tables/columns with special characters in names
- Very large result sets (should handle streaming if needed)
- Null values in data (should be represented as empty strings in CSV)
- Queries with no results (should download CSV with column headers only)
- Invalid table names or SQL injection attempts (should be rejected)
- Concurrent export requests (should handle independently)

## Acceptance Criteria
- Download buttons appear in the correct positions (left of X for tables, left of Hide for results)
- Clicking table download button immediately downloads a CSV file with all table data
- Clicking query result download button immediately downloads a CSV with the displayed results
- CSV files have proper headers matching column names
- CSV files properly escape special characters and handle null values
- Downloaded files have descriptive names (table_name.csv, query_results_timestamp.csv)
- Download functionality works in Chrome, Firefox, and Safari
- No console errors or warnings during download process
- Export endpoints return proper HTTP status codes and headers
- Security validation prevents SQL injection and unauthorized access

## Validation Commands
Execute every command to validate the feature works correctly with zero regressions.

- Read `.claude/commands/test_e2e.md`, then read and execute the new E2E `.claude/commands/e2e/test_export_functionality.md` test file to validate export functionality works.
- `cd app/server && uv run pytest` - Run server tests to validate the feature works with zero regressions
- `cd app/client && bun tsc --noEmit` - Run frontend tests to validate the feature works with zero regressions
- `cd app/client && bun run build` - Run frontend build to validate the feature works with zero regressions

## Notes
- Consider implementing streaming for very large datasets in a future enhancement to avoid memory issues
- The CSV format was chosen as it's universally supported and easily opened in spreadsheet applications
- Download icons should be intuitive but not too prominent to avoid cluttering the UI
- File naming convention includes timestamps to prevent confusion when downloading multiple exports
- Future enhancements could include other export formats (JSON, Excel) or filtered exports