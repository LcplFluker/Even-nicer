# Even-nicer

MMFL-023-M, bHgP1MIsYgoW, 10.50.11.9

scp -oControlPath=/tmp/demo <SOURCE> <DEST>

ssh -MS /tmp/<SOCKET_NAME> <USER>@<TGT IP>
You can verify the socket (OPTIONAL)

ssh -S /tmp/SOCK -O check <USER>@<TGT>
# Expected output: Master running (pid=XXXXX)
Step 2: Reuse the connection (Mulitplexing)
To open a new session, just point ssh to your scoket file. reuse exisitng connection to get a new session

ssh -S /tmp/SOCK a@1
Tip

Wait, why a@1?

Since the master connection has already authenticated, the user and host (a@1) for subsequent sessions are just placeholders to satisfy the command's syntax. They are ignored for authentication, so you can put anything there!

Step 3: Manage Port Forwards On-the-Fly
You can now add or remove port forwards (tunnels) without tearing down your connection.

# open a port forward over an existing session
ssh -S /tmp/<SOCK> -O forward -L 2222:<TGT>:22 a@1

# Remove a port forward over an existing session
ssh -S /tmp/<SOCK> -O cancel -L 2222:<TGT>:22 a@1

# Add a dynamic port forward (SOCKS)
ssh -S /tmp/<SOCK> -O forward -D 9050 a@1

# Remove the dynamic port forward
ssh -S /tmp/<SOCK> -O cancel -D 9050 a@1
Step 4: Use Sockets Through Existing tunnels.
We can also use master control sockets through existing tunnels!

# Use an existing control socket to tunnel 
ssh -p <PORT> -S /tmp/T1 <USER>@127.0.0.1 -L <LOCAL_PORT>:<TGT_IP>:<DEST_PORT>

# Create a new Master Connection over an existing tunnel
ssh -p <PORT> -MS /tmp/T2 <USER>@127.0.0.1
Here is an example

#      NIX                  T1                  T2                  T3
# +------------+      +------------+      +------------+      +------------+      
# |            |      |            |      |            |      |            |   
# |            ------->22          |      |            |      |            |   
# |        2222>====================------>22          |      |            |    
# |        3333>========================================------>80          |     
# |            |      |            |      |            |      |            |    
# +------------+      +------------+      +------------+      +------------+  

# create master connection to t1 and forward tunnel to t2
ssh -MS /tmp/t1 user@t1 -L 2222:T2:22

# create a master connection to t2 and forward tunnel to t3
ssh -p 2222 -MS /tmp/t2 user@127.0.0.1 -L 3333:T3:80

# use tunnel to hit T3 web server
wget http://127.0.0.1:3333

# open another terminal on T1
ssh -S /tmp/t1 a@1

# open another terminal on T2
ssh -S /tmp/t2 a@1
