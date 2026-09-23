# code-switching
The constraint

Claude Code reads its settings once, at session start. If you change hook settings in .claude/settings.json during a session, the old hook configuration continues to be cached and used until you restart. Permission rules behave the same way: the change only takes effect after restarting Claude Code. There are open feature requests asking for a reload command or file watching (issues dated March–May 2026 in the search results), so there is no built-in live reload to rely on. Hooks are cached on purpose, for security: this prevents malicious pull requests from injecting hooks that take effect immediately. 

That means the menu bar app can't work by rewriting settings.json. That approach will look correct in your UI and then quietly fail to affect any running session.

The fix: a hook that never changes, pointing at policy that does

You install one hook entry, once. It calls a small binary, code-switch, on every relevant event. That binary reads the currently active profile from a state file each time it runs. The menu bar app only ever writes the state file. Because the hook configuration itself never changes, Claude Code's caching never gets in the way.

How quickly each kind of rule takes effect:

Rail type	Mechanism	When it applies
Hard rails (deny, ask, or allow a tool)	PreToolUse hook, returning a permissionDecision	The very next tool call, even partway through a turn
Soft rails (instructions to the agent)	UserPromptSubmit and SessionStart hooks, returning additionalContext	The next prompt
Model, env vars, MCP servers, sandbox	These cannot be changed live	Next session. Show them in the UI but label them that way.

One thing to check on your Claude Code version: tightening rules with a hook always works, but a hook's allow may not override a deny rule in settings.json. Test that before you promise users that a profile can loosen restrictions.
