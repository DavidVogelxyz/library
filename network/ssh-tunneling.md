# SSH tunneling

NB: this guide makes frequent references to the IP address `127.0.0.1`. Obviously, this is a reference to the hostname `localhost`. Either can be used in place of the other; however, using the IP address itself avoids both the "host file lookup", as well as the potential for trying the IPv6 entry for `localhost`.

## Table of contents

- [Local SSH tunnel](#local-ssh-tunnel)
- [Remote SSH tunnel](#remote-ssh-tunnel)
- [SSH proxy tunnel](#ssh-proxy-tunnel)
- [References](#references)

## Local SSH tunnel

In a local SSH tunnel, it is best to think about how the SSH tunnel begins at the local machine, and reaches out across the internet to the remote machine.

An example where a local SSH tunnel would be useful is the following scenario:

There is a server hosted on the internet at an IP of 123.123.123.123. The server puts up a firewall such that the only connections it allows are SSH connections (port 22). However, the standard HTTP port (80) is blocked. Therefore, attempting to access http://123.123.123.123 from a local machine results in an inaccessible webpage.

This is a great use case for a "regular" SSH tunnel.

The local user would run the following command from their local machine:

```
ssh -N -L 127.0.0.1:9999:127.0.0.1:80 $USER@123.123.123.123
```

Using this command, the webpage at 123.123.123.123 would now be accessible at http://127.0.0.1:9999

Explanation:

- The `-N` flag makes the shell non-interactive, and tells the shell not to allow remote commands to be executed.
    - Without this flag, the `ssh` command would log into a shell as a normal `ssh` command would, and allows the connected user to enter commands as one may expect.
- The `-L` flag opens a local port with a specific endpoint (in this case, "127.0.0.1:9999").
- "127.0.0.1:9999" is the bind address of the connection on the local machine. This means that the SSH connection is essentially "port forwarded" to that address, at that port.
    - "What is being accessed."
    - It is common that "127.0.0.1" isn't written into this section of the command, as the connection assumes the bind address would be "127.0.0.1".
    - Therefore, `ssh -N -L 9999:127.0.0.1:80 $USER@123.123.123.123` would function the exact same way as the above command.
    - By connecting to the local machine at port 9999, the connection is tunneled over to the remote machine and talks to port 80.
- "127.0.0.1:80" is the host address of the connection on the remote machine. This means that the SSH connection is being made with "127.0.0.1:80" of the remote machine.
    - "What is received."
    - This is why the connection servers as a tunnel, allowing the local user running the `ssh` command to interact with the HTTP page on the remote address.
- "$USER@123.123.123.123" is the typical use of the `ssh` command. It specifies that `ssh` establishes a connection to the designated IP, signing in as the user specified.
    - Obviously, as would be true with any other `ssh` connection, the address could be an IP address, but could also be a domain name.
    - Example: `ssh $USER@example.com`

When running this command, the `ssh` command will appear to hang, so long as the connection is active. Cancelling out of the command breaks the tunnel, and would result in the webpage no longer being accessible to the local user.

## Remote SSH tunnel

In a remote (reverse) SSH tunnel, it is best to think about how the SSH tunnel begins at the remote machine, at the specified port, and reaches out across the internet back to the local machine.

An example where a remote SSH tunnel would be useful is the following scenario:

There is a server hosted on a local network at an IP of 192.168.10.50. The server is only accessible to users within the local network. Currently, users would either need to be on-site, connected to the local network, or they can connect remotely using a VPN connection and access the server. However, there is an interest in making this service publicly available, but in a secure way.

This is a great use case for a remote SSH tunnel.

The local user would run the following command from the local server hosting the service (192.168.10.50), connecting to a remote server (123.123.123.123) that they also control.

```
ssh -N -R 127.0.0.1:9999:127.0.0.1:80 $USER@123.123.123.123
```

Using this command, the webpage that the local server is hosting would now be available to anyone who connects to the remote server at http://123.123.123.123:9999

Explanation:

- As before, the `-N` flag makes the shell non-interactive, and tells the shell not to allow remote commands to be executed.
- The `-R` flag designates the port on the remote server ("127.0.0.1:80") to be forwarded to the given host and port on the local side ("127.0.0.1:9999").
    - As will be noted in the following bullet points, the "location" of the addresses has been reversed.
        - In the "regular" tunnel, the first address and port is for the local machine, and the second set refers to the remote machine.
        - With a remote tunnel, the first address and port refer to the remote machine, and the second set refers to the local machine.
- In this case, "127.0.0.1:9999" is the bind address of the connection on the ***remote*** machine.
    - "What is being accessed."
    - Any user logged into a session at 123.123.123.123 would be able to access the service on 192.168.10.50 by going to http://127.0.0.1:9999
    - Another result of this connection is that, with the remote server available to the public, anyone who connects to http://123.123.123.123:9999 would *also* be able to access the service.
    - It is common that "127.0.0.1" isn't written into this section of the command, as the connection assumes the bind address would be "127.0.0.1".
    - Therefore, `ssh -N -R 9999:127.0.0.1:80 $USER@123.123.123.123` would function the exact same way as the above command.
    - However, there's an additional trick that can be used with a remote tunnel:
        - If the bind address is set to "0.0.0.0", then the SSH connection will listen for any outside SSH connections connecting to port 9999.
        - This trick can be used to allow SSH connections to a local machine that's behind a firewall that doesn't allow *any* SSH connections.
- "127.0.0.1:80" is the host address on the connection on the ***local*** machine.
    - "What is received."
    - In this example, the `ssh` command is being run from the server hosting the service, which is why "127.0.0.1:80" is being used.
    - However, any machine on the local network that has access to the service could facilitate the connection by specifying "192.168.10.50:80" in its place.
        - Example: `ssh -N -R 127.0.0.1:9999:192.168.10.50:80 $USER@123.123.123.123`
        - Using the example of a server running a service to be accessed remotely, it makes sense that the server itself would open the `ssh` connection, rather than a separate machine.
        - Technically, this would be a form of a proxy tunnel. Specifically, this would be both a proxy and a remote SSH tunnel.
    - In addition, just like with other `ssh` connections, the address specified here could be a domain.
        - If the service was being hosted on example.com's internal network, the command may look like the following:
            - Example: `ssh -N -R 127.0.0.1:9999:$SERVER.example.com:80 $USER@123.123.123.123`
- "$USER@123.123.123.123" is the typical use of the `ssh` command. It specifies that `ssh` establishes a connection to the designated IP, signing in as the user specified.
    - Obviously, as would be true with any other `ssh` connection, the address could be an IP address, but could also be a domain name.
    - Example: `ssh $USER@example.com`

When running this command, the `ssh` command will appear to hang, so long as the connection is active. Cancelling out of the command breaks the tunnel, and would result in the webpage no longer being accessible to the local user.

## SSH proxy tunnel

An SSH proxy tunnel is any sort of an SSH connection where a *third* machine is involved in the data transit, rather than just two.

This was demonstrated in the previous section on remote tunnels. Originally, the connection was being made between a local server (192.168.10.50) hosting a service and a remote server (123.123.123.123) that would allow public access to the locally running service. In this original case, the SSH connection involved only those two machines: the local server and the remote server.

However, in the explanation section, a second form of the connection was described. Instead of the SSH connection being facilitated by the local server directly to the remote server, another machine (192.168.10.100) could run the `ssh` command. Now, the SSH connection involves three machines: the local server, the remote server, and an intermediary machine. This is an example of a proxied tunnel, as the remote server would see the connecting machine as the 192.168.10.100 machine (***not*** 192.168.10.50, the server actually hosting the service).

While a proxy connection would require additional resources (the third machine), it can also provide an additional layer of security. For example, it may be preferable that a remote SSH tunnel doesn't directly connect back to a host server. Imagine that the remote server that publicly hosts the local service is attacked. One of the benefits of a remote tunnel is that it ***isn't*** initiated by the remote machine, and therefore limits the extent to which an attacker would be able to find and attack the local server running the service. However, through connection logs and other methods, it may be possible for the attacker to assess where the traffic is coming from. By having a proxy machine facilitate the connection to the local service, an extra hop has been added to the traffic, meaning that the attacker would next have to compromise the proxy machine before potentially being able to find and target the local server.

Another example where a SSH proxy tunnel would be useful is the following scenario:

There is a server hosted on the internet at an IP of 123.123.123.123. The server puts up a firewall such that the only connections it allows are SSH connections (port 22). However, the standard HTTP port (80) is blocked. Therefore, attempting to access http://123.123.123.123 from a local machine results in an inaccessible webpage.

In addition, the server has a firewall rule that allows the service to be accessed, but only by machines with a public IP address of 222.222.222.222. For reference, the local user is not currently on the network with a public IP of 222.222.222.222. While the user *could* use a VPN connection to connect to the network at 222.222.222.222, there are other local services on their own network that they want to keep access to. A split VPN connection is a potential solution, but the local user would prefer *not* to use a VPN connection if possible.

This is a great use case for a proxied "regular" SSH tunnel.

The local user would run the following command from their local machine:

```
ssh -N -L 127.0.0.1:9999:123.123.123.123:80 $USER@222.222.222.222
```

Through the use of this command, the server will accept the SSH connection, and will see that the local user is connecting to the server from a public IP address of 222.222.222.222. Therefore, the webpage at 123.123.123.123 will now be accessible at http://127.0.0.1:9999

Explanation:

- The `-N` flag makes the shell non-interactive, and tells the shell not to allow remote commands to be executed.
    - Without this flag, the `ssh` command would log into a shell as a normal `ssh` command would, and allows the connected user to enter commands as one may expect.
- The `-L` flag opens a local port with a specific endpoint (in this case, "127.0.0.1:9999").
- "127.0.0.1:9999" is the bind address of the connection on the local machine. This means that the SSH connection is essentially "port forwarded" to that address, at that port.
    - It is common that "127.0.0.1" isn't written into this section of the command, as the connection assumes the bind address would be "127.0.0.1".
    - Therefore, `ssh -N -L 9999:123.123.123.123:80 $USER@222.222.222.222` would function the exact same way as the above command.
- "123.123.123.123:80" is the host address of the connection on the remote machine. This means that the SSH connection is being made with "123.123.123.123:80" of the remote machine.
    - This is why the connection servers as a tunnel, allowing the local user running the `ssh` command to interact with the HTTP page on the remote address.
- "$USER@222.222.222.222" is the typical use of the `ssh` command. It specifies that `ssh` establishes a connection to the designated IP, signing in as the user specified.
    - Obviously, as would be true with any other `ssh` connection, the address could be an IP address, but could also be a domain name.
    - Example: `ssh $USER@example.com`

When running this command, the `ssh` command will appear to hang, so long as the connection is active. Cancelling out of the command breaks the tunnel, and would result in the webpage no longer being accessible to the local user.

## References

- [YouTube - Tony Teaches Tech - How to SSH Tunnel (simple example)](https://www.youtube.com/watch?v=x1yQF1789cE)
    - Reference for "creating local SSH tunnel"
- [YouTube - Tony Teaches Tech - How to Reverse SSH Tunnel](https://www.youtube.com/watch?v=TZ6W9Hi9YJw)
    - Reference for "creating remote SSH tunnel"
- [YouTube - Tony Teaches Tech - How to Make an SSH Proxy Tunnel](https://www.youtube.com/watch?v=F-ubwghsWPM)
    - Reference for "creating SSH proxy tunnel"
- [YouTube - anthonywritescode - don't use localhost (intermediate) anthony explains #534](https://www.youtube.com/watch?v=98SYTvNw1kw)
    - A video referencing the benefits of using `127.0.0.1`, as opposed to `localhost`
