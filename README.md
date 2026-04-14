# stacked-pr-bot

A GitHub Actions workflow that automatically links stacked pull requests and prevents them from being merged out of order.

## Why stacked PRs?

Large pull requests are hard to review. Stacking lets you break a big change into small, focused PRs where each one builds on the previous. Because each PR targets the branch of the one before it, reviewers only see the diff between two adjacent steps, not the entire change at once. This makes reviews faster, more thorough, and easier to reason about.

The trade-off is bookkeeping. With a stack of PRs, you need to merge them in order (top of the stack first, working your way down), and it's easy to lose track of which PR depends on which. Someone might merge a PR before its dependencies are ready, or not realize that a PR is part of a stack at all.

ReviewBot handles that bookkeeping for you. It links the PRs together, reminds you of the merge order, and converts the root PR to a draft until all children above it are merged. You start by reviewing the root PR (which is now a draft) to get the big picture, but you don't merge it yet. Then you go to the first child PR, review it, and merge it. GitHub automatically updates the next child's base branch, so its diff stays clean and only shows its own changes. You continue working through the stack this way until all children are merged and the root PR is ready to go.

## What this does

When a pull request is opened or reopened, ReviewBot checks whether the PR's base branch has its own open pull request. If it does, the workflow:

- Comments on both the new PR and the base PR with links to each other.
- If the base PR targets the default branch (i.e. it's the root of the stack), converts it to a draft so it can't be merged prematurely.
- If the base PR is an intermediate PR in the stack, adds a comment reminding you to merge it first.

## Setup

Copy `.github/workflows/ReviewBot.yml` into your repository. That's it.

The workflow uses `GITHUB_TOKEN`, which is provided automatically by GitHub Actions. No additional secrets or configuration needed.

## How stacking works in practice

Say you have three branches forming a stack:

```
main <- feature-auth <- feature-auth-tests <- feature-auth-docs
```

Each branch has a PR targeting the one before it. When you open the PR for `feature-auth-tests` (targeting `feature-auth`), ReviewBot will:

1. Comment on the `feature-auth-tests` PR, linking to the `feature-auth` PR.
2. Comment on the `feature-auth` PR, notifying about the new child.
3. Since `feature-auth` targets `main` (the default branch), convert it to a draft.

When you then open the PR for `feature-auth-docs` (targeting `feature-auth-tests`), ReviewBot will link them and remind you to merge `feature-auth-tests` first.

## Limitations

- The workflow only runs on `opened` and `reopened` events. If you change a PR's base branch after opening it, ReviewBot won't pick that up.
- ReviewBot adds comments but does not remove or update them. If a child PR is closed, the comments on the parent will remain.
- The draft conversion only applies to the root PR (the one targeting the default branch). Intermediate PRs in the stack are not converted to drafts.

## License

MIT
