---
title: "What I want in GitHub Security Advisories"
date: 2026-10-21T13:24:28+00:00
---

Security Advisories are a GitHub repository feature that allows users to submit vulnerability reports to repository owners in a private manner. Owners can then review the inbox of submissions, and decide whether the report is worth their time or not.

LLM security research capabilities in 2026 have made this feature even more important and I'd like to point out a few omissions that, if addressed, would immensely improve the user experience for both reporters and maintainers.

In no particular order, I found my suggestions are improvements in either inbox triaging, or in the remediation workflow (currently via private fork pull requests):

- [Inbox triaging improvements](#inbox-triaging-improvements)
  - [Customizable team assignment](#customizable-team-assignment)
  - [Organization-wide inbox](#organization-wide-inbox)
  - [More visible notifications](#more-visible-notifications)
  - [Transfer advisories to other repositories](#transfer-advisories-to-other-repositories)
- [Remediation workflow improvements](#remediation-workflow-improvements)
  - [Functional CI](#functional-ci)
  - [Customizable merge message and mode](#customizable-merge-message-and-mode)
  - [Private branches](#private-branches)
- [Conclusion](#conclusion)

## Inbox triaging improvements

Incoming reports that need to be triaged have very low visibility in any account with a minimum amount of activity. These notifications are difficult to find and easy to miss among the noise, which introduces a failure point in the first step of the pipeline: seeing the report to begin with! Projects have come up with their own workarounds (automation bots, email-only workflows) to mitigate these gaps.

### Customizable team assignment

By default, advisories are assigned to the repository _owners_. These are not necessarily the maintainers handling day-to-day operations. For example, in the conda organization, the steering council team is the owner of all repositories, and different subteams are delegated to maintain them. Subteams do have admin access, but they are not owners, so they _don't even see_ advisories unless someone from the steering council sees the advisory and adds them by hand!

Can we allow some customization here? Either something like `CODEOWNERS` or a field in the Settings tab?

### Organization-wide inbox

There's no such thing, and it would be _very_ useful for anyone maintaining more than one project, or teams handling security across several projects. If anyone wants to check the security advisories of all their repositories, they'll need to use the GitHub API to get it programmatically:

```
gh api orgs/{org}/security-advisories
```

> 🌟 Bonus! 🌟 Feel free to grab [my org-wide advisories report script](https://gist.github.com/jaimergp/031f30b8f83fb2f2079b56b9e2f3b47d) to generate static, self-contained HTML documents aggregating advisories across repositories.

### More visible notifications

The [Notifications](https://github.com/notifications) page in GitHub has also seen better days when it comes to maintenance, but that would be a different blog post. I'll just mention the bits that concern security advisories: it is completely broken!

The [`is:security-advisory` filter](https://github.com/notifications?source=search_bar&query=is%3Arepository-advisory) only reveals advisories that have been accepted, but there's no trace of all the untriaged ones! [`is:repository-vulnerability-alert`](https://github.com/notifications?source=search_bar&query=is%3Arepository-vulnerability-alert) only aggregates alerts sent by Dependabot.

The only reliable way to get notifications for those is via email (tip: create a subject filter querying `GHSA` to highlight them), but then you'll get ALL notifications (advisory or not) via email.

So, this one is two-fold:

* Can we have a way to list and find all security advisories across all repositories in the Notifications page?
* Can we have a separate toggle to enable email notifications for security advisories _only_? Bonus points if they come from a different address so they are super easy to flag.

### Transfer advisories to other repositories

Sometimes users submit their reports to one repository, when in reality the vulnerability is found in a different part of the stack; e.g. the report was sent to the frontend app, but the actual vulnerability concerns a backend component. It would be very useful to have an advisory transfer feature, just like we can already do with issues, so there's no need for the user (or the maintainers) to resubmit them by hand.

## Remediation workflow improvements

Imagine you've managed to review your inbox, comb through the duplicates, non-issues and sloppy reports, and finally found something you need to work on. You accept the advisory, create a new private fork, clone it, start a PR... and wait, how can I run the tests?

### Functional CI

If the maintainers decide to accept the advisory, a private ephemeral fork can be created to start one or more PRs containing the fix(es). However, no CI service runs in these PRs, GitHub Actions, or otherwise. The maintainers need to run the CI on their own, be it locally or in yet another private fork they created on their own account.

I understand that running CI on private repositories is a paid service with only a limited amount of minutes on the free tier, but let the maintainers decide whether they _want_ to run it to begin with. Right now it's just not possible. It would be even better if these private fork CI minutes didn't count towards the quota, but maybe that's a vector for abuse. Maybe only for CVE-assigned advisories? Ah, wait, [there's a two-month queue for that](https://github.blog/security/supply-chain-security/inside-the-advisory-database-and-what-happens-when-vulnerability-volume-breaks-records/).

### Customizable merge message and mode

If you do decide to merge PRs from a private fork, the target branch on the public repository receives a mysterious "Merged from pull request" default message that can only be amended after the fact. Also, it is always a plain merge commit; no option to squash like in regular PRs.

Can we please have the same UI here? And while we are at it, can we actually make merges work? In some repositories it is almost a lottery.

### Private branches

Do we really need a whole new fork to handle an advisory? That initial clone can be costly in big repositories so... what about an option to operate on a private branch? I am aware this feature doesn't really exist, but it would make it way simpler, and, in general, it would also benefit other types of work that need to start as a non-public branch.

## Conclusion

It looks like GitHub is currently allocating some resources to security advisories, given the early October updates in the changelog. [^1] I'm happy to see this moving forward! Consider this post my opinionated wish list. Hopefully I will keep coming back to it just to update it saying _"this is now available!"_ over the next few months.

> Quick acknowledgement: the idea for this post and most of its content come from a very nice conversation I had with [Seth Larson](https://sethmlarson.dev) in New York a few months ago. Thanks for the inspiration and encouragement, Seth!

[^1]: See these blog posts: [Structured forms for private vulnerability reports](https://github.blog/changelog/2026-10-01-structured-forms-for-private-vulnerability-reports/), [Rate limits for private vulnerability reports](https://github.blog/changelog/2026-10-01-rate-limits-for-private-vulnerability-reports/), [Confidential comments on repository security advisories](https://github.blog/changelog/2026-10-02-confidential-comments-on-repository-security-advisories/), [Repository security advisory comments API in public preview](https://github.blog/changelog/2026-10-02-repository-security-advisory-comments-api-in-public-preview/), [New fields for SecurityAdvisory GraphQL API](https://github.blog/changelog/2026-10-02-new-fields-for-securityadvisory-graphql-api/).
