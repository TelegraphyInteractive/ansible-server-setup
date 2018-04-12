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
  ... to come
