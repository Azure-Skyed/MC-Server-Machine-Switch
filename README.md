# MC-Server-Machine-Switch
A fast and simple method of switching the host machine of a Minecraft Server for Arch.


This guide will show you how to easily switch the machine of where your Minecraft server is hosted, while keeping the world's data intact.
Optionally, this can swap over Playit aswell.


## GUIDE

### Step 0

Ensure you have:

A main machine, where you'd like the server to be hosted most often
A secondary machine, where you'd like the server to transfer when the main one is off.

and install these packages on both machines.

```bash
sudo pacman -S openssh rsync systemd --needed 
```

I'm going to assume you already have java, the Minecraft server, and optionally playit.



### Step 1

Set up your directories.
Edit these to your liking, though these are my personal ones.

MAIN MACHINE: `/home/azure/minecraftservers/26_2_FABRIC`

SECONDARY MACHINE: `/home/homeserver/minecraftservers/26_2_FABRIC`

Move/make your Minecraft server as usual to the main machine's folder.
(I recommend changing your server's .jar to `server.jar`, to make it more organized.)



### Step 2

Configure SSH.
SSH has to work on both machines in order to sync the world over.

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


### Step 3

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

#### Secondary

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
  
## OPTIONAL: Set up Playit

</summary>

This is to use playit.gg.
If you aren't using it, you can safely ignore this step, and any other steps that reference playit.


### Step 1

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

### Step 2

Create the Playit systemd service

On the main machine, run:

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

#### Secondary

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


### Step 4

Allow the switches to control systemd.

On the main machine, run

```bash
sudo EDITOR=nano visudo
```

Scroll to the bottom, and add

```bash
MAIN_USER ALL=(root) NOPASSWD: /usr/bin/systemctl start minecraft, /usr/bin/systemctl stop minecraft
```

<details>
<summary>
  
#### Playit

</summary>

If youre using playit, make sure you add `, /usr/bin/systemctl start playit, /usr/bin/systemctl stop playit`.

final line will be:

```bash
MAIN_USER ALL=(root) NOPASSWD: /usr/bin/systemctl start minecraft, /usr/bin/systemctl stop minecraft, /usr/bin/systemctl start playit, /usr/bin/systemctl stop playit
```
  
</details>
