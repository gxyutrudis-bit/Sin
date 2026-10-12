# Sinclair Anti-Nuke

A small Discord bot that watches the audit log for bursts of channel deletions,
role deletions, member bans, and member kicks. By default, one action triggers
quarantine (removing that user's roles); the detection window is five seconds.

## Setup

1. Create a bot in the [Discord Developer Portal](https://discord.com/developers/applications)
	and invite it with the `bot` and `applications.commands` scopes.
2. Grant it **View Audit Log**, **Manage Roles**, and **Manage Messages**. Put
	its highest role above the roles it may need to remove. For the optional ban
	response, also grant **Ban Members** and place the bot above the potential
	executor's roles.
3. Enable the **Server Members Intent** and **Message Content Intent** for the
	bot in the Developer Portal. They are needed for kick detection and prefix
	commands, respectively.
4. Copy `.env.example` to `.env`, then set `DISCORD_TOKEN` and `CLIENT_ID`.
	Set `DISCORD_GUILD_ID` to register commands immediately in one server;
	otherwise commands are registered globally and may take time to appear.
	To use image scam detection, also set `OPENAI_API_KEY`.
5. Run `npm install` and `npm start`.

## Docker container

1. Copy `.env.example` to `.env` and fill in your bot credentials.
2. Run `docker compose up --build` from the project root.
3. The bot uses a persistent `./data` volume so your server config and economy data remain across restarts.

Use `/antinuke status`, `/antinuke enable`, `/antinuke disable`,
`/antinuke threshold`, `/antinuke window`, `/antinuke punishment`, and
`/antinuke whitelist-add` and `/antinuke whitelist-remove` in Discord. These commands are restricted to
server administrators. Configuration is saved in `data/config.json`.
`/antinuke logs` shows the latest 10 recorded detections and their outcomes;
up to 25 entries per server are retained across bot restarts.
Use `/enable` and `/disable` to turn external Discord invite blocking on or
off. While enabled, only members with a role added through `/whitelist role`
can post invites to other servers; messages containing those invites are
deleted. Use `/whitelist role` to add or remove an allowed role. Invites to the
current server remain allowed.
Use `/antiscam enable`, `/antiscam disable`, and `/antiscam status` to manage
scam and crypto-scam image detection. `/antiscam logs-channel` sets the
destination for detection reports; leave its channel option empty to clear it.
When enabled, supported image attachments are sent to OpenAI for classification;
messages confidently identified as deceptive promotions are deleted and
recorded in `/antinuke logs` and the configured channel. The feature is disabled
by default and requires `OPENAI_API_KEY`. It reviews at most three images per
message, up to 8 MB each. Images are processed by OpenAI, so enable this only if
that data sharing is appropriate for your server. The classifier can make
mistakes; review logs and ensure the bot has **Manage Messages** and permission
to send messages and embeds in the configured log channel.

Use `/set prefix` with any 1-5 character prefix, such as `!`, to enable
prefix commands; use `off` to disable them. Only `afk`, `av`, `ava`, `cover`,
`cv`, and `role` are available through the configured prefix. For example,
`!afk [reason]`, `!av [@user]`, `!cover [@user]`, and
`!role <@user> <role name>`. Replace `!` with your configured prefix. Prefix
command replies use Discord Components V2 containers. Avatar and cover commands
can target a mentioned user or the user in a replied-to message.
Message Content Intent must be enabled in the Developer Portal for these
commands to work.

Use `/role user role` to assign a role. The `user` value accepts a user mention
or ID; `role` accepts a role mention or ID, an exact name, or a unique starting
part of a name (for example, `rep` for `Representative`). Exact names take
priority; ambiguous partial names require more characters. The command requires
Manage Roles, and the bot must be above the target role and member in the role
hierarchy.

Use `/autoresponder add` with a trigger and response to create an automatic
reply, `/autoresponder remove` to delete a trigger, and `/autoresponder list` to
view configured triggers. Matching is case-insensitive; if multiple triggers
match a message, only the longest trigger responds. Administrators can configure
up to 50 triggers per server. Message Content Intent must be enabled.
Welcome embeds configured with `/edit-embed` support `{user.id}`, `{user.name}`,
`{user.tag}`, `{user.avatar}`, `{server.name}`, `{server.icon}`, and
`{member.count}`; `{user}` inserts a mention.

The server currency game uses `/currency balance`, `/currency leader`,
`/currency daily`, `/currency gamble`, `/currency flip`, `/currency slots`,
`/currency mines`, `/currency blackjack`, `/currency pvp`, and
`/currency cluck`. `/currency give` transfers coins from the configured owner
to a member; `/currency add` adds new coins to a member's balance. These two
commands are restricted to the server owner and the role configured with
`/set owner`. These commands are available as slash commands only. Currency
replies use Discord Components V2 containers. The leader command lists the top
10 coin balances in the server. Players start with 100 virtual coins and can claim a random
800-2,000 coins once every 24 hours. Coin
flip and gamble are even-money 50/50 games; Slots pays 3x-10x for three matching
symbols. Mines uses a 12-tile board where players choose 1-6 bombs; fewer bombs
mean lower cash-out multipliers. Safe picks raise the cash-out value, while
hitting a bomb loses the bet. Bets are capped at 1,000,000 coins. Balances are
separate per server and persist across bot restarts; coins have no real-world
value. `/currency pvp` starts a blackjack duel; each player uses **+ Card** to
hit and **- Card** to stand.
Both bets are held until the duel ends, and unplayed hands stand automatically
after two minutes. Currency command results and interactive game boards are
sent in Discord Components V2 containers. Autoresponder configuration replies
and triggered auto-replies also use V2 containers.

## Limits

Discord delivers these events after an action has happened. This bot can respond
to an identified executor, but cannot prevent the first action, restore deleted
channels or roles, undo bans, or protect against the server owner. Keep a trusted
owner account secure and maintain backups. Audit-log attribution can also be
delayed; if a matching audit entry is not available promptly, the action is
ignored rather than guessing who to punish.