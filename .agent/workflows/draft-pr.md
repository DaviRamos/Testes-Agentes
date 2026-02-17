---
description: PR Description Generator
---

Goal: Draft a professional Pull Request description based on recent changes. Steps:

Run git diff main (or the target branch) to see all changes made.

Summarize the changes into three sections:

What Changed: A bulleted list of technical changes.

Why: The reasoning behind the changes (infer from code or comments).

Testing: How these changes can be verified.

Format the output as a Markdown block ready to be pasted into GitHub/GitLab.
