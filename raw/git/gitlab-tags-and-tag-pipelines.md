# Tags | GitLab Docs

> Source: https://docs.gitlab.com/user/project/repository/tags/
> Collected: 2026-08-10
> Published: Unknown

In Git, a tag marks an important point in a repository’s history. Git supports two types of tags:

- Lightweight tags point to specific commits, and contain no other information. Also known as soft tags. Create or remove them as needed.
- Annotated tags contain metadata, can be signed for verification purposes, and can’t be changed.

The creation or deletion of a tag can be used as a trigger for automation, including:

- Using a webhook to automate actions like Slack notifications.
- Signaling a repository mirror to update.
- Running a CI/CD pipeline with `if: $CI_COMMIT_TAG`.

When you create a release, GitLab also creates a tag to mark the release point. Many projects combine an annotated release tag with a stable branch. Consider setting deployment or release tags automatically.

## Trigger pipelines from a tag

GitLab CI/CD provides a predefined variable, `CI_COMMIT_TAG`, to identify tags in your pipeline configurations. You can use this variable in job rules and workflow rules to test if a pipeline was triggered by a tag.

By default, if your CI/CD jobs don’t have specific rules in place, they are included in a tag pipeline for any newly created tag. Tag pipelines are only created when a tag targets a commit.

In your `.gitlab-ci.yml` file for the CI/CD pipeline configuration of your project, you can use the `CI_COMMIT_TAG` variable to control pipelines for new tags:

- At the job level with `rules:if`.
- At the pipeline level with the `workflow` keyword.
