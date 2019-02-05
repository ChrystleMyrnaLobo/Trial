# Working with networked machine
Remote: server ip `10.1.2.3` and user `chrystle` alias as `viz`. Following commands are run from local  

## Passworless SSH
- File `/etc/hosts`, alias the hostname
```
10.1.2.3 viz
```
- File `~/.ssh`, mention user
```
Host viz
	User chrystle
```
- For passwordless ssh, generate key-pairs 
```
ssh-keygen
ssh-copy-id -i ~/.ssh/id_rsa.pub viz
```

## Sync files
- Good old scp (secure copy)
```
scp -r foo_dir viz:~/downloads
```
- Faster rsync
```
# Use foo_dir/ for content of foo_dir
# Use foo_dir to copy the dir and contents within

# dry run: 
rsync -avn foo_dir viz:~/downloads

# actual data
#rsync -avzh foo_dir viz:~/downloads
```

## Temporary port forwarding
- Redirect all connections to the portA on local to portB of server
```
ssh -L portA:remotenode:portB user@remotenode -N
```
- Open `http://localhost:portA/`
- Specific example: Redirects all connections to the port 8080 on your local machine to 80 port of your remote machine. Access `http://192.168.0.10:8080/`
```
ssh -L 192.168.0.10:8080:10.0.0.10:80 root@10.0.0.10
```