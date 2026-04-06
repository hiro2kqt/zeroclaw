# Telegram Inline Buttons

ZeroClaw's Telegram channel supports inline keyboard buttons for interactive messages.

## Overview

Inline buttons appear directly below messages in Telegram and allow users to tap options without typing. The agent receives button clicks as callback events.

## Usage

### Sending Buttons

To send a message with buttons, use the `[BUTTONS]` marker in your response:

```
Here are your options:

[BUTTONS]
[Yes|confirm_yes] [No|confirm_no]
[Maybe Later|confirm_later]
[BUTTONS]
```

**Format:**
- Each line between `[BUTTONS]` markers represents a row
- Buttons in the same line appear side-by-side
- Format: `[Text|callback_data]` for callback buttons
- Format: `[Text|https://url]` for URL buttons (links open in browser)

### Example 1: Yes/No Confirmation

```
Would you like to proceed with this action?

[BUTTONS]
[✅ Yes|action_confirm] [❌ No|action_cancel]
[BUTTONS]
```

### Example 2: Multiple Rows

```
Select your language:

[BUTTONS]
[English|lang_en] [日本語|lang_ja]
[Español|lang_es] [Français|lang_fr]
[中文|lang_zh] [한국어|lang_ko]
[BUTTONS]
```

### Example 3: Mixed Callback and URL Buttons

```
Need help?

[BUTTONS]
[📖 Documentation|https://docs.zeroclaw.dev]
[💬 Contact Support|support_ticket]
[BUTTONS]
```

## Handling Button Clicks

When a user clicks a button, the agent receives a message like:

```
[Button: confirm_yes]
```

The `callback_data` field contains the button's callback data (e.g., `confirm_yes`).

**Example conversation flow:**

```
Agent: Would you like to deploy now?

[BUTTONS]
[Deploy|deploy_confirm] [Cancel|deploy_cancel]
[BUTTONS]

User: *clicks "Deploy"*

Agent receives: [Button: deploy_confirm]

Agent: Deployment initiated! Starting build process...
```

## Best Practices

1. **Keep text short**: Button text should be concise (max ~20 chars)
2. **Use clear callback_data**: Make callback identifiers descriptive
3. **Limit buttons per row**: 2-3 buttons per row for readability
4. **Max 8 rows**: Telegram allows up to 8 rows of buttons
5. **Use emojis**: Emojis make buttons more visually appealing
6. **Provide fallback**: Always allow text-based responses too

## Callback Data Guidelines

- Use descriptive prefixes: `action_`, `nav_`, `setting_`
- Keep under 64 bytes (Telegram limit)
- Use snake_case: `confirm_yes`, `lang_en`, `support_ticket`
- Avoid spaces and special characters

## Error Handling

If button syntax is invalid:
- Malformed buttons are ignored
- Message sends without buttons
- Check logs for parsing errors

## Limitations

- Buttons only work in Telegram channel
- Other channels (Discord, Slack, etc.) ignore button markers
- URL buttons open externally (no callback received)
- Callback data max 64 bytes per button
