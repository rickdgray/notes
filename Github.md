---
title: Github
lastmod: 2023-05-28
---
# Github
## Verified Commits from Linux
1. Install git and pass
```
sudo apt install git pass
```
2. Install [Git Credential Manager Core (GCM Core)](https://docs.github.com/en/get-started/getting-started-with-git/caching-your-github-credentials-in-git?platform=linux).
   1. Download the latest [.deb](https://github.com/microsoft/Git-Credential-Manager-Core/releases/latest) file.
   2. Install the .deb file.
   ```
   sudo dpkg -i <path-to-package>
   ```
   3. Initialize GCM Core.
   ```
   git-credential-manager configure
   ```
3. Create a new identity. The "Real Name" you choose here will be your "user id" in step 5. If you decide to set a passphrase, it will need to be entered for every commit.
```
gpg --gen-key
```
4. Add the public key of your newly created gpg identity to your [github keys](https://github.com/settings/keys) so that github can verify your locally signed commits.
   1. Find your newly created identity.
   ```
   gpg --list-secret-keys --keyid-format=long
   ```
      In the example print out below, the gpg id is "9794C0815DD517AC".
   ```
   sec   rsa3072/9794C0815DD517AC 2022-08-23 [SC] [expires: 2024-08-22]
         C3E518D31EDC2F2055036E4C9794C0815DD517AC
   uid                 [ultimate] John Doe <jdoe@contoso.com>
   ssb   rsa3072/CA147602C62ED40C 2022-08-23 [E] [expires: 2024-08-22]
   ```
   2. Print the public key.
   ```
   gpg --armor --export 9794C0815DD517AC
   ```
   3. Copy the public key and add it to your github keys.
5. Initialize pass with the gpg identity's user id.
```
pass init "John Doe"
```
6. Tell git to use gpg identities for credential storage, to use your newly created key, and to sign your commits with the private gpg key.
```
git config --global credential.credentialStore gpg
git config --global user.signingkey 9794C0815DD517AC
git config --global commit.gpgsign true
```
7. Don't forget to set name and email in the git configuration if you haven't yet.
```
git config --global user.name "John Doe"
git config --global user.email jdoe@contoso.com
```
8. Append this line to the end of your "~/.bashrc" file.
```
export GPG_TTY=$(tty)
```
9. Now try to clone a private repo or some other privileged access action. GCM will then ask for authentication via either your browser or a [personal access token (PAT)](https://docs.github.com/en/github/authenticating-to-github/keeping-your-account-and-data-secure/creating-a-personal-access-token). Either will work.

After this initial authorization, GCM will store your authentication in the password store (pass). Now, when doing any privileged git operations, git will automatically use your gpg identity -> password store (pass) -> stored authentication. If you set up a passphrase on your gpg identity, you will only have to remember that to decrypt the gpg key. If not, it will be entirely automatic.
## Github Actions
You can impersonate the github actions bot with this name and email:
```bash
git config --global user.name "github-actions[bot]"
git config --global user.email "github-actions[bot]@users.noreply.github.com"
```