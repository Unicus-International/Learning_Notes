## Persistent use of ssh-keys
Depending on setup and environment there are times when simply using the vs_code git interface is not enough to connect to remote repositories through ssh.  
The last time I experienced this was with vs_code using Windows Subsystem for Linux and connecting to both github and gitlab with vscode. It ended up having issues with connecting to gitlab with ssh during git actions.  
### Per repo settings
Each git repository can set ssh settings in the git config. While the terminal is in the project you can set the repository ssh id with:  
```
git config core.sshCommand "ssh -i ~/.ssh/id_ed25519"
```
Add the ```--global``` flag to set for every repository. This is inadvisable if you use both github and gitlab on the same computer with different ssh keys.  
Be advised that this will only set the ssh option for git actions. Running ```ssh -T git@github.com``` to verify your connection will not connect properly.

### Per host setting
Git will grab the correct correct ssh config if the config file is set. This can be set per host, and is a per user on the computer thing. Remember to restrict the folder and files if you share a computer with someone.  
```
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
chmod 600 ~/.ssh/config
```
This will work for git, but will behave as a global setting per repo provider. It can differentiate between github and gitlab as well as private gitlab hosts, but will not differentiate on repositories with the same provider if you have a reason to use different keys there.
```
Host github.com
 User git
 IdentityFile ~/.ssh/id_github
 IdentitiesOnly yes

Host gitlab.company.com
 User git
 IdentityFile ~/.ssh/id_gitlab_work
 IdentitiesOnly yes

Host gitlab.private.com
 User git
 IdentityFile ~/.ssh/id_gitlab
 IdentitiesOnly yes
```
