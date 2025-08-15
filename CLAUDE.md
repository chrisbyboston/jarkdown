# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Setup and Development
```bash
# Create virtual environment and install dependencies
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Run the tool
./jira-download ISSUE-KEY
./jira-download ISSUE-KEY --output /path/to/output
./jira-download ISSUE-KEY --verbose

# Direct Python execution (if in venv)
python jira_download.py ISSUE-KEY
```

### Testing Connectivity
```bash
# Test Jira API connection
source venv/bin/activate && python test_connection.py

# Find issues with images for testing
source venv/bin/activate && python find_issue_with_images.py
```

## Architecture

### Core Implementation (`jira_download.py`)
Single-file Python implementation with `JiraDownloader` class that:
1. Authenticates with Jira Cloud API using Basic Auth (email + API token)
2. Fetches issue data using `/rest/api/3/issue/{ISSUE-KEY}` with `expand=renderedFields`
3. Downloads attachments via their content URLs with streaming for large files
4. Converts HTML to Markdown using `markdownify` library
5. Replaces Jira attachment URLs with local file references using regex patterns
6. Composes final markdown with metadata, description, and attachments sections

### Key Design Decisions
- **HTML-to-Markdown conversion** chosen over direct ADF-to-Markdown for reliability
- **Attachment URL replacement** uses regex patterns to handle multiple Jira URL formats
- **Filename conflict handling** appends counters to duplicate filenames
- **Image detection** via MIME types to embed images vs linking other files
- **Virtual environment wrapper** (`jira-download`) ensures correct Python environment

### Environment Configuration
Credentials loaded from `.env` file:
- `JIRA_DOMAIN`: Atlassian domain (e.g., company.atlassian.net)
- `JIRA_EMAIL`: User email for authentication
- `JIRA_API_TOKEN`: API token from Atlassian account settings

### Output Structure
Creates directory named after issue key containing:
- `{ISSUE-KEY}.md`: Markdown file with issue content
- Downloaded attachments with original filenames (conflicts resolved with counters)

## Current Limitations
- Single issue export only (no bulk operations yet)
- Comments not included (marked as future enhancement)
- Jira Cloud only (not Server/Data Center)
- No direct ADF parsing (uses HTML rendering instead)