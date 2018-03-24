
`ansible --inventory hosts all -u root -m ping`

- inventory file is `hosts` in current directory
- run against the hosts using the `root` user
- command is to `ping`. `ping` is a "module name" `-m` specifies a module

The `ansible.cfg` file captures ansible configuration settings.
By placing

```
[defaults]
inventory = hosts
```

in that file, it's possible to leave-off the `--inventory` command line flag.

`ansible all -u root -m ping`

## Handlers

Handlers listen to a topic. Tasks notify the topic.
All notified handlers run once when flushed.
If a handler has been notified multiple times, it will run only once.

Notify occurs if the notifying task runs successfully and causes a change.
Otherwise, there is no notify.

Running all notified handlers is referred to as "flushing."
Handlers are flushed
- at the end of `pre_tasks`, `tasks`, and `post_tasks` sections within
a role or a play.
- when flushed manually with `meta: flush-handlers`

```
handlers:
    - name: restart memcached
      service: name=memcached state=restarted
      listen: "restart web services"
    - name: restart apache
      service: name=apache state=restarted
      listen: "restart web services"

tasks:
    - name: restart everything
      command: echo "this task will restart the web services"
      notify: "restart web services"
```

## Running playbooks

`ansible-playbook playbook.yml`

You can check syntax and learn which hosts will be affected with,
`ansible-playbook --check-syntax --list-hosts playbook.yml`

You can get more verbose output by adding the `--verbose` tag.

## Pieces and parts

The `import_<type> <part.yml>` directive brings in and parses imported
content at the very beginning as "static" content.
- Variables from inventory sources can't be used.
- Loops will not work

The `include_<type> <part.yml>` directive is parsed when execution reaches
the statement as "dynamic" content.
- You can't notify or jump to tasks within the dynamic content.
- The included tasks or tags won't be listed for the play

`import_playbook` runs the named playbook file at the point of the import.

`import_tasks` runs the named tasks file at the point of the import.

## Roles
Are groups of ansible tasks, handlers, templates, and vars encapsulated into
a module.

They exist as named subdirectories of the `roles` directory.

## Playbook order of execution
the order of execution for your playbook is as follows:

    1. Run `pre_tasks` defined in the play.
       - Runs tasks and roles included or imported by those tasks
         (`include_tasks`, `include_roles`, `import_tasks`, `import_roles`)
    1. Flush handlers
    1. Execute each role listed in `roles` in turn.
       - Any role dependencies defined in the `meta/main.yml` of those roles
       will run first, subject to tag filtering and conditionals.
       - The `roles` directive is equivalent to writing,
       `import_role ...` at the beginning of the `tasks`.
    1. Run `tasks` defined in the play.
       - Runs tasks and roles included or imported by those tasks
    1. Flush handlers
    1. Run `post_tasks` defined in the play.
       - Runs tasks and roles included or imported by those tasks
    1. Flush handlers

Note that:

```
- hosts: webservers
  roles:
    - { role: foo_app_instance, dir: '/opt/a', app_port: 5000 }
  tasks:
  # tasks here...
```

is equivalent to

```
- hosts: webservers
  tasks:
    - include_role:
         name: foo_app_instance
      vars:
        dir: '/opt/a'
        app_port: 5000
    # tasks here...
```

and that the second is the newly preferred syntax.

## Debug trick

```
- hosts: webservers
  tasks:
    - debug:
        msg: "before we run our role"
    - import_role:
        name: example
    - include_role:
        name: example
    - debug:
        msg: "after we ran our role"
```

Conditionally execute a role

```
- hosts: webservers
  tasks:
  - include_role:
      name: some_role
    when: "ansible_os_family == 'RedHat'"
```

## Directory structure

There's a full directory structure [here](
  http://docs.ansible.com/ansible/latest/playbooks_best_practices.html
)

For roles
```
site.yml
webservers.yml
fooservers.yml
roles/
   common/
     tasks/
     handlers/
     files/
     templates/
     vars/
     defaults/
     meta/
   webservers/
     tasks/
     defaults/
     meta/
```

The directories under each of `roles`:

  * tasks - contains the main list of tasks to be executed by the role.
  * handlers - contains handlers, which may be used by this role or even anywhere outside this role.
  * defaults - default variables for the role (see Variables for more information).
  * vars - other variables for the role (see Variables for more information).
  * files - contains files which can be deployed via this role.
  * templates - contains templates which can be deployed via this role.
  * meta - defines some meta data for this role.

## Variables

Come from a number of places:

  - `vars:` within the host inventory:
  ```
    hosts: webservers
    vars:
      http_port: 80
  ```
  - files in the `vars` directory of a role or playbook
  - a vault
  - the command line
  - invocation, such as `include_role`

Reference them with `"{{ variable_name }}"` inside of yml files.

## Gather facts

Ansible does a query to gather a bunch of data about each host before
executing a play or role. To turn this off for designated hosts:
```
- hosts: whatever
  gather_facts: no
```

And then to gather them manually later, in a play, just run the `setup`
module as a task.

## Using when

```
tasks:
  - command: /bin/false
    register: result
    ignore_errors: True

  - command: /bin/something
    when: result|failed

  - command: /bin/something_else
    when: result|succeeded

  - command: /bin/still/something_else
    when: result|skipped
```

The directive `register:` stores the result of the command in the named
variable.


