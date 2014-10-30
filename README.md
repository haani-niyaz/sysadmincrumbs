# Sys Admin Crumbs #


This is a collection of useful (atleast to me) snippets and recipes. It is a work in progress so may not contain a proper structure at this stage.


## Sudo ##


### How to Add groups/users to `/etc/sudoers`? ###

This is the recommended approach as it avoides polluting the original sudo file and makes management & maintenance easy.

Instead of modifying the `/etc/sudoers` file do the following:

1. Create a `/etc/sudoers.d` dir if it does not exist
2. Create a file with no extension and file permission 0440

```
chmod 0440 <filename>
chown root:root <filename>
```

3. Add the file to `/etc/sudoers.d/` dir. 

4. Include the directory in the sudoers file by performing:
   
   ```
   # /usr/sbin/visudo
   sudo visudo

   # Add the following to the sudoers file. Note the hash has been explicitly added:
   #includedir /etc/sudoers.d 

   ```
4. All files now added to `/etc/sudoers.d` will be automatically imported 


### How to check what sudo acces a user has? ###

`sudo -l -U bob`



## SSH ##


### How to copy your key to a remote server? ###

`ssh-copy-id -i username@remoteserver`

ssh-copy-id is a script that uses ssh to log into a remote machine to append your key to the authorized_keys file and sets the correct permission on `~/.ssh` and `~/.ssh/authorized_keys` file.

The '-i' identity option gives it the identify file  (defaults to `~/.ssh/id_rsa.pub`)

