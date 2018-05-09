# ansible-server-setup
This takes a bare-metal server and whips it into shape for a mysql database
and/or nginx server.

It:
* locks down the server against ssh login attempts
  - logs in as root
  - creates a new deploy user with sudo privileges
  - prevents root login
  - disables password login
  - enables login with public key authentication for the deploy user
  - changes the ssh port
  - locks-out the standard ssh port using iptables
* installs rbenv ruby for the deploy user
* installs and configures nginx
* installs and configures passenger
* installs and cenfigures mysql
* installs an ssl certificate using EFF letsencrypt and certbot

## Usage

1. Clone the repository. You want to include submodules. The following clones
to a subdirectory named "ansible":
```
git clone --recurse-submodules git@github.com:TelegraphyInteractive/ansible-server-setup.git ansible
```
From here on we refer to this `ansible` directory,
where we cloned the project, as "*the root*".

1. Switch to *the root* to complete all of the following, e.g. `cd ansible`.
1. Make yourself an inventory file, e.g. `staging`.
(You give it the name you want.)
Sample contests of `ansible/staging`:
```
[webservers]
staging.mysite.com

[dbservers]
staging.mysite.com
```
1. Make yourself a vault password:
   1. Generate the password in a password safe
   2. Paste the password into file `.vault_pass` of *the root*, e.g.
      `echo "AzhUyw7q1xn0gKPg8hyXTQfGA8FtUKqwvMlmUm" >.vault_pass`
   3. The repository is already set-up to ignore any file named, "`.vault_pass`"
      for `git`. (See `cat .gitignore`).
      If you use a different filename, or different source code repository,
      ensure that the file is not stored in the source code repository.
      `echo ".vault_pass" >>.gitignore`
1. Make yourself an `ansible.cfg`.
Sample contents of `ansible/ansible.cfg`:
```
[defaults]
inventory = staging
vault_password_file = .vault_pass
```

_work in progress_
