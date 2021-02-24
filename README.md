# vault-standalone-setup



## Notes

### Running Podman - Development Mode

```
podman run -d -v $PWD/volumes/logs:/vault/logs -v $PWD/volumes/file:/vault/file -p 8200:8200 vault:1.6.2
```

NOTE: This doesn't actually use the file volume since the development mode stores in memory

NOTE: Also this rootless podman doesn't enable memory locking since it doesn't add the IPC_LOCK
capability as you can do with docker

You can access the UI for vault using the [URL](http://localhost:8200/ui)

You need the `Root Token` to login to the UI and you can find this in the output from the container.
Run `podman ps` to list the containers and then `podman logs <container name>` to see the output.




### Preventing other access to environment variables

The suggestion is to use environment variables to pass secrets into processes.  I'm concerned that for a 
given user, say "tomcat", it would be possible to access `/proc/<pid>/environ` for any process and 
therefore capture the keys/tokens/password.




### Dealing with files with mapped uid/gid

After running vault in a podman container using the following command, the `volumes/file` and `volumes/logs`
directories become owned by user `100999` and it's impossible to switch them back from the host shell




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


## Linux

* [User namespaces](https://manpages.debian.org/buster/manpages/user_namespaces.7.en.html#User_and_group_ID_mappings:_uid_map_and_gid_map)
* [Proc filesystem](https://www.kernel.org/doc/Documentation/filesystems/proc.txt)
* [hidepid /proc](https://linux-audit.com/linux-system-hardening-adding-hidepid-to-proc/)


## Vagrant

* [Vagrant](https://www.vagrantup.com)
* [VirtualBox](https://www.virtualbox.org)
* [Ubuntu 20.04 Focal base box](https://app.vagrantup.com/ubuntu/boxes/focal64)
