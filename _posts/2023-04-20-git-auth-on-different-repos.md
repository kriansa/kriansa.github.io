---
layout: post
title: Authenticate to Git with different users on different repositories
date: 2023-04-20 22:00:00 BRT
tags: git, github, ssh
categories: articles
---

Sometimes you might need to use different Git repositories on your computer, but authenticating to
separate users, such as your personal and your work user.

Fortunately, git provides us with the config `core.sshCommand`, which allows us to define a
different SSH command for a given repo (or a subset of repos).

## For a single repository
In most cases, you will be using a standard authentication for most of your repositories, and having
a few ones that should connect using a separate key. For that case, you'll be best served by
configuring the exceptions one by one.

1. First and foremost, you need to have both SSH authentication keys correctly located at your
   `~/.ssh` directory and with the correct permissions (0600).
2. Then, on the repo(s) you want to diverge from the default SSH key list (determined by running
   `ssh-add -l`) use the following command:
   ```sh
	 git config core.sshCommand "ssh -o IdentitiesOnly=yes -i ~/.ssh/your_separate_key.pub"
	 ```
3. Now try and run a git command such as `git pull`

A small caveat here is that in case your repository have submodules that you also want to
authenticate using that same credential, then you will need to run that command for each submodule
you have:

```sh
git submodule foreach git config core.sshCommand "ssh -o IdentitiesOnly=yes -i ~/.ssh/your_separate_key.pub"
```
## For multiple repositories
In other cases, you can have multiple folders, each one belonging to a different context of work
(clients, projects, etc) and you may well have multiple users yourself to manage that work. Git also
got you covered due to its global configuration functionality.

1. Get all the keys you will use and place them with the correct permissions (0600) at `~/.ssh`
2. Now, we will add a global config entry that override certain configs depending where your
   repository is located at. To do so, edit your `~/.gitconfig` file and add the following content
   at **the end of it**:

```gitconfig
[includeIf "gitdir:~/Projects/"]
  path = ~/.gitconfig-myprojects
```

**PS:** Make sure the path ends with a trailing slash, otherwise git will interpret it differently.
Adding this section to the end of the file also guarantees that these configurations will take
precedence over the ones defined above.

You might guess that this simply adds a new override file for all projects under that folder, and
then you're able to set a different configuration. Now all you need is to create that file and add
the following content:

```gitconfig
[core]
  sshCommand = "ssh -o IdentitiesOnly=yes -i ~/.ssh/your_separate_key.pub"
```

Because you can use this section to override any git config for that directory, it might also be
convenient to change your `name` and `email` as well.
