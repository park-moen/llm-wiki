# Protected tags | GitLab Docs

> Source: https://docs.gitlab.com/user/project/protected_tags/
> Collected: 2026-08-10
> Published: Unknown

Protected tags:

- Allow control over who has permission to create tags.
- Prevent accidental update or deletion once created.

Each rule allows you to match either:

- An individual tag name.
- Wildcards to control multiple tags at once.

To create or delete a protected tag, you must be in the Allowed to create list for that protected tag.

## Prevent tag creation with branch names

A tag and a branch with identical names can contain different commits. If your tags and branches use the same names, users running `git checkout` commands might check out the tag `qa` when they instead meant to check out the branch `qa`. As an added security measure, avoid creating tags with the same name as branches. Confusing the two could lead to potential security or operational issues.

## Run pipelines on protected tags

The permissions to create protected tags define if a user can:

- Initiate and run CI/CD pipelines.
- Execute actions on jobs associated with these tags.

These permissions ensure that only authorized users can trigger and manage CI/CD processes for protected tags.

Protected tags can only be deleted by using GitLab either from the UI or API. These protections prevent you from accidentally deleting a tag through local Git commands or third-party Git clients.
