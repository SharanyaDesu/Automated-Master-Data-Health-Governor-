# Automated-Master-Data-Health-Governor

This project automates the entire database health audit process using three AI agents in n8n. It scans a PostgreSQL database for data-quality issues, gets higher authority approval before applying any fixes, runs SQL corrections automatically, and delivers a professional report via Google Docs and email all with a single click.

# ⚙️ Tech Stack

n8n --> Workflow automation platform

PostgreSQL --> Target database

Grok / Google Gemini --> AI models powering the agents

Gmail -->  Email notifications and approvals

Google Docs / Google Drive --> Report generation and sharing

# Step 1 - Trigger and Database Investigation

Section label in workflow: Investigate the database and list down the issues

<img width="841" height="507" alt="image" src="https://github.com/user-attachments/assets/f486071f-84c6-4440-a075-dcd5a118d72a" />

# What happens here:

The workflow starts with a Manual Trigger (clicking "Execute Workflow" in n8n).

This fires the Data Investigator Agent, which is powered by the Grok Chat Model.

The agent is equipped with two tools:

Execute a SQL Query in PostgreSQL - reads the full schema and samples rows from all tables.

Call n8n Workflow Tool - allows the agent to invoke sub-workflows if needed.


A Structured Output Parser formats the agent's findings into a clean JSON list of issues.

The output passes into a Split Out node, which separates each issue into its own item for individual processing downstream.

# Issues identified by the agent:

Placeholder values like "N/A" or "UNKNOWN"

Inconsistent date formats (e.g., DD/MM/YYYY vs YYYY-MM-DD)

NULL values in required/non-nullable fields

Delete duplicate records

# Step 2 - Initial Approval Process

Section label in workflow: Initial Approval Process

<img width="642" height="532" alt="image" src="https://github.com/user-attachments/assets/ef783d7e-690a-412d-a011-9e3961658e12" />

# What happens here:

Each issue from Step 1 is passed through a Filter node.

Issues marked as "Fixable with SQL" continue to the approval flow.

Issues that require policy decisions or team discussion are routed out and logged separately (via the false branch of the If node --> No Operation, do nothing).

For the fixable issues, an email is sent via Gmail using "Send message and wait for response" mode.

The email asks the manager/stakeholder: "Are you available to review and approve SQL fixes?"

The workflow pauses and waits for a YES/NO reply before proceeding.

If the response is YES (true branch) --> items go to the Edit Fields node to prepare them for the fix loop, then into Split Out1.

If the response is NO (false branch) --> the workflow stops with No Operation, do nothing.

# Key design decision:

No SQL fix is ever applied without explicit human approval. This ensures full auditability.

# Step 3 - Fix Issues with SQL (Data Cleaner Agent)

Section label in workflow: #3 Fixed with SQL

<img width="1162" height="537" alt="Workflow_2" src="https://github.com/user-attachments/assets/221469ad-16f8-45d9-b91c-4944aa4112d8" />

# What happens here:

Approved issues enter a Loop Over Items node that processes each issue one by one.

For each issue in the loop: A detailed email is sent via Gmail ("Send message and wait for response1") containing:

Description of the issue

The proposed SQL fix

Approve / Reject buttons


The workflow waits for the per-issue response, evaluated in the If1 node.

If approved (true branch), the Data Cleaner Agent runs:

Powered by Google Gemini Chat Model

Tools available to it:

Execute a SQL Query in PostgreSQL1 --> runs the actual fix query on the live database

Call 'Sql_Query_result_generator' --> a sub-workflow call for handling query results


A Structured Output Parser1 captures the result whether the fix succeeded and the exact SQL used.


The loop then continues to the next item.

When all items are processed, the Loop Over Items node exits via the done path.

What the Data Cleaner Agent does:

Generates the precise UPDATE or ALTER SQL statement for each issue

Executes it directly against the database

Returns a fix status --> success / failed and the exact query used

# Step 4 - Create Executive Summary & Deliver Report

Section label in workflow: #4 Create Executive Summary

<img width="1222" height="422" alt="Workflow_3" src="https://github.com/user-attachments/assets/3d9f0292-5a87-470a-97a4-44ebaa544ddd" />

# What happens here:

Results from two paths are combined:

Input 1: The original issue list (all issues found - fixed and unfixed)

Input 2: The SQL fix outputs from the Data Cleaner agent


A Merge node combines both streams, followed by an Aggregate node that consolidates everything into a single data package.

The Report Generator Agent (Google Gemini Chat Model1) receives the full picture then groups issues into

Fixed, Failed, and Needs Discussion

Uses Create a document in Google Docs to generate a professionally formatted report with headings, spacing, and bullet points

Uses Update a document in Google Docs to finalize content

Uses Share file in Google Drive to make the document publicly accessible (view-only)


A Structured Output Parser2 extracts the shareable document link.

Finally, a Send a message (Gmail) node emails the team with a direct link to the finished report.

Report contents:

Issues found during the audit

Which were fixed and with what SQL

Which were skipped/rejected

Which need team policy discussion

# Workflow Diagram

<img width="1465" height="566" alt="workflow" src="https://github.com/user-attachments/assets/d7de3b29-6424-4695-ab45-2d6f984b8a50" />


Note

Actual workflow files and datasets are not included for security and privacy reasons as it contains database credentials, API keys. Workflow architecture images are provided.

# Final Outcome

Trigger --> One click (manual or scheduled)

Database scan --> Fully automated by AI agent

Approval --> Human in the loop before any fix

Data fixes --> Applied automatically with SQL

Report --> Google Doc created, shared, and emailed

Team visibility --> Everyone receives the link via Gmail






