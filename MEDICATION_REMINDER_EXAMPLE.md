# Medication Reminder Example - How It Works

## User Request (Vietnamese)
```
"giờ mỗi ngày lúc 11h trưa đến 14h thì mỗi 30p nhắc tôi 1 lần uống thuốc,
lúc nhắc thì thêm nút đã uống vào, nếu tôi bấm đã uống thì tắt thông báo
cho đến ngày mai"
```

**Translation:**
"Now every day from 11am to 2pm, remind me every 30 minutes to take medicine.
When reminding, add a 'taken' button. If I click 'taken', turn off notifications
until tomorrow."

## What the Agent Will Do (After Rebuild)

### Step 1: Understand the Request
The agent analyzes your message and identifies:
- ✅ Schedule: Every 30 minutes between 11:00-14:00 daily
- ✅ Action: Send reminder to Telegram
- ✅ Button needed: "Đã uống" (Taken) button
- ✅ Button behavior: When clicked, disable reminders until next day

### Step 2: Create Cron Jobs
The agent will use the `cron_add` tool to create scheduled jobs:

**Job 1: 11:00 reminder**
```json
{
  "name": "medication_reminder_11_00",
  "schedule": {
    "kind": "cron",
    "expr": "0 11 * * *",
    "tz": "Asia/Ho_Chi_Minh"
  },
  "job_type": "agent",
  "prompt": "Send medication reminder to user via Telegram with 'taken' button",
  "delivery": {
    "mode": "announce",
    "channel": "telegram",
    "to": "<your_chat_id>"
  }
}
```

**Job 2: 11:30 reminder**
```json
{
  "schedule": { "kind": "cron", "expr": "30 11 * * *", "tz": "Asia/Ho_Chi_Minh" }
}
```

**Job 3: 12:00 reminder**
```json
{
  "schedule": { "kind": "cron", "expr": "0 12 * * *", "tz": "Asia/Ho_Chi_Minh" }
}
```

**Job 4: 12:30 reminder**
```json
{
  "schedule": { "kind": "cron", "expr": "30 12 * * *", "tz": "Asia/Ho_Chi_Minh" }
}
```

**Job 5: 13:00 reminder**
```json
{
  "schedule": { "kind": "cron", "expr": "0 13 * * *", "tz": "Asia/Ho_Chi_Minh" }
}
```

**Job 6: 13:30 reminder**
```json
{
  "schedule": { "kind": "cron", "expr": "30 13 * * *", "tz": "Asia/Ho_Chi_Minh" }
}
```

### Step 3: Format Reminder Message with Button
Each reminder will be sent with this format:

```
⏰ Nhắc nhở uống thuốc!

Đã đến giờ uống thuốc. Hãy nhớ uống đúng liều lượng nhé!

[BUTTONS]
[✅ Đã uống|med_taken_<timestamp>] [⏰ Nhắc lại sau 10p|med_snooze_<timestamp>]
[BUTTONS]
```

### Step 4: Handle Button Clicks
When you click "✅ Đã uống":

1. Agent receives: `[Button: med_taken_<timestamp>]`
2. Agent uses `cron_list` to find all active medication reminder jobs
3. Agent uses `cron_remove` to disable those jobs
4. Agent responds: "Đã ghi nhận! Tôi sẽ nhắc lại vào ngày mai."
5. Agent creates new jobs for tomorrow using `cron_add`

## How the Agent Knows What to Do

After you rebuild with the latest code, the agent's system prompt includes:

### 1. Telegram Button Capabilities (Automatic)
```markdown
## Telegram Interactive Buttons

On Telegram, you can send interactive inline buttons...

[BUTTONS]
[Button Text|callback_data]
[BUTTONS]

Example - Reminder with action:
⏰ Time to take your medication!

[BUTTONS]
[✅ Taken|med_taken] [⏰ Snooze 10min|med_snooze]
[BUTTONS]
```

### 2. Cron Tools (Already Available)
- `cron_add` - Create scheduled jobs
- `cron_list` - List active jobs
- `cron_remove` - Remove jobs
- `cron_update` - Update job schedules

### 3. Memory System (For Tracking State)
The agent can use `memory_write` to track:
- When you last took medication
- Which reminders are active
- When to re-enable reminders

## Complete Flow Example

### Initial Setup
**You:** "giờ mỗi ngày lúc 11h trưa đến 14h thì mỗi 30p nhắc tôi 1 lần uống thuốc..."

**Agent:**
```
Được rồi! Tôi đã thiết lập lịch nhắc uống thuốc cho bạn:

⏰ Thời gian: 11:00, 11:30, 12:00, 12:30, 13:00, 13:30 (mỗi ngày)
✅ Nút hành động: "Đã uống" để tắt nhắc nhở
🔄 Tự động reset vào ngày hôm sau

Tôi đã tạo 6 cron jobs cho lịch này. Bạn có thể kiểm tra bằng lệnh "hiển thị lịch nhắc" bất cứ lúc nào.
```

