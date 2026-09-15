# Create a tag from the command line | GitLab Docs

> Source: https://docs.gitlab.com/user/project/repository/tags/#from-the-command-line
> Collected: 2026-08-10
> Published: Unknown

To create either a lightweight or annotated tag from the command line, and push it upstream:

1. To create a lightweight tag, run the command `git tag TAG_NAME`, changing `TAG_NAME` to your desired tag name.
2. To create an annotated tag, run one of the versions of `git tag` from the command line:

```bash
# In this short version, the annotated tag's name is "v1.0",
# and the message is "Version 1.0".
git tag -a v1.0 -m "Version 1.0"

# Use this version to write a longer tag message
# for annotated tag "v1.0" in your text editor.
git tag -a v1.0
```

3. Push your tags upstream with `git push origin --tags`.

Tags can also be created from the GitLab UI. For Create from, select an existing branch name, tag, or commit SHA. Add a Message to create an annotated tag, or leave blank to create a lightweight tag.
