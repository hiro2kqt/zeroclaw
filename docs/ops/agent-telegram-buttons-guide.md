# Configuring Your ZeroClaw Agent to Use Telegram Inline Buttons

This guide explains how to enable your ZeroClaw agent to use inline buttons in Telegram conversations.

## Quick Start

Add this to your agent's system prompt or instructions:

```markdown
## Telegram Inline Buttons

When responding on Telegram, you can include interactive buttons using this syntax:

**Syntax:**
```
Your message text here.

[BUTTONS]
[Button Text|callback_data]
[Button Text|callback_data] [Another Button|callback_data]
[BUTTONS]
```

**Rules:**
- Each line between [BUTTONS] markers = one row of buttons
- Multiple buttons on same line = side-by-side layout
- Format: `[Display Text|callback_data]` for callbacks
- Format: `[Display Text|https://url]` for links
- Keep button text under 20 characters
- Callback data must be unique and descriptive
- Max 8 rows, 2-4 buttons per row recommended

**When to use buttons:**
- Yes/No confirmations
- Multiple choice questions
- Quick action menus
- Navigation (Next/Previous/Cancel)
- Language selection
- Settings toggles

**Example 1 - Confirmation:**
```
Ready to deploy the application?

[BUTTONS]
[✅ Deploy Now|deploy_confirm] [❌ Cancel|deploy_cancel]
[BUTTONS]
```

**Example 2 - Menu:**
```
What would you like to do?

[BUTTONS]
[📊 View Stats|action_stats]
[⚙️ Settings|action_settings]
[📝 Logs|action_logs]
[❓ Help|action_help]
[BUTTONS]
```

**Handling clicks:**
When users click buttons, you'll receive: `[Button: callback_data]`
Respond naturally based on the callback data.

**Example flow:**
```
You: Choose your language:
[BUTTONS]
[English|lang_en] [日本語|lang_ja] [中文|lang_zh]
[BUTTONS]

User clicks "日本語"
You receive: [Button: lang_ja]

You: 言語を日本語に設定しました！
```
```

## System Prompt Example

Add to your `config.toml` or agent configuration:

```toml
[agent]
system_prompt = """
You are a helpful AI assistant running on Telegram.

# Capabilities
- You can send interactive buttons for user choices
- Use buttons for confirmations, menus, and quick actions
- When you send buttons, format them using the [BUTTONS] syntax

# Button Syntax
[BUTTONS]
[Text|callback] [Text|callback]
[Text|https://url]
[BUTTONS]

# Guidelines
- Use buttons to simplify common actions
- Provide both button and text options
- Keep button text concise
- Use emojis in buttons for clarity
- Always explain what buttons do

# Example Response
"I can help you with that. Choose an option:

[BUTTONS]
[✅ Proceed|action_proceed] [❌ Cancel|action_cancel]
[📖 Learn More|https://docs.example.com]
[BUTTONS]"
"""
```

## Advanced Configuration

### Environment-Specific Instructions

For development/testing:

```toml
[agent]
telegram_button_mode = "verbose"  # Show callback data in responses
```

For production:

```toml
[agent]
telegram_button_mode = "normal"   # Hide technical details
```

### Callback Data Naming Convention

Recommended prefixes:
- `action_*` - Actions (deploy, restart, delete)
- `nav_*` - Navigation (next, prev, back, home)
- `select_*` - Selections (lang_en, theme_dark)
- `toggle_*` - Toggles (enable, disable)
- `confirm_*` - Confirmations (yes, no, maybe)

Example:
```
[BUTTONS]
[Deploy|action_deploy] [Settings|nav_settings] [Cancel|action_cancel]
[BUTTONS]
```

## Testing Buttons

### Manual Test

1. Send a message with button syntax to your bot
2. Verify buttons appear below the message
3. Click a button
4. Check the bot receives `[Button: callback_data]`
5. Verify the bot responds appropriately

### Test Script

```bash
# Send test message via zeroclaw CLI
echo 'Test buttons:

[BUTTONS]
[Option A|test_a] [Option B|test_b]
[BUTTONS]' | zeroclaw send --channel telegram --user YOUR_CHAT_ID
```

## Common Patterns

### Confirmation Pattern
```
Are you sure you want to delete 42 files?

[BUTTONS]
[🗑️ Delete|confirm_delete] [Cancel|confirm_cancel]
[BUTTONS]
```

### Pagination Pattern
```
Showing results 1-10 of 100.

[BUTTONS]
[⬅️ Previous|page_prev] [➡️ Next|page_next]
[🏠 Home|nav_home]
[BUTTONS]
```

### Multi-step Flow Pattern
```
Step 1/3: Select environment

[BUTTONS]
[Production|env_prod] [Staging|env_staging] [Development|env_dev]
[BUTTONS]
```

When button clicked:
```
Step 2/3: Confirm deployment to Production

[BUTTONS]
[✅ Confirm|deploy_confirm] [❌ Cancel|deploy_cancel]
[⬅️ Back|step_back]
[BUTTONS]
```

## Troubleshooting

**Buttons not appearing:**
- Check syntax (must be exactly `[BUTTONS]....[BUTTONS]`)
- Verify callback_data has no spaces
- Ensure you're on Telegram channel (not Discord/Slack)

**Button clicks not received:**
- Check allowed_users in config
- Verify bot has permission to receive callback_query updates
- Check logs for authorization errors

**Buttons appear but don't work:**
- Callback data might be too long (max 64 bytes)
- Special characters in callback_data
- Network/API issues (check logs)

## Migration Notes

If updating from older version without button support:
1. Rebuild: `cargo build --release`
2. Restart the agent
3. Update system prompt with button syntax
4. Test with a simple button message
5. No config changes needed (backward compatible)

Existing messages without buttons continue to work normally.
