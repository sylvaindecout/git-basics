# Merge strategies

## Context

```mermaid
gitGraph
  commit id: "A"
  commit id: "B"
  branch feature/1
  commit id: "F1-1"
  checkout main
  branch feature/2
  checkout feature/2
  commit id: "F2-1"
  commit id: "F2-2"
```

You want to merge `feature/2`, then `feature/1`. Easy, but there are multiple ways you can do it.

## Options

### Merge commit

`git merge --no-ff`

```mermaid
gitGraph
  commit id: "A"
  commit id: "B"
  branch feature/1
  commit id: "F1-1"
  checkout main
  branch feature/2
  commit id: "F2-1"
  commit id: "F2-2"
  checkout main
  merge feature/2
  merge feature/1
```

However, after a while, your Git history is likely to turn into a subway map:

```mermaid
gitGraph
  commit
  branch feature/1
  commit
  commit
  checkout main
  branch feature/2
  commit
  commit
  commit
  checkout main
  commit
  branch feature/3
  commit
  commit
  checkout main
  branch feature/4
  commit
  commit
  checkout main
  commit
  branch feature/5
  commit
  commit
  checkout main
  merge feature/1
  merge feature/3
  merge feature/2
  merge feature/4
  merge feature/5
```

Now why would that be a problem?

* Solving conflicts becomes extra hard when you fail to picture what is going on. How can you check everything went fine if you don't know what fine is?
* This is hard to navigate, even for robots. On the contrary, a linear history makes it possible to use some nice commands like `git bisect`.

### Merge commit with rebase (semi-linear history)

`git rebase && git merge --no-ff`

```mermaid
gitGraph
  commit id: "A"
  commit id: "B"
  branch feature/1
  commit id: "F1-1"
  checkout main
  merge feature/1
  branch feature/2
  commit id: "F2-1"
  commit id: "F2-2"
  checkout main
  merge feature/2
```

### Fast-forward merge (linear history)

`git rebase && git merge`

```mermaid
gitGraph
  commit id: "A"
  commit id: "B"
  commit id: "F1-1"
  commit id: "F2-1"
  commit id: "F2-2"
```

### To squash or not to squash?

Some like to keep intermediary commits, some don't. It mostly depends on team practices, especially regarding PR atomicity and code review.

## Configuration for Gitlab

<picture>
  <source media="(prefers-color-scheme: light)" srcset="img/light-gitlab_merge_config.png" />
  <img src="img/dark-gitlab_merge_config.png" alt="Gitlab settings" title="Gitlab settings" />
</picture>

```mermaid
flowchart
  start("Use Gitlab")
  choice1{"Are intermediary\n commits relevant\n after merge? <sup>1</sup>"}
  choice2{"Is dev team\n mature? <sup>2</sup>"}
  action1["Do not allow squash"]
  action2["Require squash"]
  action3["Allow squash /\n Encourage squash"]
  action4["Fast-forward merge\n (no merge commit)"]
  action5["Merge commit with\n semi-linear history"]
  start --> choice1
  choice1 -- Yes --> action1
  choice1 -- No --> action2
  choice1 -- It depends --> action3
  action1 --> action4
  action2 --> action4
  action3 --> choice2
  choice2 -- Yes --> action4
  choice2 -- No --> action5
```

## Configuration for GitHub

<picture>
  <source media="(prefers-color-scheme: light)" srcset="img/light-github_merge_config.png" />
  <img src="img/dark-github_merge_config.png" alt="GitHub settings" title="GitHub settings" />
</picture>

```mermaid
flowchart
  start("Use GitHub")
  choice1{"Is dev team\n mature? <sup>2</sup>"}
  choice2{"Are intermediary\n commits relevant\n after merge? <sup>1</sup>"}
  action1["Allow merge commits"]
  action2["Allow squash merging"]
  action3["Allow rebase merging"]
  action4["Allow squash merging\n +\n Allow rebase merging"]
  start --> choice1
  choice1 -- Yes --> choice2
  choice1 -- No --> action1
  choice2 -- Yes --> action3
  choice2 -- No --> action2
  choice2 -- It depends --> action4
```

<sup>1</sup> Is PR split into commits supposed to help people review it? Or is it supposed to remain relevant even after the branch has been merged? For instance, do you have commits like "take PR comments into account"?

<sup>2</sup> If the team is mature enough, you can let developers choose whether they need to squash or not. But if it is not, you may want to avoid this additional source of mistakes.
