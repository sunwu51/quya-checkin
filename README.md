# Quya Playwright CLI check-in

Create one repository Actions secret named `QUYA_ACCOUNTS` under **Settings > Secrets and variables > Actions**. Its value is a JSON array:

```json
[
  {
    "username": "user1@example.com",
    "password": "password1"
  },
  {
    "username": "user2@example.com",
    "password": "password2"
  }
]
```

Add or remove objects in this Secret to change the account count; the workflow does not need to be edited. The JSON must contain at least one account, and every account must have non-empty string values for `username` and `password`.

The workflow runs every day at 09:00 Asia/Shanghai and can also be started manually from the Actions page. Each account gets a fresh headless browser session. Screenshots are uploaded as a GitHub Actions artifact and retained for seven days.

The check-in schedule uses this UTC Unix-day calculation:

```text
floor(current Unix timestamp / 86400) % 8
```

When the remainder is `1`, the workflow deliberately skips check-in. It still logs in, fully loads the points page, reads `span#balance`, takes a screenshot, appends one line per account to `log.txt`, and pushes the updated log. On the other seven days it checks in normally.

The balance log uses this format:

```text
2026-08-02 account-1 123
2026-08-02 account-2 456
2026-08-02 account-3 789
```

The account labels deliberately omit email addresses because the repository is public. The skip-day log is committed and pushed only when all configured accounts complete successfully. Extracted usernames and passwords are explicitly masked in the Actions log.

The workflow treats an already-disabled `今日已签到` button as success, so manually rerunning it on the same day is safe.

After login, the workflow briefly opens the dashboard's `积分中心` link before navigating to the public points URL. That link sets the `www.quya.org` SSO cookie; going straight from the dashboard to `www.quya.org/points` returns HTTP 401.

The workflow needs **Settings > Actions > General > Workflow permissions > Read and write permissions**. A protected default branch must also allow pushes from `github-actions[bot]`, otherwise the skip-day commit step will fail.
