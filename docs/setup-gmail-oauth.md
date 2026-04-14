# Gmail OAuth Setup for n8n

This guide explains how to create Gmail OAuth credentials in Google Cloud and connect them to n8n.

## Prerequisites

- A Google account with access to Google Cloud Console
- A running n8n instance at `http://178.104.202.207:5678`
- Access to the n8n admin interface

## 1. Create a Google Cloud project

1. Open [Google Cloud Console](https://console.cloud.google.com/).
2. Click the project selector in the top navigation bar.
3. Select **New Project**.
4. Enter a project name such as `n8n-email-assistant`.
5. Click **Create** and switch to the new project.

## 2. Enable the Gmail API

1. In Google Cloud Console, open **APIs & Services**.
2. Click **Library**.
3. Search for **Gmail API**.
4. Open the Gmail API page and click **Enable**.

## 3. Configure the OAuth consent screen

1. Go to **APIs & Services** -> **OAuth consent screen**.
2. Choose **External** as the user type.
3. Click **Create**.
4. Fill in the required application details.
5. Add the following scopes:
   - `https://www.googleapis.com/auth/gmail.readonly`
   - `https://www.googleapis.com/auth/gmail.send`
6. Add test users if your app is still in testing mode.
7. Save the consent screen configuration.

## 4. Create an OAuth 2.0 Client ID

1. Open **APIs & Services** -> **Credentials**.
2. Click **Create Credentials**.
3. Select **OAuth client ID**.
4. Choose **Web application** as the application type.
5. Set a descriptive name such as `n8n-gmail-oauth`.

## 5. Add the authorized redirect URI

Add this URI to **Authorized redirect URIs**:

```text
http://178.104.202.207:5678/rest/oauth2-credential/callback
```

Then click **Create**.

## 6. Copy the Client ID and Client Secret

After Google creates the client, copy:

- Client ID
- Client Secret

Keep both values available for the n8n credential setup.

## 7. Create the Gmail credential in n8n

1. Open n8n in your browser.
2. Go to **Settings** -> **Credentials**.
3. Click **New**.
4. Select **Gmail OAuth2 API**.
5. Paste the **Client ID** and **Client Secret** from Google Cloud.
6. Save the credential.
7. Click **Connect**.

## 8. Authorize the connection

1. n8n will open the Google authorization flow in your browser.
2. Sign in with the Gmail account that the workflow should monitor.
3. Review the requested permissions.
4. Approve access.
5. Return to n8n and confirm that the credential status shows as connected.

## Notes

- If you deploy n8n on a different domain or port later, you must add the new redirect URI in Google Cloud.
- If Google shows an unverified app warning during testing, continue only if you trust the project and have configured the consent screen correctly.
- Gmail OAuth credentials should be stored only in the n8n credentials UI, not in `.env`.
