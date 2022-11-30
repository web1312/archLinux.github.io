# Wireguard VPN Install
## Digital Ocean
 - Sign up for Digital Ocean Account with promotional link for $200 credit
 ### Create Ubuntu Droplet
 - Click 'Get started with a droplet'
 - Create a new droplet with following specifications
    - Region - New York NYC1
    - Image - Click 'Marketplace' >> Docker 20.10.21 on Ubuntu 20.04
    - Size - Basic
    - Regular Intel CPU with SSD
    - Cheapest droplet available - $6/mo
    - Authentication Method - Password - 1WestonBurr
    - Give it a cool hostname

### Wireguard
- Open console for droplet you just created 
- Install Wireguard with:

     `mkdir -p ~/wireguard/`
      
    `mkdir -p ~/wireguard/config/`
      
    `nano ~/wireguard/docker-compose.yml`
- In the nano window that opens, paste :
    
        version: '3.8'
        services:
        wireguard:
            container_name: wireguard
            image: linuxserver/wireguard
            environment:
            - PUID=1000
            - PGID=1000
            - TZ=America/Chicago
            - SERVERURL=206.189.188.152
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

- Change TZ to different timezone if necessary and SERVERURL to your droplet IPV4 address

- Start Wireguard: 
   
        cd ~/wireguard/
        docker-compose up -d

- After Wireguard creation is done, run: `
    
      docker-compose logs -f wireguard

- Open wireguard app on phone and scan QR code
- You will know VPN is working if you IP address changes like this: [Before](before.PNG) and [After](afterWireguard.PNG).

 Source: https://thematrix.dev/setup-wireguard-vpn-server-with-docker/
