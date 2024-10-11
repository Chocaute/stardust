---
created: 2024-05-29T16:45
updated: 2024-05-30T17:35
---
The configuration and setup described make use of the `AuthorizedKeysCommand` directive in OpenSSH's SSH daemon (`sshd`). This directive allows you to dynamically retrieve SSH public keys for user authentication from an external source, such as a web server.

### How `AuthorizedKeysCommand` Works

The `AuthorizedKeysCommand` parameter in `sshd_config` specifies a command to be executed when looking up authorized keys for a user. This command can be any executable script or program that fetches and outputs the user's public keys.

___
### Textual Diagram

```
+------------+      1. User attempts to log in     +-------------+
|   User     | ----------------------------------> |  SSH Server |
+------------+                                     +-------------+
                                                        |
                                                        |
                                                        v
                                           2. Run AuthorizedKeysCommand
                                                        |
                                                        |
                                                        v
                                            +-----------------------+
                                            |   Fetch Keys Script   |
                                            +-----------------------+
                                                        |
                                            3. Construct URL and fetch keys
			                                            |
                                                        v
                                          +-----------------------------+
                                          |       Caddy Web Server      |
                                          |  (serving public key files) |
                                          +-----------------------------+
                                                        |
                                            4. Serve public key file
		                                                |
                                                        v
                                            +-----------------------+
                                            |   Fetch Keys Script   |
                                            +-----------------------+
                                                        |
                                               5. Output public keys
		                                                |
                                                        v
                                                 +-------------+
                                                 |  SSH Server |
                                                 +-------------+
                                                        |
                                                6. Authenticate user
		                                                |
                                                        v
                                                 +------------+
                                                 |   User     |
                                                 +------------+
```

### How It Works

1. When a user attempts to log in via SSH, `sshd` runs the script specified in `AuthorizedKeysCommand`, passing the username as an argument.
2. The script uses `curl` to request the public key(s) for the given username from the specified URL.
3. The web server responds with the user's public key(s).
4. `sshd` uses the retrieved keys to authenticate the user.

### Advantages

- **Centralized Key Management**: Public keys are managed centrally on the web server. Adding or removing a user's access is as simple as editing their key file.
- **Dynamic Key Retrieval**: Keys are fetched in real-time, allowing for immediate updates without needing to restart the SSH daemon or modify individual servers.
- **Scalability**: This setup scales well across many servers, as all servers can use the same web-based key repository.

### Security Considerations

- **Man-in-the-Middle Attacks**:
    
    - **Issue**: If the communication between the SSH server and the Caddy web server is not secured, an attacker could intercept and modify the public keys.
    - **Mitigation**: Ensure the web server uses HTTPS to encrypt the communication. Caddy makes this easy as it supports automatic HTTPS by default.
- 
- **Script Injection**:
    
    - **Issue**: If the username (`%u`) is not properly sanitized, it could lead to command injection vulnerabilities.
    - **Mitigation**: Ensure the script properly sanitizes input to prevent injection attacks. Using a well-defined and simple script can help mitigate this risk.
- 
___
### Example Setup

#### On the target host

1. **Configure `sshd_config`**: 
Open your SSH daemon configuration file (`/etc/ssh/sshd_config`) and add the following lines:

```
AuthorizedKeysCommand /path/to/your/script %u 
AuthorizedKeysCommandUser nobody
```

- `AuthorizedKeysCommand`: Specifies the command to run. `%u` is replaced by the username of the user trying to log in.
- `AuthorizedKeysCommandUser`: Specifies the user as which the command is run. Using a non-privileged user like `nobody` is a good security practice.

2. **Create the Script**: 
Create a script (e.g., `/path/to/your/script`) that fetches the public keys. Here’s a sanitized example using `curl`:

```bash
#!/bin/bash
# Ensure the input contains only valid characters (alphanumeric and limited special characters)
if [[ "$1" =~ ^[a-zA-Z0-9_-]+$ ]]; then
	curl -s https://authserver.example.com/keys/$1 
else
	echo "Invalid username" >&2     
	exit 1 
fi
```

 Ensure the script is executable: `chmod +x /path/to/your/script`.

#### On the web server

1. **Install Caddy, then create a Caddyfile**:
To define your site and the directory where the keys will be stored. Typically located at `/etc/caddy/Caddyfile`.

For example, let's say your domain is `authserver.example.com`.

```perl
authserver.example.com {
	root * /var/www/keys
    file_server 
}
```

This configuration tells Caddy to serve files from the `/var/www/keys` directory when accessed via `authserver.example.com`.

2. **Create the Keys Directory**:
Ensure that the directory `/var/www/keys` exists and is accessible by Caddy:

```bash
sudo mkdir -p /var/www/keys 
sudo chown -R caddy:caddy /var/www/keys
```

3. **Add Public Key Files**:
In the `/var/www/keys` directory, add public key files named after the usernames. For example, if you have a user `jdoe`, create a file `/var/www/keys/jdoe` containing the public keys:

```bash
echo "ssh-rsa AAAAB3... user@example.com" > /var/www/keys/jdoe
```

You can add, update, or remove key files as needed to manage access.

___
### Notes

This method implies that there's no specific admin on any given server. There's Tom, or Lola, but no 'Administrator'.

It is an interesting thing to think about. At first, no admin user seems dumb, but the more you think about it, the more it makes sense - especially about traceability. 

You'll always know who's connected to the server, and what they did specifically. 

But I'll mean a different and more involved kind of user management on the servers. 

___

### User management
#### **Traceability and Accountability**
- Every user actions should be logged, and those logs protected from deletions.
#### Security
- Permissions have to be distributed with automation.
- With least privilege, EITHER permissions needs to be tailored for each user OR define different access level profiles.
- Clear automated processes for updating permissions efficiently as roles change and situations arise.

What about the creation of profiles? -> Ansible can handle it.

How do we automatically update who is allowed to go where? -> I don't have a simple solution.

___

### Further notes

Thing is, at the end of the day, we shouldn't need to connect to a server's shell - Ansible should have to handle changes. Logs will be aggregated. Etc etc etc.

So instead of giving user permanent access for the duration of his tenure, how about an only-temporary shell access policy ...?

The workflow is such. An admin named 'Lola' need shell access to a server for some reason. They sent a request, I approve. I instruct Ansible to create and enable a specific user on the server (if it already doesn't exist), with a defined access level profile. 

The admin 'Lola' SSH to the server with his username, the server looks up his public key on the website, you know the drill.

The admin 'Lola' can now access the server. Once he's finished or after the alloted period of time, the admin 'Lola' user on the server is disabled. 

Same thing - I'd grant access for myself. The important part is the granting. 

Naaaah, too complicated. I don't know. 

___

