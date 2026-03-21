# ACL

For all child datasets, NFSv4 ACL is preferable over POSIX ACL since it allows the TrueNAS UI to apply execute permissions on only directory and not files.
To achieve this, set the ACL like so:

![Read and write ACL](/Read-Write.png)

![Execute ACL](/Execute.png)

Do not use the basic "Read" ACL, as it will add the execute permission on files!

To give read access to another user, add these additional ACEs:

![Additional user read ACL](/Additional-Read.png)

![Additional user execute ACL](/Additional-Execute.png)