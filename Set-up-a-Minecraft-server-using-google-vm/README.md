# Create a VM hosting a Minecraft Server
It is possible to host a Minecraft Server directly on a Compute Engine instance. To make sure there is plenty of room for the Minecraft server's world data, it is suggested to attach a high-performance SSD to the instance.

## Goal of the project
- Create an application server
- Install software and dependency
- Configure networking features
- Schedule regular backups
- Automate server maintenance with startup and shutdown scripts

## Create a VM
Create a VM using Cloud Console or Cloud Shell with the following constraints:
- Name: `mc-server`
- Region: custom
- Zone: custom
- Boot: `Debian GNU/Linux 11 (bullseye)`
- Access scopes for Storage: R/W
- Persistent disk: at least 50 GB SSD
- Encryption type: `Google-managed`
- Network tag: minecraft-server
- External IP Address: `Reserve Static` (name: mc-server-ip)

## Install software and dependency
The server will run on the top of the Java Virtual Machine, hence it requires the Java Runtime Environment.

```bash
sudo apt-get update
sudo apt-get install -y default-jre-headless 
```

Install wget
```bash
sudo apt-get install -y wget
```

Install screen which allows to create a virtual terminal
```bash
sudo apt-get install -y screen
```

## Configure the persistent disk and run the application
Activate the SSH connection through the VM and set up a new folder as a mount point.
```bash
sudo mkdir -p /home/minecraft
```

To format the disk
```bash
sudo mkfs.ext4 -F -E lazy_itable_init=0, lazy_journal_init=0, discard /dev/disk/by-id/google-minecraft-disk
```

Mount the disk in the created folder
```bash
sudo mount -o discard,defaults /dev/disk/by-id/google-minecraft-disk /home/minecraft
```

Download the latest Minecraft server JAR file
```bash
cd /home/minecraft
sudo wget https://launcher.mojang.com/v1/objects/d0d0fe2b1dc6ab4c65554cb734270872b72dadd6/server.jar
```

Inizialize the server
```bash
sudo java -Xmx1024M -Xms1024M -jar server.jar nogui
```
But, it won't run unless you set `eula=true` in the file `eula.txt`. Since starting again the server, it will be tied to the life of the SSH session. Hence, it is possible to run the server as a background process using a virtual terminal.

```bash
sudo screen -S mcs java -Xmx1024M -Xms1024M -jar server.jar nogui
```

The screen terminal can be detached pressing `Ctrl+A`, `Ctrl+D`. To reattach the terminal, instead, you can use `sudo screen -r mcs`.

## Setting the Firewall rules
 
In order to enable external traffic it is mandatory to set a new firewall rule from the Firewall tab in the VPC network menu. Then, create a new Firewall rule.
 
- Name: `minecraft-rule`
- Target: `Specified target tags`
- Target tags: `minecraft-server`
- Source filter: `IPv4 ranges`
- Source IPv4 ranges: `0.0.0.0/0`
- Protocols and ports: `Specified protocols and ports`
- Protocol: `TCP`
- Port: `25565`
 
## Exploiting the Minecraft Server Status to test the application
Visit https://mcsrvstat.us/ and test your server.
 
1. Go to Compute Engine > VM instances and note the `External IP Address` of your VM server.
2. Paste your External IP Address on the web page.
3. Click on `Get server status`.
 
## Scheduling regular backups
 
Set a persistent variable name for your bucket.
 
```bash
export YOUR_BUCKET_NAME=<Enter your bucket name here>
 
echo YOUR_BUCKET_NAME=$YOUR_BUCKET_NAME >> ~/.profile
```
 
Then, create the bucket via Cloud SDK
```shell
gcloud storage buckets create gs://$YOUR_BUCKET_NAME-minecraft-backup
```

Copy the file `backup.sh` into `/home/minecraft/backup.sh`. Then, make the script executable and run it.
```shell
sudo chmod 755 /home/minecraft/backup.sh
. /home/minecraft/backup.sh
```

Now, you can schedule a cron job to automate the task. Open the cron table:
```shell
sudo crontab -e
```
and enter at the bottom:
```shell
0 */X * * * /home/minecraft/backup.sh
```
Change the X value, according to your preference, to run backups every X hours.

## Automate server maintenance

To perform server maintenance, you need to i) shut down the server and ii) stop the VM instance. This operation unmount the persistent disk from the VM. So, you need to configure and run application again (see section above). But, you automate this process using metadata scripts.

- Click `edit` on your VM instances and looking for the `Metadata` section. Then, click `Add Item`
 
    - startup-script-url: `https://storage.googleapis.com/cloud-training/archinfra/mcserver/startup.sh`
    -  shutdown-script-url: `https://storage.googleapis.com/cloud-training/archinfra/mcserver/shutdown.sh`


