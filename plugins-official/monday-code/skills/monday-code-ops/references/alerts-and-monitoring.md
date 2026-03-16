# Alerts and Monitoring Reference

Detailed documentation for setting up and managing monday code alerts.

## Overview

monday code alerts help you proactively monitor backend applications and catch issues before they affect end-users. Alerts auto-create a monday.com board for notifications with context, timestamps, and regional insights.

## Alert Types

| Alert Type | What It Monitors | Suggested Threshold | Suggested Window |
|---|---|---|---|
| HTTP error rate | Percentage of HTTP requests returning 4xx/5xx status codes | 1-5% | 10-15 minutes |
| HTTP latency response | Percentage of response times exceeding a threshold at a given percentile | 500-2000ms at P95/P99 | 5-10 minutes |
| Runtime limit | Daily runtime execution limit quota usage | 70-80% | N/A (checked periodically) |

## Setup Steps

### Create an Alert

1. Open your app in the Developer Center
2. Navigate to **Host on monday**
3. Click **Server-side code**
4. Select the **Alert policies** tab
5. Click **Create alert**

### Configure an Alert

1. **First time only:** Select a workspace to create the alert board in
2. Enter a descriptive name (e.g., "High Error Rate - Production")
3. Select the alert type from the Metric Type dropdown
4. Configure the parameters:

**HTTP error rate:**
- **Threshold above (%):** Error rate percentage (e.g., 5% means 5 out of 100 requests are errors)
- **Time window (minutes):** The evaluation period

**HTTP latency response:**
- **Threshold above (ms):** Maximum acceptable response time
- **Percentile:** What percentile of requests can exceed the threshold (e.g., P95, P99)
- **Time window (minutes):** The evaluation period

**Runtime limit:**
- **Threshold above (%):** Percentage of daily runtime limit consumed

## Alert Board

When you create your first alert, monday code auto-generates a board in your selected workspace. This board:

- Receives new items when alerts trigger
- Contains columns for: Alert Name, Type, Status, Region, Timestamp, Details
- Supports standard monday.com automations and integrations

### Working with the Alert Board

**Query alerts programmatically:**
```
mcp__monday__get_board_items_page({ boardId: ALERT_BOARD_ID })
```

**Build automations on the alert board:**
- Send Slack notifications when a new alert item is created
- Assign alerts to team members based on alert type
- Move alerts through a status workflow (New > Investigating > Resolved)
- Trigger external webhooks for PagerDuty/Opsgenie integration

**Find the alert board ID:**
- Look in Developer Center > App > Alerts section
- The board URL contains the board ID: `https://<slug>.monday.com/boards/<BOARD_ID>`

## Best Practices

1. **Start with conservative thresholds** and tighten over time based on baseline metrics
2. **Set up alerts before promoting to live** to catch issues early
3. **Build notification automations** on the alert board (Slack, email, etc.)
4. **Use multiple alert types together** - error rate + latency + runtime limit gives comprehensive coverage
5. **Review alert board regularly** to identify patterns and recurring issues
6. **For multi-region apps**, alerts are global (apply to all regions) - the alert details include the region where the issue occurred