### Daily Reminder (11:00 AM)
**Agent sends:**
```
⏰ Nhắc nhở uống thuốc!

Đã đến giờ 11:00 - đừng quên uống thuốc nhé! 💊

[BUTTONS]
[✅ Đã uống|med_taken_20260406_1100] [⏰ Sau 10p|med_snooze_10]
[BUTTONS]
```

### You Click "✅ Đã uống"
**Agent receives:** `[Button: med_taken_20260406_1100]`

**Agent:**
```
✅ Đã ghi nhận! Bạn đã uống thuốc lúc 11:00.

Tôi đã tắt tất cả lời nhắc hôm nay. Sẽ nhắc lại vào ngày mai cùng giờ.

Thông tin: Đã xóa 5 lời nhắc còn lại (11:30, 12:00, 12:30, 13:00, 13:30)
```

### Next Day (Reset)
**Agent automatically:**
1. Re-creates all 6 cron jobs at midnight
2. Or keeps the jobs and tracks "taken" state in memory
3. Starts reminding again at 11:00 AM

## State Management Approaches

### Approach 1: Stateless (Cron Job Recreation)
- Delete all jobs when "taken" is clicked
- Recreate jobs at midnight via a master cron
- Simple but requires daily job creation

### Approach 2: Stateful (Memory Tracking)
- Keep cron jobs running
- Store "taken" timestamp in memory: `memory_write medication_taken_date 2026-04-06`
- Each job checks memory before sending
- More efficient but requires memory checks

## Example Agent Memory
```json
{
  "medication_schedule": {
    "times": ["11:00", "11:30", "12:00", "12:30", "13:00", "13:30"],
    "timezone": "Asia/Ho_Chi_Minh",
    "last_taken": "2026-04-06T11:00:00+07:00",
    "job_ids": [
      "med_reminder_1",
      "med_reminder_2",
      "med_reminder_3",
      "med_reminder_4",
      "med_reminder_5",
      "med_reminder_6"
    ]
  }
}
```

## Testing After Rebuild

### 1. Rebuild
```bash
cargo build --release
```

### 2. Restart ZeroClaw
```bash
./target/release/zeroclaw
```

### 3. Send Test Message
Send in Telegram:
```
Tạo lời nhắc test cho tôi lúc [current_time + 2 minutes] với nút "Done"
```

### 4. Verify
- Wait 2 minutes
- You should receive a message with a "Done" button
- Click the button
- Agent should acknowledge the click

## Troubleshooting

**Agent says it can't send buttons:**
- Make sure you rebuilt: `cargo build --release`
- Restart the agent completely
- The system prompt should now include the Telegram Buttons section

**Buttons don't appear:**
- Check syntax in agent's message (should see `[BUTTONS]` markers)
- Verify you're on Telegram (not Discord/Slack)
- Check logs for parsing errors

**Button clicks not received:**
- Verify you're in the `allowed_users` list in config
- Check agent logs for callback_query updates
- Ensure bot has proper permissions

**Cron jobs not working:**
- Use `cron_list` tool to verify jobs were created
- Check timezone is correct (Asia/Ho_Chi_Minh for Vietnam)
- Use `cron_runs` to see execution history

## Advanced: Custom Reminder Logic

If you want more complex behavior, you can instruct the agent:

```
"Khi tôi bấm 'Đã uống':
1. Lưu thời gian vào memory
2. Tắt tất cả lời nhắc hôm nay
3. Nếu tôi uống trước 11:30, chỉ nhắc 1 lần vào 13:00
4. Nếu tôi uống sau 13:00, không nhắc nữa
5. Mỗi tuần gửi thống kê: đã uống đúng giờ bao nhiêu lần"
```

The agent will adapt its logic based on these instructions and use the available tools (cron, memory, buttons) to implement it.

## Summary

✅ **After rebuild, the agent knows:**
- It can send Telegram inline buttons
- How to use `[BUTTONS]` syntax
- It has cron tools for scheduling
- It has memory tools for state tracking

✅ **When you make the request:**
- Agent creates 6 cron jobs (11:00, 11:30, 12:00, 12:30, 13:00, 13:30)
- Each job sends a reminder with a "Đã uống" button
- Button callback data includes timestamp for tracking

✅ **When you click "Đã uống":**
- Agent receives `[Button: med_taken_<timestamp>]`
- Agent removes remaining jobs for today
- Agent saves state to memory
- Agent will resume tomorrow

🔧 **No manual configuration needed!**
The system prompt automatically teaches the agent these capabilities after rebuild.
