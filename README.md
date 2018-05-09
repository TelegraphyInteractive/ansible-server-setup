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
1. Make yourself a vault of secret settings:
    `ansible-vault create vars/vault.yml`
    The playbook expects this file name. (See the `include` directive at the
    top of `site.yml`) The contents of the file are something like:
    ```
    ---
      # initial root password whose login will be disabled
      vault_root_password: kO9pkIkmO6uBAopFp7OQGBElSYn5lPFAnfaCSG

      # locked down ssh user, port
      vault_devops_ssh_user: devious
      vault_devops_ssh_port: 22898

      # These for the mysql role. See the doc's for that
      vault_mysql_root_password: t36WlQAyr7j508OIi9p4wZDOtbkneYBIcmqLiI
      vault_mysql_databases:
      - name: appdb
        encoding: utf8mb4
        collation: utf8mb4_general_ci
      mysql_users:
      - name: "{{ vault_devops_ssh_user }}"
        password: "{{ vault_mysql_root_password }}"
        priv: "{{ vault_devops_ssh_user }}.*:ALL"
      - name: rails
        password: 96xK7Q57bna3yH6dIDoQhOUkYMsbEeZbzTxKcg
        priv: "rails.*:INSERT,ALTER,CREATE,DELETE,DROP,INDEX,SELECT,UPDATE"
      - name: backup
        password: IUeIILc8VWQTUvgIgfYBd6XPltbrFdZptKXGRN
        priv: "backup.*:SELECT,LOCK TABLES"

      # for rails
      vault_rails_master_key: P8lqsUJKUOGmogNrjKDEyMXNx4seubJ2ipTuWK

      # for nginx role, site template
      site_root: "app_name/current/public"

      vault_letsencrypt_account_email: "admin@mysite.com"
    ```
    Note: This can be improved. The database settings should be opinionated in
    `vars/main.yml` and then unstructured - just `name: value` pairs
    like the rest.
1. Set-up your server. If you haven't already:
    1. create the server using the root password
    1. put the server ip address in your `/etc/hosts`
       (assuming the server isn't in DNS)
    1. copy your public key to the server
    1. accept the server into your known-hosts
1. Run the play: `ansible-playbook site.yml`
