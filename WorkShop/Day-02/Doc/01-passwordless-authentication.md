# How to Set Up Passwordless Authentication

## EC2 Instances

### Using Public Key

```
ssh-copy-id -f "-o IdentityFile <PATH TO PEM FILE>" ubuntu@<INSTANCE-PUBLIC-IP>
```

* **ssh-copy-id**: Command used to copy your public key to a remote machine.
* **-f**: Forces the copying of keys, useful if keys already exist and need overwriting.
* **"-o IdentityFile <PATH TO PEM FILE>"**: Specifies the identity file (private key) to use for the connection. The `-o` flag passes this option to the underlying SSH command.
* **ubuntu@<INSTANCE-IP>**: The username (`ubuntu`) and the public IP address of the remote server.

After running this command, you should be able to SSH into the instance without entering a password:

```
ssh ubuntu@<INSTANCE-PUBLIC-IP>
```

---

### Using Password

1. Open the SSH configuration file:

   ```
   sudo nano /etc/ssh/sshd_config.d/60-cloudimg-settings.conf
   ```

2. Update the following line:

   ```
   PasswordAuthentication yes
   ```

3. Restart the SSH service to apply the changes:

   ```
   sudo systemctl restart ssh
   ```

You can now log in to your EC2 instance using a password if needed.
