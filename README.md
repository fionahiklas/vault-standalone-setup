# vault-standalone-setup

## Prerequisites 

* [Vault](https://learn.hashicorp.com/tutorials/vault/getting-started-install)
* [Podman](https://podman.io)
* [Vagrant](https://www.vagrantup.com)
* [VirtualBox](https://www.virtualbox.org)





## Notes

### Running Vault with Podman - Development Mode

```
podman run -d -v $PWD/volumes/logs:/vault/logs -v $PWD/volumes/file:/vault/file -p 8200:8200 vault:1.6.2
```

NOTE: This doesn't actually use the file volume since the development mode stores in memory

NOTE: Also this rootless podman doesn't enable memory locking since it doesn't add the IPC_LOCK
capability as you can do with docker

You can access the UI for vault using the [URL](http://localhost:8200/ui)

You need the `Root Token` to login to the UI and you can find this in the output from the container.
Run `podman ps` to list the containers and then `podman logs <container name>` to see the output.


### Running Vault with Podman - Server Mode

```
podman run -d -e 'VAULT_LOCAL_CONFIG={"backend": {"file": {"path": "/vault/file"}}, "default_lease_ttl": "168h", "max_lease_ttl": "720h", "disable_mlock": "true" }' -v $PWD/volumes/logs:/vault/logs -v $PWD/volumes/file:/vault/file -p 8200:8200 vault:1.6.2 server
```

NOTE: The `disable_mlock` option is needed for rootless podman as it's not possible to enable locking by passing in the 
`--cap-add IPC_LOCK` unless you're launching this as root.

The above command appears to start the vault server but there is no obvious way to connect to it.  Running a shell in the 
container to investigate using the following command

```
podman exec -it <container name/id>  /bin/sh
```




### Preventing other access to environment variables

The suggestion is to use environment variables to pass secrets into processes.  I'm concerned that for a 
given user, say "tomcat", it would be possible to access `/proc/<pid>/environ` for any process and 
therefore capture the keys/tokens/password.

A suggested approach to at least help with this is using the `hidepid=2` option which prevents other 
users from being able to see the `/proc/<pid>` entries for others on the same system.  If we make sure that 
rootless podman containers run under different host users then that should limit damage should an attacker
gain access.

I guess the other precaution is to remove the login shell for users but then it's unclear how processes could 
be run using `podman` - unless for root `su - <user>` could be used, or possibly `systemctl` or rc scripts used to 
launch applications.



### Dealing with files with mapped uid/gid

After running vault in a podman container using the following command, the `volumes/file` and `volumes/logs`
directories become owned by `100099.100999` and it's impossible to switch them back from the host shell

```
podman run -d -v $PWD/volumes/logs:/vault/logs -v $PWD/volumes/file:/vault/file -p 8200:8200 vault:1.6.2
```

Checking ownership you get this

```
ls -al volumes/
total 16
drwxrwxr-x 4 fiona  fiona  4096 Feb 24 11:40 .
drwxrwxr-x 6 fiona  fiona  4096 Feb 24 15:16 ..
drwxrwxr-x 2 100099 100999 4096 Feb 24 11:42 file
drwxrwxr-x 2 100099 100999 4096 Feb 24 11:47 logs
```

You can actually check the uid/gid that has been mapped

```
podman unshare ls -aln volumes/
total 16
drwxrwxr-x 4   0    0 4096 Feb 24 11:40 .
drwxrwxr-x 6   0    0 4096 Feb 24 15:18 ..
drwxrwxr-x 2 100 1000 4096 Feb 24 11:42 file
drwxrwxr-x 2 100 1000 4096 Feb 24 11:47 logs
```

You can't change the ownership from outside

```
chown fiona.fiona volumes/logs/
chown: changing ownership of 'volumes/logs/': Operation not permitted
```

You can do it from "inside" with the following command

```
podman unshare chown -R root.root volumes/
```

Inside the namespace the uid/gid for "fiona" is mapped to "root" which is allowed to change permissions,
outside that namespace the files have now been returned to the previous owner.


## References

### Vault

* [Vault project page](https://www.vaultproject.io)
* [Vault docker container](https://hub.docker.com/_/vault)
* [Installing Vault](https://learn.hashicorp.com/tutorials/vault/getting-started-install)



### Git

* [Amend commits](https://www.git-tower.com/learn/git/faq/change-author-name-email/)


### Podman 

* [Podman run](http://docs.podman.io/en/latest/markdown/podman-run.1.html)
* [Changing UID on files with rootless podman](https://github.com/containers/podman/issues/7052)
* [Running rootless podman](https://www.redhat.com/sysadmin/rootless-podman-makes-sense)
* [Podman and newuidmap](https://superuser.com/questions/1529632/why-is-a-normal-user-allowed-to-give-away-a-file-folder-by-running-podman-unsha)
* [Podman unshare](https://www.mankier.com/1/podman-unshare)


## Linux

* [User namespaces](https://manpages.debian.org/buster/manpages/user_namespaces.7.en.html#User_and_group_ID_mappings:_uid_map_and_gid_map)
* [Proc filesystem](https://www.kernel.org/doc/Documentation/filesystems/proc.txt)
* [hidepid /proc](https://linux-audit.com/linux-system-hardening-adding-hidepid-to-proc/)
* [SSL Load locations, SSL_CERT_DIR](https://www.openssl.org/docs/man1.1.0/man3/SSL_CTX_set_default_verify_paths.html)

## Vagrant

* [Vagrant](https://www.vagrantup.com)
* [VirtualBox](https://www.virtualbox.org)
* [Ubuntu 20.04 Focal base box](https://app.vagrantup.com/ubuntu/boxes/focal64)
