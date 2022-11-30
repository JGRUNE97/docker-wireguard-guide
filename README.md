# Guide to installing Wireguard VPN on Digital Ocean.
This guide will go through the quick steps to creating a Droplet through Digital Ocean and installing Wireguard VPN upon it and linking to your phone. If you had not created an account for Digital Ocean, you can find it here: https://m.do.co/c/4d7f4ff9cfe4

1. After creating your Digital Ocean account, go to your interface and click the green 'Create' button on top of the website next to your username and select 'Droplets'. It will then take you to an interface where you can create a Virtual Computer (in Digital Ocean, they are called droplets) and from there you will select the following options as they appear:
    - **Choose an image:** Ubuntu 20.0.4(LTS) x64
    - **Choose a plan:** Basic
    - **CPU options:** Regular with SSD at $4/mo
    - **Choose a datacenter:** New York
    - **Authentication:** Password (*For this example we use a password, however it is recommended you use SSH keys*)
    - **Select additional options:** *Leave Blank*
    - **Finalize and create:** 1 Droplet. Keep host name.
    - **Add tags:** *Leave Blank*
    - **Select Project:** first-project

  2. After selecting these values, you will then create the Droplet which will take only a few minutes for it to start and run and will open a new window to your interface. On the main interface, once the light next to your Droplet name turns green, click on the 'Console:' button found on the top far right side of the IP addresses line and it will then open a new window to a shell interface signing you in as a root user. You will then need to install Docker on your Droplet through the SSH window starting with by installing the required tools. Install these tools by typing in: `sudo apt install apt-transport-https ca-certificates curl software-properties-common -y`
  3. Once it finishes installing the tools, install the Docker key next by entering the command:| `curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -` |
  Once it finishes and mounts the key, you then will follow up and install a repo based on your systems hardware either in 32 bit or 64 bit (for this example, we will install the 32/64 bit repo). Copy and paste the command using 'CTRL+SHIFT+V' into your shell:| `sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"` | After it finishes installing the repository, you then need to switch to the correct repository cache into the shell by typing and entering:| `apt-cache policy docker-ce` | and finally install Docker itself by entering the command:| `sudo apt install docker-ce -y`
4. After Docker finishes installing, install its compose file by entering in the shell:| `sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`| and set its permissions once the compose finishes installing by typing:| `sudo chmod +x /usr/local/bin/docker-compose` |
5. Now that you have Docker installed, you can then make the directories for wireguard as well as creating the docker-compose.yml for its installation. Start out by creating the directories in the shell using the following commands (NOTE, each command will be entered seperately by a '|' character): | `mkdir -p ~/wireguard/` | `mkdir -p ~/wireguard/config/` | After you create the directories, you then want to create and open your compose file in the newly created wireguard folder by typing out the following in shell: | `nano ~/wireguard/docker-compose.yml` |
6. Now opened in the nano bash scripting window for docker-compose.yml, copy and paste the following code block in the file with the hot key 'CTRL+SHIFT+V':

```version: '3.8'
services:
  wireguard:
    container_name: wireguard
    image: linuxserver/wireguard
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Hong_Kong
      - SERVERURL=1.2.3.4
      - SERVERPORT=51820
      - PEERS=pc1,pc2,phone1
      - PEERDNS=auto
      - INTERNAL_SUBNET=10.0.0.0
    ports:
      - 51820:51820/udp
    volumes:
      - type: bind
        source: ./config/
        target: /config/
      - type: bind
        source: /lib/modules
        target: /lib/modules
    restart: always
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
```
Afterwards, you then want to modify the following values in correlation to either you or your server location in the following:
- **TZ:** Change the [Timezone](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) wherever you're at.
- **SERVERURL:** Paste the primary IP from your 'Droplet' dashboard interface.
- **PEERS:** Specify the systems allowed on Wireguard. (*pc1,pc2,phone1 will suffice for this project.*)

Once the changes are made, press 'CTRL+X' to exit the scripting window, and enter 'Y' to save the changes made and 'Enter' to return to the shell.

7. Now that you have your compose file and Docker installed, it's time to install Wireguard onto your server. Change the directory to the Wireguard folder with the command:| `cd ~/wireguard/` | and install the Docker compose file by typing in bash:| `docker-compose up -d` | This should take a couple of minutes depending on your internet speed, but once it is finished it should read back in the command line **Creating wireguard   ... done** indicating it was installed succesfully.
8. Install the Wireguard App on your phone by going to the Google Play store for Android users (or the Apple Store for IPhone users) and search for 'Wireguard'. Install the App and open it which will take you to the main interface showing it to be blank, you then want to pick the option to link the server to your App using a QR Code which can be generated in the shell window by typing in the command:| `docker-compose logs -f wireguard` |. Scan the QR Code with your phone and follow the instructions prompted on the App and it will then open an interface to your phones properties and IP, and the endpoint to your server from the Droplet. When you turn it on, it will then activate the VPN and on the 'Peer' interface, a counter will appear showing the amount of bytes being transferred from your phone's connection to the server itself. You can verify if it's working by going to IPLeak.net and checking to see if the IP address currently on your phone matches to the IP Address from your server. 

### That concludes the walkthrough guide, the following images provided are proof of concept of how the QR Code appears through the Shell on a PC as well as the IP address before Wireguard was turned on and after on a phone.