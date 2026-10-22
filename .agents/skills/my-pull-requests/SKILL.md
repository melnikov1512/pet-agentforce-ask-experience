---
name: my-pull-requests
description: 'List my pull requests in the current repository'
---

Determine the current repository and authenticated GitHub user using available GitHub tools or the gh CLI, then list pull requests assigned to that user.

Describe the purpose and details of each pull request.

If a PR is waiting for someone to review, highlight that in the response.

If there were any check failures on the PR, describe them and suggest possible fixes.

Report the review status. Do not request reviews or send messages unless the user explicitly asks.
