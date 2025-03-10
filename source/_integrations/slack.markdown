---
title: Slack
description: Instructions on how to add Slack notifications to Home Assistant.
ha_category:
  - Notifications
ha_release: pre 0.7
ha_config_flow: true
ha_domain: slack
ha_iot_class: Cloud Push
ha_codeowners:
  - '@bachya'
  - '@tkdrob'
ha_platforms:
  - notify
  - sensor
ha_integration_type: service
---

The `slack` platform allows you to deliver notifications from Home Assistant to [Slack](https://slack.com/).

## Setup

### Bot posting as you

1. Create a [new app](https://api.slack.com/apps) under your Slack.com account.
2. Click the `OAuth & Permissions` link in the sidebar, under the Features heading.
3. In the Scopes section, add the `chat:write` and `dnd:read` scopes and select `Send messages as user`. Many errors can occur if these options are not set correctly.
4. Scroll up to `OAuth Tokens & Redirect URLs` and click `Install to Workspace`.
5. Copy your `OAuth Access Token` and put that key into the config flow.

<div class='note'>

There is an app credential Verification Token on the Basic Settings of your app. This is **not** the API key you want.

</div>

### Bot posting as its own user

It is also possible to use Slack bots as users. Just create a new bot at https://[YOUR_TEAM].slack.com/apps/build/custom-integration and use the provided token for that. You can add an icon from the frontend for Home Assistant and give the bot a meaningful name.

Don't forget to invite the bot to the room where you want to get the notifications.

### Icons

Slack uses the standard emoji sets used [here](https://www.webpagefx.com/tools/emoji-cheat-sheet/). Alternatively a publicly accessible URL may be used.

<div class='note'>

In order to modify your Slack bot's username and icon, you must ensure your Slack app has the `chat:write.customize` OAuth scope. See [the Slack API documentation](https://api.slack.com/methods/chat.postMessage#authorship) for more information.

The added `notify` service will be named after the chat server the app is installed on. For example, a server named "Slack Chat" wil display as `notify.slack_chat`.

</div>

{% include integrations/config_flow.md %}

One sensor entity will be created:

- **Do Not Disturb Timer**: The amount of time left for Do Not Disturb status.

### Slack Service Data

The following attributes can be placed inside the `data` key of the service call for extended functionality:

| Attribute              | Optional | Description |
| ---------------------- | -------- | ----------- |
| `username`               |      yes | The username of the Slack bot.
| `icon`                   |      yes | The icon of the Slack bot.
| `file`                   |      yes | A file to include with the message; see below.
| `blocks`                 |      yes | Array of [Slack blocks](https://api.slack.com/messaging/composing/layouts). *NOTE*: if using `blocks`, they are shown **in place of** the `message` (note that the `message` is required nonetheless).
| `blocks_template`        |      yes | The same as `blocks`, but able to support [templates](https://www.home-assistant.io/docs/configuration/templating).

Note that using `file` will ignore all usage of `blocks` and `blocks_template` (as Slack does not support those frameworks in messages that accompany uploaded files).

To include a local file with the Slack message, use these attributes underneath the `file` key:

| Attribute              | Optional | Description |
| ---------------------- | -------- | ----------- |
| `path`                   |      no  | A local filepath that has been [whitelisted](/docs/configuration/basic/#allowlist_external_dirs).

To include a remote file with the Slack message, use these attributes underneath the `file` key:

| Attribute              | Optional | Description |
| ---------------------- | -------- | ----------- |
| `url`                    |      no  | A URL that has been [whitelisted](/docs/configuration/basic/#allowlist_external_urls).
| `username`               |      yes | An optional username if the URL is protected by HTTP Basic Auth.
| `password`               |      yes | An optional password if the URL is protected by HTTP Basic Auth.

### Examples

To send a file from local path:

```yaml
message: Message that will be added as a comment to the file.
title: Title of the file.
data:
  file:
    path: /path/to/file.ext
```

To send a file from remote path:

```yaml
message: Message that will be added as a comment to the file.
title: Title of the file.
data:
  file:
    url: "http://site.com/image.jpg"
```

To send a file from remote path that is protected by HTTP Basic Auth:

```yaml
message: Message that will be added as a comment to the file.
title: Title of the file.
data:
  file:
    url: "http://site.com/image.jpg"
    username: user
    password: pass
```

To use the block framework:

```yaml
message: Fallback message in case the blocks don't display anything.
title: Title of the file.
data:
  blocks:
    - type: section
      text:
        type: mrkdwn
        text: 'Danny Torrence left the following review for your property:'
    - type: section
      block_id: section567
      text:
        type: mrkdwn
        text: "<https://example.com|Overlook Hotel> \n :star: \n Doors had too many
          axe holes, guest in room 237 was far too rowdy, whole place felt stuck in
          the 1920s."
      accessory:
        type: image
        image_url: https://is5-ssl.mzstatic.com/image/thumb/Purple3/v4/d3/72/5c/d3725c8f-c642-5d69-1904-aa36e4297885/source/256x256bb.jpg
        alt_text: Haunted hotel image
    - type: section
      block_id: section789
      fields:
      - type: mrkdwn
        text: |-
          *Average Rating*
          1.0
```

Send a message directly to a user by setting the target to their member ID. Here are [instructions](https://www.workast.com/help/articles/61000165203/) to obtain a member ID.

```yaml
message: "Hello there!"
target: "U12345"
title: "Hi"
data:
  blocks: []
```

Send a message to a channel that mentions (@username, highlights a users name in yellow) a user. Here are [instructions](https://www.workast.com/help/articles/61000165203/) to obtain a member ID.

```yaml
message: "<@U12345> your appointment starts soon"
target: "#general"
title: "Reminder"
data:
  blocks: []
```
