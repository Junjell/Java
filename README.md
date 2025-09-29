# User Acceptance Testing (UAT) Plan for AS400 Auto Job Monitor

## Introduction
This UAT document outlines test scenarios for the AS400 Auto Job Monitor application. The goal is to verify that the application meets user requirements for monitoring AS400 jobs, fetching logs, displaying results, exporting data, and providing summaries. 

**Scope**: 
- GUI functionality and user interactions.
- Job fetching from AS400.
- Real-time monitoring.
- Data export to Excel.
- End-of-period summaries.

**Prerequisites**:
- Python 3.8+ installed.
- AS400 ODBC driver configured.
- `.env` file with connection details (AS400_DRIVER, AS400_SYSTEM, AS400_USER, AS400_PASSWORD).
- Run `pip install pyodbc python-dotenv pandas` if not done.
- Launch the app: `python main.py`.
- Test on a system with access to the AS400 'AUTOJOB' message queue.
- Use predefined batches for testing to simulate real workflows.

**Test Environment**:
- OS: Windows (as per project setup).
- AS400 connection: Valid credentials for testing.
- Data: Ensure sample jobs exist in the AUTOJOB queue within test time ranges.

**Pass/Fail Criteria**:
- All test cases pass if expected results match without errors.
- Report any bugs, UI issues, or discrepancies to the development team.

## Test Scenarios

### 1. Application Launch and Basic GUI
**Objective**: Verify the application starts correctly and displays the interface.

| Test Case ID | Description | Steps | Expected Result |
|--------------|-------------|-------|-----------------|
| UAT-001 | Launch the application | 1. Run `python main.py` from the project root.<br>2. Observe the window. | - Window titled "AS400 Auto Job Monitor" opens (1000x600 size).<br>- Fields for Job Names (text area), Batch combobox, Start/End Time entries (readonly initially), Interval label.<br>- Treeview table with columns: MESSAGE_TEXT, MESSAGE_TIMESTAMP.<br>- Buttons: "Fetch Jobs", "Export to Excel".<br>- No errors in console. |
| UAT-002 | Batch selection auto-fills | 1. Select a batch (e.g., "Batch 1") from combobox.<br>2. Observe Job Names, Start/End Time, Interval label. | - Job Names populates with comma-separated jobs (e.g., CLR_SRL, INTEG_AT20, ...).<br>- Start Time: 00:00:00, End Time: 04:05:00 (for Batch 1).<br>- Interval label updates to "Interval: 00:00:00 - 04:05:00".<br>- Time fields remain readonly. |

### 2. Job Fetching and Monitoring
**Objective**: Verify fetching, display, and real-time updates for jobs in a time range.

| Test Case ID | Description | Steps | Expected Result |
|--------------|-------------|-------|-----------------|
| UAT-003 | Fetch jobs with valid input | 1. Enter/Select jobs (e.g., via Batch 1).<br>2. Ensure Start/End times are set.<br>3. Click "Fetch Jobs".<br>4. Wait 5-10 seconds; observe table and console. | - Monitoring starts; table populates with records from AUTOJOB queue matching jobs and time range.<br>- Each row: MESSAGE_TEXT (trimmed, e.g., "CLR_SRL completed normally"), MESSAGE_TIMESTAMP (e.g., '2023-07-25 11:10:00').<br>- Table updates every 5 seconds with new data.<br>- No duplicate rows; ordered by timestamp DESC.<br>- If no data, table remains empty (no crash). |
| UAT-004 | Continuous monitoring until end time | 1. Set a short end time (e.g., current time + 1 min; manually edit if needed by making fields editable temporarily).<br>2. Click "Fetch Jobs".<br>3. Wait until end time passes. | - Table updates every 5s until end time.<br>- At end time, monitoring stops; summary popup appears (see Scenario 3).<br>- No further updates after stop. |
| UAT-005 | Manual job input | 1. Clear batch; manually enter 2-3 job names (e.g., "CLR_SRL, INTEG_AT20").<br>2. Set custom times (e.g., 00:00:00 - 01:00:00).<br>3. Click "Fetch Jobs". | - Fetches only specified jobs in time range.<br>- Table shows matching records; ignores unrelated jobs. |
| UAT-006 | Invalid input handling | 1. Leave Job Names empty; click "Fetch Jobs".<br>2. Enter invalid time (e.g., "25:00:00"); click "Fetch Jobs". | - Warning popup: "Input Required" for empty fields.<br>- For invalid time, app handles gracefully (e.g., no fetch or error message); no crash. |

### 3. End-Time Summary
**Objective**: Verify the summary popup at monitoring end.

| Test Case ID | Description | Steps | Expected Result |
|--------------|-------------|-------|-----------------|
| UAT-007 | Summary with sample data | 1. Run fetch for a batch with known completed/submitted jobs.<br>2. Wait for end time. | - Popup titled "Batch Results".<br>- Shows: "Job Monitoring Finished (Interval End: HH:MM:SS)"<br>- Counts: ✅ Completed Normally: X<br>📤 Submitted: Y<br>❌ Missing Jobs: "JOB1, JOB2" or "None".<br>- Counts accurate based on records (e.g., "completed normally" for completed, "submitted" for submitted). |

### 4. Export Functionality
**Objective**: Verify data export to Excel.

| Test Case ID | Description | Steps | Expected Result |
|--------------|-------------|-------|-----------------|
| UAT-008 | Export fetched records | 1. Fetch jobs (ensure some records in table).<br>2. Click "Export to Excel".<br>3. Note the saved file path. | - Success popup: "Records exported to [filepath]" (e.g., data/job_records_HH-MM-SS_HH-MM-SS.xlsx).<br>- Excel file opens/creates with sheets or columns for jobs, timestamps, messages.<br>- Records sorted by input job order.<br>- Includes all fetched data; no corruption. |
| UAT-009 | Export with no data | 1. Fetch with no matching jobs (empty table).<br>2. Click "Export to Excel". | - Exports empty or minimal file (e.g., headers only).<br>- Success message shown; no error. |

### 5. Edge Cases and Error Handling
**Objective**: Verify robustness.

| Test Case ID | Description | Steps | Expected Result |
|--------------|-------------|-------|-----------------|
| UAT-010 | No AS400 connection | 1. Temporarily break .env (e.g., wrong password).<br>2. Click "Fetch Jobs". | - Error handling: Popup or console error (e.g., connection failed); no crash.<br>- Table empty; monitoring stops gracefully. |
| UAT-011 | Large job list | 1. Enter 50+ jobs (beyond batches).<br>2. Fetch. | - Handles large queries; no timeout/crash.<br>- Performance acceptable (table loads within 10s). |
| UAT-012 | Time range with no data | 1. Set future time range (e.g., tomorrow).<br>2. Fetch. | - Table empty; continues monitoring until end.<br>- Summary shows all jobs as missing. |

## Post-Testing
- **Verify Exports**: Open Excel files; check data integrity.
- **Logs**: Review console for errors during tests.
- **Cleanup**: Delete test Excel files if needed.
- **Feedback**: Document any issues (e.g., screenshots of errors, unexpected behavior).

## Approval
- Tester Name: ____________________
- Date: ____________________
- All tests passed: Yes/No
- Comments: ____________________

This UAT ensures the application is ready for client use. Contact the development team for any failures.
