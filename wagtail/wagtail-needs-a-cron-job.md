# Scheduled Publishing Did Not Work

## Root Cause

You need to run a cron job on your Wegtail server to publish scheduled pages.

## How To Fix

You need to run a Wagtail/Django manage.py command every 15 (or whatever interval) minutes. Here's how to set up a cron job to do that.
Put this in ./scripts of your Wagtail project.

```bash
#!/bin/bash

# Script to set up cron job for Wagtail scheduled publishing (runs every 15 minutes)

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
PROJECT_DIR="$(cd "$SCRIPT_DIR/.." && pwd)"

PYTHON_BIN="$PROJECT_DIR/.venv/bin/python"
if [ ! -f "$PYTHON_BIN" ]; then
    PYTHON_BIN="$(which python3)"
fi

LOG_DIR="$PROJECT_DIR/logs"
mkdir -p "$LOG_DIR"

CRON_CMD="*/15 * * * * cd $PROJECT_DIR && $PYTHON_BIN manage.py publish_scheduled >> $LOG_DIR/cron_publish.log 2>&1"

# Check if the cron job already exists
(crontab -l 2>/dev/null | grep -F "manage.py publish_scheduled") >/dev/null 2>&1

if [ $? -eq 0 ]; then
    echo "Cron job for publish_scheduled is already configured."
else
    # Append cron job to existing crontab or create new crontab
    (crontab -l 2>/dev/null; echo "$CRON_CMD") | crontab -
    echo "Successfully installed cron job:"
    echo "$CRON_CMD"
fi

```
