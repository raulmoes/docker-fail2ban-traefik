Before running docker compose up, we must create a username and password to access the traefik control panel. For this we install apache2 utils (*apt install apache2-utils*) and then we put the following command: **htpasswd -nb user secure_password**, it will generate an output with the *user:password*. 

We copy it and insert it in the *traefik_dynamic.toml* file, replacing **user:passwordencrypt**

Next, change **yourdomaininforcontrolpaneltraefik** in the same file with the domain that accesses the control panel.

Save the file and access the **traefik.toml** file and indicate your email in the **"email"** field for Lets Encrypt certificates.

Then change permissions chmod 600 for file acme.json

Finally, create a network with **docker network create web** and run docker compose up.

In order for the containers that we want to come out through Traefik, we must put the web network as external in each docker compose of these services, so that Traefik sees them when they are on the same network:

**networks: 
   web: 
       external: true

And then assign it to the services that only go out through the Traefik proxy.

Finally, we assign the labels tag to each service, replacing *NAME* with an alias and *youraccessdomain* with the domain through which it is accessed.

The port does not need to be indicated as it is automatically detected by Traefik.

```
labels:
  - traefik.http.routers.NAME.rule=Host(`youraccessdomain`)
  - bringefik.http.routers.NAME.tls=true
  - traefik.http.routers.NAME.tls.certresolver=lets-encrypt
# - traefik.port=80
```

In the case of portainer to access through the proxy, if we put the previous labels it does not work, for portainer it is necessary to indicate other parameters that are the following:

```
  - traefik.http.routers.NAME.rule=Host(`youraccessdomain`)
  - traefik.http.services.frontend.loadbalancer.server.port=9000
  - bringefik.http.routers.NAME.tls=true
  - traefik.http.routers.NAME.tls.certresolver=lets-encrypt
# - traefik.port=9000
```

*If fail2ban doesn't work, it's because your distribution uses nftables backend, so we should do the following, using iptables in legacy mode.

Initially we will install the following packages to ensure that our operating system can use Iptables in Legacy mode: **apt install -y iptables arptables ebtables**

Once the packages are installed we will put Iptables in Legacy mode by executing the following commands in the terminal:

   **• sudo update-alternatives --set iptables /usr/sbin/iptables-legacy 
   • sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy 
   • sudo update-alternatives --set arptables /usr/sbin/arptables-legacy 
   • sudo update-alternatives --set ebtables /usr/sbin/ebtables-legacy

If we want to leave it as we had it initially, that is, using the nftables backend, we will put the following commands:

*• sudo update-alternatives --set iptables /usr/sbin/iptables-nft 
• sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-nft 
• sudo update-alternatives --set arptables /usr/sbin/arptables-nft 
• sudo update-alternatives --set ebtables /usr/sbin/ebtables-nft

Another action that I have taken to solve the error is to activate the Kernel Multiport module. To see if you have this module loaded in the Kernel, you have to execute the following command in the terminal: **cat /proc/net/ip_tables_matches**

If the word multiport does not appear, we will execute the command: **sudo modprobe –v xt_multiport**

Finally, we reboot the system and when it boots, we stop the fail2ban and traefik container and we start it again to apply the new iptables and it works.
