## Copy SSH
```shell
# MacOS
$ pbcopy < ~/.ssh/id_rsa.pub
# Windows
$ clip < ~/.ssh/id_rsa.pub
```

## Disabling Password Authentication on your Server
> Ref : https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server

If you were able to login to your account using SSH without a password, you have successfully configured SSH key-based authentication to your account. However, your password-based authentication mechanism is still active, meaning that your server is still exposed to brute-force attacks.

Before completing the steps in this section, make sure that you either have SSH key-based authentication configured for the root account on this server, or preferably, that you have SSH key-based authentication configured for an account on this server with sudo access. This step will lock down password-based logins, so ensuring that you have will still be able to get administrative access is essential.

Once the above conditions are true, log into your remote server with SSH keys, either as root or with an account with sudo privileges. Open the SSH daemon's configuration file:
```shell
sudo nano /etc/ssh/sshd_config
```
Inside the file, search for a directive called PasswordAuthentication. This may be commented out. Uncomment the line and set the value to "no". This will disable your ability to log in through SSH using account passwords:
```
PasswordAuthentication no
```
Save and close the file when you are finished. To actually implement the changes we just made, you must restart the service.
On Ubuntu or Debian machines, you can issue this command:
```shell
sudo service ssh restart
```
On CentOS/Fedora machines, the daemon is called sshd:
```shell
sudo service sshd restart
```
After completing this step, you've successfully transitioned your SSH daemon to only respond to SSH keys.

# Initial Server Setup with Ubuntu 16.04
> Ref : https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04
