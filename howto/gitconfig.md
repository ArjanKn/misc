# gitconfig

see blog post [core git dev config](https://blog.gitbutler.com/how-git-core-devs-configure-git/)

[xkcd git](https://xkcd.com/1597/)

```
# clearly makes git better

[column]
    ui = auto
[branch]
    sort = -committerdate
[tag]
    sort = version:refname
[init]
    defaultBranch = main
[diff]
    algorithm = histogram
    colorMoved = plain
    mnemonicPrefix = true
    renames = true
[push]
    default = simple
    autoSetupRemote = true
    followTags = true
[fetch]
    prune = true
    pruneTags = true
    all = true

# why the hell not?

[help]
    autocorrect = prompt
[commit]
    verbose = true
[rerere]
    enabled = true
    autoupdate = true
[core]
    excludesfile = ~/.gitignore
[rebase]
    autoSquash = true
    autoStash = true
    updateRefs = true
[apply]
    # Remove trailing whitespaces
    whitespace = fix

# a matter of taste

[core]
    # fsmonitor = true
    # untrackedCache = true
[merge]
    # (just 'diff3' if git version < 2.3)
    # conflictstyle = zdiff3 
[pull]
    # rebase = true
[log]
     date = iso-local
[format]
    pretty = fuller
[alias]
    hist = log --pretty=format:'%h %ad | %s%d [%an]' --graph --date=short
```


