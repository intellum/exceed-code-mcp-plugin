---
name: exceed-setup
description: "Configure your Exceed domain for the admin API plugin"
user_invocable: true
---

# Exceed Setup

Configure which Exceed instance to connect to.

## Steps

1. **Ask the user** for their Exceed domain URL. Examples:
   - `https://mycompany.exceedlms.com`
   - `https://intellum.exceed.qa-env-36.dev.intellum.com`

2. **Validate the URL** — must start with `https://` (or `http://` for local dev).

3. **Check if the Exceed instance has the OAuth app** registered:
   ```bash
   curl -s -o /dev/null -w "%{http_code}" "USER_DOMAIN/oauth2/authorize?client_id=exceed-code-mcp&response_type=code&redirect_uri=urn:ietf:wg:oauth:2.0:oob&scope=ai_agent"
   ```
   - 302 = OAuth app exists, redirect to login (good)
   - 401 = OAuth app not found (needs to be created on the Exceed side)

4. **If the OAuth app doesn't exist**, tell the user:
   "The `exceed-code-mcp` OAuth app hasn't been registered on this Exceed instance yet. Contact your Exceed admin to run the setup task, or if you have Rails console access, run:
   ```ruby
   app = Doorkeeper::Application.find_or_initialize_by(name: 'Exceed Code MCP')
   app.assign_attributes(uid: 'exceed-code-mcp', secret: Doorkeeper::OAuth::Helpers::UniqueToken.generate, redirect_uri: "https://localhost\nurn:ietf:wg:oauth:2.0:oob", scopes: 'ai_agent admin super_admin', confidential: false)
   app.save!
   ```"

5. **Save the configuration** — tell the user to update their plugin config:
   "Run this to update your plugin configuration:
   ```
   /plugin configure exceed-admin@intellum
   ```
   Set `exceed_domain` to `USER_DOMAIN`."

6. **Tell the user**: "Setup complete! Now run `/exceed-login` to authenticate."
