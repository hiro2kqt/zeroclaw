# Telegram Inline Buttons - Quick Start

## For Agent Users (How to Tell Your Agent to Use Buttons)

After rebuilding and rerunning ZeroClaw with the button feature, add this to your agent's system prompt:

### Option 1: Simple Instruction (Recommended)

Add to your agent's system prompt or instructions:

```markdown
When responding on Telegram, you can add interactive buttons using this syntax:

[BUTTONS]
[Button Text|callback_data] [Another Button|callback_data]
[Third Button|callback_data]
[BUTTONS]

Examples:

1. Yes/No question:
Choose an option:

[BUTTONS]
[✅ Yes|confirm_yes] [❌ No|confirm_no]
[BUTTONS]

2. Menu:
What would you like to do?

[BUTTONS]
[View Stats|action_stats]
[Settings|action_settings]
[Help|https://docs.example.com]
[BUTTONS]

When users click buttons, you'll receive: [Button: callback_data]
Respond naturally based on what they clicked.
```

### Option 2: Configuration File

Edit your `config.toml`:

```toml
[agent]
system_prompt = """
You are a helpful assistant on Telegram with button support.

# Telegram Button Syntax
[BUTTONS]
[Text|callback_data]
[Text|https://url]
[BUTTONS]

Use buttons for:
- Confirmations (Yes/No)
- Multiple choice
- Quick actions
- Navigation
"""
```

### Option 3: Runtime Instruction

Just tell your agent during a conversation:

```
You can now send interactive buttons in Telegram. Use this format:

[BUTTONS]
[Button Text|callback_id]
[BUTTONS]

Try it on your next response!
```

## Testing It Works

1. Rebuild: `cargo build --release`
2. Restart ZeroClaw
3. Send a message to your bot on Telegram
4. Ask: "Can you send me some buttons to test?"
5. The agent should respond with clickable buttons

## Example Agent Response

**User:** "Should I proceed with deployment?"

**Agent:**
```
Deploying to production is a significant action. Please confirm:

[BUTTONS]
[✅ Deploy Now|deploy_confirm] [❌ Cancel|deploy_cancel]
[📖 Review Changes|deploy_review]
[BUTTONS]
```

**User:** *clicks "Deploy Now"*

**Agent receives:** `[Button: deploy_confirm]`

**Agent:** "Deployment initiated! Starting build process..."

## Button Syntax Rules

1. **Format**: `[Display Text|callback_data]`
2. **URLs**: `[Link Text|https://example.com]` (opens in browser)
3. **Rows**: Each line = one row of buttons
4. **Side-by-side**: Multiple buttons on same line appear horizontally
5. **Limits**: Max 8 rows, 2-4 buttons per row recommended
6. **Callback data**: Must be unique, no spaces, under 64 bytes

## Common Patterns

### Confirmation
```
Are you sure?

[BUTTONS]
[Yes|yes] [No|no]
[BUTTONS]
```

### Pagination
```
Results 1-10 of 100

[BUTTONS]
[⬅️ Previous|page_prev] [➡️ Next|page_next]
[BUTTONS]
```

### Menu
```
Choose a category:

[BUTTONS]
[📊 Analytics|cat_analytics]
[⚙️ Settings|cat_settings]
[📝 Logs|cat_logs]
[❓ Help|cat_help]
[BUTTONS]
```

## Troubleshooting

**Buttons don't appear:**
- Check syntax exactly matches `[BUTTONS]...[BUTTONS]`
- Ensure callback_data has no spaces
- Verify you're using Telegram (not Discord/Slack)

**Clicks not received:**
- Check allowed_users in config includes the clicking user
- Look for authorization errors in logs

**Already rebuilt but agent doesn't use buttons:**
- Add the button syntax to system prompt (see Option 1 above)
- Test by explicitly asking agent to send buttons
- Check logs for parsing errors

## Full Documentation

- **API Reference**: `docs/reference/telegram-inline-buttons.md`
- **Configuration Guide**: `docs/ops/agent-telegram-buttons-guide.md`
- **Code**: `src/channels/telegram.rs` (parse_button_markers function)

## Quick Test Command

```bash
# After rebuild, test manually:
echo 'Test:

[BUTTONS]
[A|test_a] [B|test_b]
[BUTTONS]' > /tmp/test.txt

# Then paste that message in Telegram
```

## Key Points

✅ Backward compatible - old messages still work
✅ Works in both text syntax and programmatic API
✅ Buttons only appear on Telegram, ignored on other platforms
✅ URL buttons open externally, callback buttons send data to agent
✅ Agent receives `[Button: callback_data]` when clicked

The agent will naturally learn to use buttons if you add the syntax to its instructions. Start with simple yes/no buttons and expand from there!
