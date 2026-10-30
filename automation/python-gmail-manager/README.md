# Python Gmail Manager (CT106)

**Status:** In progress as of October 30, 2026.

Debian 13 LXC container on HomeProx3 running a Python-based Gmail classifier for two authorized accounts. The project uses Gmail API access, SQLite processed-message/audit logging, classification rules and guarded execution.

Latest confirmed batch configuration: **10 messages per account, maximum 20 per scheduled run**. This confirms configuration, not completion of inbox processing.

Safety rules: preserve read/unread, starred and important state; skip protected messages and Spam/Trash; label retained mail and archive as appropriate; label unwanted messages **Delete** without moving them to Trash; avoid duplicate labels; audit actions and handle retries.

**Next:** Continue processing, inspect logs and exception cases, verify coverage and eventual completion; do not claim inboxes empty without a fresh count.
