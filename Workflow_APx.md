This tutorial is best if you want or plan to ssh authenticate with several services. Aparantly, you just can't use github and gitlab at the same time. You need a config file and some other stuff. Wish someone told me.

The operating thing to change for other services is "gitlab" and "github". The emails are just labels and don't actually matter. It's just good practice to use the email associated with those services.

1. Generate SSH key pairs for both services:

-   Open a terminal on your local machine.
-   Generate an SSH key pairs for both services by running the following commands:

<pre>
ssh-keygen -t ed25519 -C "rmarroquinsolares@gmail.com" -f ~/.ssh/github
</pre>

<pre>
ssh-keygen -t ed25519 -C "russelld@cern.ch" -f ~/.ssh/gitlab
</pre>

2. Add the SSH keys to the SSH agent:

* Start the SSH agent by running:
<pre>
eval "$(ssh-agent -s)"
</pre>

* add the ssh keys to the agent:
<pre>
ssh-add ~/.ssh/github
</pre>

<pre>
ssh-add ~/.ssh/gitlab
</pre>

3. Create a `config` file to manage multiple SSH keys:

 * In the terminal, create and edit the `config` file by running:
<pre>
nano ~/.ssh/config
</pre>

  * then add the following content to the file:
<pre>
# GitHub account
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/github

# CERN GitLab account
Host gitlab.cern.ch
  HostName gitlab.cern.ch
  User git
  IdentityFile ~/.ssh/gitlab
</pre>

  * Save and close the file by pressing `Ctrl + X`, then `Y`, and then `Enter`.

4. Add the public SSH keys to your GitHub and GitLab accounts:
-   Copy the public SSH key to your clipboard by copying the results of these command.
- For github:
<pre>
cat ~/.ssh/github.pub
</pre>

* Then add the resulting ssh key [here](https://github.com/settings/ssh/new).

* similarly for gitlab:
<pre>
cat ~/.ssh/gitlab.pub
</pre>
* then find the place to add ssh keys there.

5. Finally, test the ssh connections
* For github:
<pre>
ssh -T git@github.com
</pre>

* for gitlab:
<pre>
ssh -T git@gitlab.cern.ch
</pre>

This was a pain in the ass.
