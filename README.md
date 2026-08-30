# MC-Server-Machine-Switch
A fast and simple method of switching the host machine of a Minecraft Server for Arch.


This guide will show you how to easily switch the machine of where your Minecraft server is hosted, while keeping the world's data intact.
Optionally, this can swap over Playit aswell.


# GUIDE

## Step 0

Ensure you have:

A main machine, where you'd like the server to be hosted most often
A secondary machine, where you'd like the server to transfer when the main one is off.

and install these packages on both machines.

''' sudo pacman -S openssh rsync systemd --needed '''


I'm going to assume you already have java, the Minecraft server, and optionally playit.

## Step 1

Set up your directories.
Edit these to your liking, though these are my personal ones.

MAIN MACHINE: /home/azure/minecraftservers/26_2_FABRIC
SECONDARY MACHINE: /home/homeserver/minecraftservers/26_2_FABRIC

Move/make your Minecraft server as usual to the main machine's folder.

## Step 2

Configure SSH.
SSH has to work on both machines in order to sync the world over.

First, we need to know both machine's IPs.
run this command on both.

''' ip addr '''

Find an ip that looks similar to this: 192.168.1.123

On your main machine, generate an SSH key
