# Ansible KeePass Lookup Plugin

Perhaps, from a security view point, this solution is the same as `ansible-vault`.
Just if you are storing secrets data in KeePass, then why not use it, 
instead of duplicating to `ansible-vault`. 


## Installation

    pip install pykeepass --user
    mkdir -p ~/.ansible/plugins/lookup && cd "$_"
    curl https://raw.githubusercontent.com/viczem/ansible-keepass/master/keepass.py -o ./keepass.py

[More about ansible plugins installation](https://docs.ansible.com/ansible/latest/dev_guide/developing_locally.html)


## Variables

- `keepass_dbx` - path to KeePass file
- `keepass_psw` - password. [*optional*] if the socket is used
- `keepass_key` - [*optional*] path to keyfile


## Usage

For global variables define them once in `group_vars/all`.

For security reasons, do not store KeePass password in plain text. 
Use `ansible-vault encrypt_string` to encrypt the password. 
I'm not sure, but I think that for simplicity, 
it is safe to use the same `ansible-vault` password as KeePass password.
To decrypt the passwod use `--ask-vault-pass`
 e.g. `ansible all -m ping --ask-vault-pass`.


    # file: group_vars/all
    
    keepass_dbx: "~/.keepass/database.kdbx"
    keepass_psw: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          ...


### Alternative usage with UNIX socket

> _This usage is more preferred for performance reason, 
because of KeePass file stay decrypted and not need to reopen after done each playbook task 
[(see the issue for more info)](https://github.com/viczem/ansible-keepass/issues/1)_

In this case, there is no need to enter a password for KeePass each time Ansible is called.
Run socket by the command and after that enter a password to make to open KeePass file.

    python kpsock.py ~/.keepass/database.kdbx


The command will creates UNIX socket in the same directory as KeePass file. 
> **WARNING**: The KeePass file and password are stay decrypted in memory while the socket is open.

The socket timeout is 1 minute since past access (will be closed automatically when not used).
To change timeout use `--ttl` argument (see help `python kpsock.py -h`)

To send the running command in background press <kbd>CTRL</kbd>+<kbd>Z</kbd> and execute `bg` 
(`fg` to get the job into the foreground again).


## Example

Define variables you need e.g. in any file in group_vars


    ansible_user       : "{{ lookup('keepass', 'path/to/entry', 'username') }}"
    ansible_become_pass: "{{ lookup('keepass', 'path/to/entry', 'password') }}"


You can get another [properties of an KeePass entry](https://github.com/pschmitt/pykeepass/blob/master/pykeepass/entry.py)
(not only `username` or `password`)

 
`ansible-doc -t lookup keepass` - to get description of the plugin