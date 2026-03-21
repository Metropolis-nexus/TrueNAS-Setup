# ACL

For all child datasets, NFSv4 ACL is preferable over POSIX ACL since it allows the TrueNAS UI to apply execute permissions on only directory and not files.
To achieve this, set the ACEs like so:

![Read and write ACE](/Read-Write.png)

![Execute ACE](/Execute.png)

Do not use the basic "Read" option, as it will add the execute bit to files!

Set the equivalent of 000 for group and others:
![Group ACE](/Group.png)
![Others ACE](/Others.png)

To give read access to another user, add these additional ACEs:

![Additional user read ACE](/Additional-Read.png)

![Additional user execute ACE](/Additional-Execute.png)