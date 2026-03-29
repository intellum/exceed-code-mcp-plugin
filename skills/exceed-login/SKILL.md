---
name: exceed-login
description: "Login to your Exceed instance to use the admin API tools"
user_invocable: true
---

# Exceed Login

Authenticate with your Exceed instance via OAuth.

## Steps

1. **Read the configured domain** from the plugin option. Run:
   ```bash
   echo "${CLAUDE_PLUGIN_OPTION_exceed_domain:-}"
   ```
   If empty, ask the user: "What is your Exceed domain URL? (e.g., https://mycompany.exceedlms.com)"

2. **Open the browser** for OAuth authorization:
   ```bash
   open "DOMAIN/oauth2/authorize?client_id=exceed-code-mcp&redirect_uri=urn%3Aietf%3Awg%3Aoauth%3A2.0%3Aoob&response_type=code&scope=ai_agent+admin+super_admin"
   ```
   Replace DOMAIN with the actual domain from step 1.

3. Tell the user: "I've opened your browser to Exceed. Log in and click Authorize. Then copy the code shown on the page and paste it here."

4. **Wait for the user** to paste the authorization code.

5. **Exchange the code for a token**:
   ```bash
   curl -s -X POST "DOMAIN/oauth2/token" \
     -d "grant_type=authorization_code" \
     -d "code=USER_CODE" \
     -d "client_id=exceed-code-mcp" \
     -d "redirect_uri=urn:ietf:wg:oauth:2.0:oob"
   ```
   Parse the JSON response. Extract `access_token`, `refresh_token`, `expires_in`.

6. If the exchange **fails**, tell the user the error and suggest running `/exceed-login` again (codes are single-use).

7. If **successful**, save the auth. Run:
   ```bash
   mkdir -p "${CLAUDE_PLUGIN_DATA:-.}"
   cat > "${CLAUDE_PLUGIN_DATA:-.}/auth.json" << 'EOF'
   {"accessToken":"ACCESS_TOKEN","refreshToken":"REFRESH_TOKEN","domain":"DOMAIN","expiresIn":EXPIRES_IN}
   EOF
   ```
   Replace the placeholder values with the actual token data.

8. Tell the user: "Logged in to DOMAIN. Token valid for N hours. Restart Claude Code to start using the Exceed admin API tools."
