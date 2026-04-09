# Column Types Reference

Quick reference for JSON formats when reading and writing monday.com column values via the GraphQL API.

> For the full and up-to-date list of column types and their formats, see the [official Column Types Reference](https://developer.monday.com/api-reference/reference/column-types-reference).

## How Column Values Work

Column values in the monday.com API use JSON strings. When writing, the `column_values` argument takes a **stringified JSON object** keyed by column ID:

```javascript
const columnValues = JSON.stringify({
  "status": { "label": "Done" },
  "date4": { "date": "2026-01-15" },
  "numbers": "42"
});
```

When reading, use `column_values { id type text value }` on items. The `value` field contains raw JSON; the `text` field contains a human-readable string.

For type-specific reading, use inline fragments:
```graphql
column_values {
  id
  type
  ... on StatusValue { label index }
  ... on DateValue { date time }
}
```

## Column Type Reference

| Type | Column Type | Write Format | Read Fields | Example Write |
|------|-------------|-------------|-------------|---------------|
| Auto Number | `auto_number` | Read-only | `value` (number) | N/A |
| Board Relation | `board_relation` | `{"item_ids": [123, 456]}` | `linked_item_ids` | `{"item_ids": [12345]}` |
| Checkbox | `checkbox` | `{"checked": true}` | `checked` (boolean) | `{"checked": true}` |
| Color Picker | `color_picker` | `{"color": "#FF0000"}` | `color`, `changed_at` | `{"color": "#FF5733"}` |
| Country | `country` | `{"countryCode": "US", "countryName": "United States"}` | `country_code`, `country_name` | `{"countryCode": "US", "countryName": "United States"}` |
| Date | `date` | `{"date": "YYYY-MM-DD"}` or `{"date": "YYYY-MM-DD", "time": "HH:MM:SS"}` | `date`, `time` | `{"date": "2026-01-15", "time": "09:00:00"}` |
| Dropdown | `dropdown` | `{"ids": [1, 2]}` | `values` (array of {id, name}) | `{"ids": [3, 7]}` |
| Email | `email` | `{"email": "user@example.com", "text": "Label"}` | `email`, `text` | `{"email": "dev@monday.com", "text": "Contact"}` |
| Hour | `hour` | `{"hour": 14, "minute": 30}` | `hour`, `minute` | `{"hour": 9, "minute": 0}` |
| Item ID | `item_id` | Read-only | `item_id` | N/A |
| Link | `link` | `{"url": "https://...", "text": "Label"}` | `url`, `text` | `{"url": "https://monday.com", "text": "Monday"}` |
| Location | `location` | `{"lat": 40.7128, "lng": -74.0060, "address": "New York"}` | `lat`, `lng`, `address` | `{"lat": 40.7128, "lng": -74.006, "address": "NYC"}` |
| Long Text | `long_text` | `{"text": "content"}` | `text`, `updated_at` | `{"text": "Detailed description here"}` |
| Mirror | `mirror` | Read-only (derived from connected boards) | `display_value` | N/A |
| Name | `name` | Set via `item_name` argument, not `column_values` | `name` on the item | N/A |
| Numbers | `numbers` | `"42"` (number as string, or just the number) | `number` (float) | `"99.5"` |
| People | `people` | `{"personsAndTeams": [{"id": N, "kind": "person"}]}` | `persons_and_teams` array | `{"personsAndTeams": [{"id": 123, "kind": "person"}]}` |
| Phone | `phone` | `{"phone": "+1234567890", "countryShortName": "US"}` | `phone`, `country_short_name` | `{"phone": "+15551234567", "countryShortName": "US"}` |
| Rating | `rating` | `{"rating": 4}` | `rating` (1-5) | `{"rating": 5}` |
| Status | `status` | `{"index": N}` or `{"label": "Label"}` | `label`, `index`, `label_style` | `{"label": "Done"}` |
| Tags | `tags` | `{"tag_ids": [1, 2]}` | `tag_ids`, `tags` | `{"tag_ids": [100, 200]}` |
| Team | `team` | `{"team_id": 123}` | `team_id` | `{"team_id": 456}` |
| Text | `text` | `"hello"` (plain string) | `text` | `"hello world"` |
| Timeline | `timeline` | `{"from": "YYYY-MM-DD", "to": "YYYY-MM-DD"}` | `from`, `to`, `visualization_type` | `{"from": "2026-01-01", "to": "2026-03-31"}` |
| Time Tracking | `time_tracking` | Read-only (managed via UI) | `duration`, `started`, `additional_value` | N/A |
| Vote | `vote` | Read-only | `voter_ids`, `changed_at` | N/A |
| Week | `week` | `{"week": {"startDate": "YYYY-MM-DD", "endDate": "YYYY-MM-DD"}}` | `start_date`, `end_date` | `{"week": {"startDate": "2026-01-06", "endDate": "2026-01-12"}}` |
| World Clock | `world_clock` | `{"timezone": "America/New_York"}` | `timezone` | `{"timezone": "Europe/London"}` |
| Progress | `progress` | Read-only (calculated from dependent items) | `progress` (percentage) | N/A |
| Formula | `formula` | Read-only (calculated) | `display_value` | N/A |
| Creation Log | `creation_log` | Read-only | `created_at`, `creator_id` | N/A |
| Last Updated | `last_updated` | Read-only | `updated_at`, `updater_id` | N/A |
| Dependency | `dependency` | `{"item_ids": [123]}` | `linked_item_ids` | `{"item_ids": [789]}` |
| Subtasks | `subtasks` | Read-only (managed via subitems) | `linked_item_ids` | N/A |
| File | `file` | Use `add_file_to_column` mutation instead | `files` array | N/A |
| Button | `button` | Read-only (triggers via UI) | `label`, `color` | N/A |

## Common Mistakes

1. **Passing a raw object instead of a JSON string** — `column_values` expects a stringified JSON string, not a raw object
2. **Wrong status format** — Use `{"label": "Done"}` not `"Done"` or `{"status": "Done"}`
3. **Numbers as objects** — Numbers column takes a plain string `"42"`, not `{"value": 42}`
4. **Text as objects** — Text column takes a plain string `"hello"`, not `{"text": "hello"}`
5. **People without kind** — Must include `"kind": "person"` or `"kind": "team"` in each entry
6. **Date without quotes** — Date value must be a string `"2026-01-15"`, not an unquoted value
7. **File upload via column_values** — Cannot set file columns via `column_values`. Use the `add_file_to_column` mutation instead.

## Using change_multiple_column_values

Prefer `change_multiple_column_values` over multiple `change_simple_column_value` calls:

```graphql
mutation ($boardId: ID!, $itemId: ID!, $columnValues: JSON!) {
  change_multiple_column_values(
    board_id: $boardId
    item_id: $itemId
    column_values: $columnValues
  ) {
    id
    name
  }
}
```

Variables:
```json
{
  "boardId": "1234567890",
  "itemId": "9876543210",
  "columnValues": "{\"status\": {\"label\": \"Done\"}, \"date4\": {\"date\": \"2026-01-15\"}, \"numbers\": \"42\"}"
}
```

## Finding Column IDs

Column IDs are NOT the display names. To find column IDs for a board:

```graphql
query {
  boards(ids: [BOARD_ID]) {
    columns {
      id
      title
      type
    }
  }
}
```
