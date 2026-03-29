---
name: exceed-login
description: "Login to your Exceed instance to use the admin API MCP tools"
user_invocable: true
---

# Exceed Login

Authenticate with your Exceed instance. Opens your browser for OAuth authorization.

## Steps

1. **Get the Exceed domain** from the plugin config:
   ```bash
   echo "${EXCEED_DOMAIN:-not configured}"
   ```
   If "not configured", tell the user to run `/exceed-setup` first.

2. **Open the browser** for OAuth authorization. Run this command:
   ```bash
   open "${EXCEED_DOMAIN}/oauth2/authorize?client_id=exceed-code-mcp&redirect_uri=urn%3Aietf%3Awg%3Aoauth%3A2.0%3Aoob&response_type=code&scope=ai_agent+admin+super_admin"
   ```

3. **Tell the user**: "I've opened your browser to the Exceed authorization page. Please log in and click 'Authorize'. Exceed will show you an authorization code — copy it and paste it here."

4. **Wait for the user** to paste the authorization code.

5. **Exchange the code for a token**:
   ```bash
   curl -s -X POST "${EXCEED_DOMAIN}/oauth2/token" \
     -d "grant_type=authorization_code" \
     -d "code=THE_CODE_FROM_USER" \
     -d "client_id=exceed-code-mcp" \
     -d "redirect_uri=urn:ietf:wg:oauth:2.0:oob"
   ```
   Parse the JSON response to extract `access_token`, `refresh_token`, and `expires_in`.

6. **If the exchange fails**, show the error and suggest:
   - Check if the OAuth app `exceed-code-mcp` exists on the Exceed instance
   - Verify the code was copied correctly (codes are single-use)
   - Try `/exceed-login` again

7. **If successful**, save the token. Write to `${CLAUDE_PLUGIN_DATA}/auth.json`:
   ```bash
   mkdir -p "${CLAUDE_PLUGIN_DATA}"
   cat > "${CLAUDE_PLUGIN_DATA}/auth.json" << 'AUTHEOF'
   {
     "accessToken": "THE_ACCESS_TOKEN",
     "refreshToken": "THE_REFRESH_TOKEN",
     "domain": "THE_EXCEED_DOMAIN",
     "expiresIn": EXPIRES_IN_SECONDS,
     "obtainedAt": "ISO_TIMESTAMP"
   }
   AUTHEOF
   ```

8. **Tell the user**: "Logged in successfully! Token valid for X hours. Restart Claude Code to start using the Exceed admin API tools (`exit` then `claude`). You'll have `search` and `execute` tools available."
