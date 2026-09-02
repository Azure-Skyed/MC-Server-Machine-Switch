# MC-Server-Machine-Switch
A fast and simple method of switching the host machine of a Minecraft Server for Arch.

### Watch a demo!
https://www.youtube.com/watch?v=Xnc5Dc_Kl3U

Optionally, this can swap over Playit aswell.

### MANUAL GUIDE

</summary>

#### Step 0

Ensure you have:

A main machine, where you'd like the server to be hosted most often.

A secondary machine, where you'd like the server to transfer when the main one is off.

Install these packages on both machines.

```bash
sudo pacman -S openssh rsync systemd --needed 
```

I'm going to assume you already have java, the Minecraft server, and optionally playit.

#### Step 1

Set up your directories.
Edit these to your liking, though these are my personal ones.

MAIN MACHINE: `/home/azure/minecraftservers/26_2_FABRIC`

SECONDARY MACHINE: `/home/homeserver/minecraftservers/26_2_FABRIC`

Move/make your Minecraft server as usual to the main machine's folder.
(I recommend changing your server's .jar to `server.jar`, to make it more organized.)
It's always a good idea to make a backup of your server.



#### Step 2

Configure SSH.
SSH has to work on both machines in order to sync the world over.

Start it on both machines.
```bash
sudo systemctl enable --now sshd
```

First, we need to know both machine's local IP.
Run this command on both machines.

```bash
ip addr 
```

Find an IP that looks similar to this: 192.168.1.123,
Copy them and save them for later.

On your main machine, generate an SSH key

```bash
ssh-keygen -t ed25519
```
Just press enter for all of them

Next, copy that key over to the secondary machine.

```bash
ssh-copy-id SECONDARY_USERNAME@SECONDARY_IP
```
Replace SECONDARY_USERNAME and SECONDARY_IP with your secondary machine's user's username, and IP which we saved earlier.

SSH may ask if you trust the host, just type `yes`
Then it may ask you for the password, put that in too.

Test the connection.
```bash
ssh SECONDARY_USERNAME@SECONDARY_IP
```
You shouldn't be asked for a password now.
`exit` once you know it works

#### Secondary

Now we have to do that the other way around on the secondary machine.

```bash
ssh-keygen -t ed25519
```
Press enter for all


```bash
ssh-copy-id MAIN_USERNAME@MAIN_IP
```
Replace MAIN_USERNAME and MAIN_IP with your main machine's user's username, and IP which we saved earlier.

SSH may ask if you trust the host, just type `yes`
Then it may ask you for the password, put that in too.

Test the connection.
```bash
ssh MAIN_USERNAME@MAIN_IP
```
You shouldn't be asked for a password again.
`exit` once you know it works

#### Step 3

Create the Minecraft systemd service

On the main machine, run:
```bash
sudo nano /etc/systemd/system/minecraft.service
```

Paste this in there:

```bash
[Unit]
Description=Minecraft Server
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=MAIN_USER # replace this with your username
WorkingDirectory=/home/MAIN_USER/minecraftservers/SERVER # replace this with your directory
ExecStart=/usr/bin/java -Xmx6G -jar server.jar nogui # replace this with your start command
Restart=no
TimeoutStopSec=30
KillSignal=SIGINT

[Install]
WantedBy=multi-user.target
```
Please read the comments in that to see what you need to replace

##### Secondary

Now do the same on the secondary machine

On the secondary machine, run:
```bash
sudo nano /etc/systemd/system/minecraft.service
```

Paste this in there:

```bash
[Unit]
Description=Minecraft Server
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=SECONDARY_USER # replace this with your username
WorkingDirectory=/home/SECONDARY_USER/minecraftservers/SERVER # replace this with your directory
ExecStart=/usr/bin/java -Xmx6G -jar server.jar nogui # replace this with your start command
Restart=no
TimeoutStopSec=30
KillSignal=SIGINT

[Install]
WantedBy=multi-user.target
```
Please read the comments in that to see what you need to replace.


<details>
<summary>
  
### OPTIONAL: Set up Playit

</summary>

This is to use playit.gg.
If you aren't using it, you can safely ignore this step, and any other steps that reference playit.


#### Step 1

Sync the config.

Run playit and set it up on your main machine.
Once you did that, copy the config over to the secondary machine. It should be in
`~/.config/playit_gg/playit.toml`

Run this to copy it over.

```bash
scp /home/MAIN_USER/.config/playit_gg/playit.toml SECONDARY_USER@SECONDARY_IP:/home/SECONDARY_USER/.config/playit_gg/playit.toml
```
Replace `MAIN_USER`, `SECONDARY_USER`, and `SECONDARY_IP`.

Also, download and run playit on the secondary machine and ensure everything works as it is supposed to.
Probably not a good idea to run playit on both machines at the same time.

#### Step 2

Create the Playit systemd service

On the main machine, run:

```bash
sudo nano /etc/systemd/system/playit.service
```

Paste this in there:

```bash
[Unit]
Description=Playit TunnelAdded optional Playit setup instructions in details.
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=MAIN_USER # replace this with your username
WorkingDirectory=/home/MAIN_USER # replace this with your playit directory
ExecStart=/home/MAIN_USER/playit-linux-amd64 # replace this with your username
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```
Please read the comments in that to see what you need to replace

Reload systemd
```bash
sudo systemctl daemon-reload
```

##### Secondary

Now do the same on the secondary machine

On the secondary machine, run:

```bash
sudo nano /etc/systemd/system/playit.service
```

Paste this in there:

```bash
[Unit]
Description=Playit Tunnel
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=SECONDARY_USER # replace this with your username
WorkingDirectory=/home/SECONDARY_USER # replace this with your playit directory
ExecStart=/home/SECONDARY_USER/playit-linux-amd64 # replace this with your username
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```
Please read the comments in that to see what you need to replace

Reload systemd
```bash
sudo systemctl daemon-reload
```

Playit will be sprinkled in later, just remember to use the playit dropdown.

</details>


#### Step 4

Allow the switches to control systemd.

On the main machine, run

```bash
sudo EDITOR=nano visudo
```

Scroll to the bottom, and add

```bash
MAIN_OR_SECONDARY_USER ALL=(root) NOPASSWD: /usr/bin/systemctl start minecraft, /usr/bin/systemctl stop minecraft
```
Do this on both machines.

<details>
<summary>
  
##### Playit

</summary>

If youre using playit, make sure you add `, /usr/bin/systemctl start playit, /usr/bin/systemctl stop playit`.

final line will be:

```bash
MAIN_OR_SECONDARY_USER ALL=(root) NOPASSWD: /usr/bin/systemctl start minecraft, /usr/bin/systemctl stop minecraft, /usr/bin/systemctl start playit, /usr/bin/systemctl stop playit
```
Do this on both machines.
  
</details>

#### Step 5

Install `mc-main`

This will be the command to run in your terminal to make the server run on your main machine.

Make this only on the main machine.

```bash
sudo nano /usr/local/bin/mc-main
```

<details>
<summary>
  
##### Playit

</summary>

```bash
#!/bin/bash

set -e

SECONDARY="SECONDARY_USER@SECONDARY_IP" # Secondary server's username and ip
SECONDARY_SERVER="/home/SECONDARY_USER/minecraftservers/SERVER/" # Secondary server's directory
MAIN_SERVER="/home/MAIN_USER/minecraftservers/SERVER/" # Main server's directory
# you dont need to change anything else

echo "==> Stopping Minecraft on secondary..."
ssh "$SECONDARY" "sudo -n systemctl stop minecraft"

echo "==> Waiting for Minecraft to stop..."
ssh "$SECONDARY" "sudo -n systemctl is-active minecraft >/dev/null 2>&1 && exit 1 || true"

echo "==> Stopping Playit on secondary..."
ssh "$SECONDARY" "sudo -n systemctl stop playit"

echo "==> Syncing secondary → main..."
rsync -a --delete --info=progress2 \
    "$SECONDARY:$SECONDARY_SERVER" \
    "$MAIN_SERVER"

echo "==> Starting Minecraft on main..."
sudo systemctl start minecraft

echo "==> Starting Playit on main..."
sudo systemctl start playit
echo
echo "================================"
echo " Minecraft is now running at MAIN"
echo "================================"
```
</details>

<details>
<summary>
  
##### Minecraft only

</summary>

```bash
#!/bin/bash

set -e

SECONDARY="SECONDARY_USER@SECONDARY_IP" # Secondary server's username and ip
SECONDARY_SERVER="/home/SECONDARY_USER/minecraftservers/SERVER/" # Secondary server's directory
MAIN_SERVER="/home/MAIN_USER/minecraftservers/SERVER/" # Main server's directory
# you dont need to change anything else

echo "==> Stopping Minecraft on secondary..."
ssh "$SECONDARY" "sudo -n systemctl stop minecraft"

echo "==> Waiting for Minecraft to stop..."
ssh "$SECONDARY" "sudo -n systemctl is-active minecraft >/dev/null 2>&1 && exit 1 || true"

echo "==> Syncing secondary → main..."
rsync -a --delete --info=progress2 \
    "$SECONDARY:$SECONDARY_SERVER" \
    "$MAIN_SERVER"

echo "==> Starting Minecraft on main..."
sudo systemctl start minecraft

echo
echo "================================"
echo " Minecraft is now running at MAIN"
echo "================================"
```

</details>

Look for the comments near the top of the script and edit it.


Lastly, make it executable.

```bash
sudo chmod +x /usr/local/bin/mc-main
```

#### Step 6

Now we do a similar thing for the other way around. this will be `mc-secondary`.

Make this only on the main machine.

```bash
sudo nano /usr/local/bin/mc-secondary
```

<details>
<summary>
  
##### Playit

</summary>

```bash
#!/bin/bash

set -e

SECONDARY="SECONDARY_USER@SECONDARY_IP" # Secondary server's username and ip
MAIN_SERVER="/home/MAIN_USER/minecraftservers/SERVER/" # Main server directory
SECONDARY_SERVER="/home/SECONDARY_USER/minecraftservers/SERVER/" # Secondary server's directory
# nothing else needs to be edited

echo "==> Stopping Minecraft on main..."
sudo systemctl stop minecraft

echo "==> Stopping Playit on main..."
sudo systemctl stop playit

echo "==> Syncing main → secondary..."
rsync -a --delete --info=progress2 \
    "$MAIN_SERVER" \
    "$SECONDARY:$SECONDARY_SERVER"

echo "==> Starting Minecraft on secondary..."
ssh "$SECONDARY" "sudo -n systemctl start minecraft"

echo "==> Starting Playit on secondary..."
ssh "$SECONDARY" "sudo -n systemctl start playit"

echo
echo "=================================="
echo " Minecraft is now running at SECONDARY"
echo "=================================="
```

Edit the first few lines with the comments next to them.

</details>

<details>
<summary>
  
##### Minecraft only

</summary>

```bash
#!/bin/bash

set -e

SECONDARY="SECONDARY_USER@SECONDARY_IP" # Secondary server's username and ip
MAIN_SERVER="/home/MAIN_USER/minecraftservers/SERVER/" # Main server directory
SECONDARY_SERVER="/home/SECONDARY_USER/minecraftservers/SERVER/" # Secondary server's directory
# nothing else needs to be edited

echo "==> Stopping Minecraft on main..."
sudo systemctl stop minecraft

echo "==> Syncing main → secondary..."
rsync -a --delete --info=progress2 \
    "$MAIN_SERVER" \
    "$SECONDARY:$SECONDARY_SERVER"

echo "==> Starting Minecraft on secondary..."
ssh "$SECONDARY" "sudo -n systemctl start minecraft"

echo
echo "=================================="
echo " Minecraft is now running at SECONDARY"
echo "=================================="
```

</details>

Remember to make it executable.

```bash
sudo chmod +x /usr/local/bin/mc-secondary
```

#### Step 7

Finally, let's start the server.

From your main machine, run

```bash
sudo systemctl start minecraft
```

<details>
<summary>
  
##### Playit

</summary>
If you are using playit, also run

```bash
sudo systemctl start playit
```

</details>

Open minecraft, and make sure that server is working.

Once you're ready to swap it over, open a terminal and run 
```bash
mc-secondary
```
It should sync over to the secondary machine, and run the server there.
The first swap will take significantly longer than any other sync.

**Never run both Minecraft servers at the same time.**

</details>
