# git-upstream

Turn a fork that has periodic merges of upstream into easily upstreamable rebaseable linear history (git-imerge rebase through merges)

## Context

They made a fork, they periodically merge upstream changes, but they can't contribute back, as it's hard to extract all patches in a way that would cleanly apply to upstream. Plus once you merged at least once, interactive rebase goes out of the window. Plus if you don't merge very very often, then merges by internal git tools become harder because they lack context necessary for correct minimal conflict resolution. `git-imerge` is an awesome tool that tremendously helps with any merges or rebases when both branches contain multiple commits, but if you try to do `git imerge rebase` of a branch that has merge commits, it will politely decline, stating that --goal='rebase' is not yet supported for branches that include merges.

This project is basically a tool for turning fork history with periodic merges into linear history on top of the upstream.

## How does it work

Below is the gist of the mental model for it. 

### find commits

```
git log origin/main..my_fork_with_periodic_merges
```

### mark commits

1. mark latest common commit before any merges and mark as root point (Ar), its first commit mark A1,A2... upto next merge commit
2. mark merge point origins (Bo, Co, ...)
3. mark merge point commits (Bm, Cm, ...) and commits after them B1,B2, ..., C1, C2, ...

```
Ar -> T            # parent of A1    
A1                 # oldest commit that is returned by the command above    
A2                 # A1's child    
A3                 # A2's child    
Bm (merge A3, Bo)  # merge-commit in the fork that merges origin/main    
B1                 # Bm's child    
B2                 # B1's child    
Cm (merge B2, Co)  # and so on...    
C1    
C2    
C3    
C4
```

### do the magic

    git switch Ar -C T

    git cherry-pick A1,A2,A3
    git imerge rebase Bo
    git imerge finish
    git switch -C A-over-Bo-main
    git switch Bm -C A-over-Bo-full
    git reset --soft A-over-Bo-main
    git commit -m "Additional changes from Bm: $(extract-commit-message Bm)"
    git checkout T
    git reset --hard A-over-Bo-full
    
    git cherry-pick B1, B2
    git imerge rebase Co
    git imerge finish
    git switch -C B-over-Co-main
    git switch Cm -C B-over-Co-full
    git reset --soft B-over-Co-main
    git commit -m "Additional changes from Cm: $(extract-commit-message Cm)"
    git checkout T
    git reset --hard B-over-Co-full

    git cherry-pick C1, C2, C3, C4

## How to use it

Put git-upstream somewhere on your path and then

    git checkout my_branch
    git upstream origin/master

At this point it will create a branch named `temp` and will start working its magic. When it encounters a conflict (because we are using imerge, they are tiny and exactly at the point where conflicting ideas of the code were introduced), it will pause and ask you to resolve the conflict and press Enter to continue. After all conflicts are resolved it will say that it's done, and you will be left with the `temp` branch that would contain desired history.

## Plans for improvements

It is reasonably easy to add support for passing not just the target, but also the source commit, so that if you already ran git-upstream, and then new commits appeared in your fork you can jus process the new part.

## License

MIT
