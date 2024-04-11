# 1. Description:

I'm creating this repo because of the nginx problem in CORS,..etc. during setup. The following Quick Start assumes that you already have nginx proxy manager installed and want to install netmaker using nginx proxy manager.

(Note: If you haven't configure Nginx Proxy Manager, please refer to [Section 5](#5-additional-setup-nginx-proxy-manager)

# 2. Quick Start:

Following the original netmaker documentation, just run `nm-quick.sh` as normally:

```
sudo wget -qO /root/nm-quick.sh https://raw.githubusercontent.com/gravitl/netmaker/master/scripts/nm-quick.sh && sudo chmod +x /root/nm-quick.sh && sudo /root/nm-quick.sh
```

Then stop right when it start to deploy everything (Ctrl + C), the nm-quick.sh is for preparing a good start with configuration and the environment.

Then pull the configuration:

```
curl -O https://raw.githubusercontent.com/SMUEric1127/netmaker-nginx-proxy-manager/main/docker-compose.yml
```

Then easily deploy it with (Note: I added the --env-file as sometime docker-compose failed to read the environment file)

```
docker-compose --env-file netmaker.env -f docker-compose.yml up -d --force-recreate 
```

# 3. Nginx Proxy Manager:

Here is the current Nginx Proxy Manager screen: <img width="1220" alt="image" src="https://github.com/SMUEric1127/netmaker-nginx-proxy-manager/assets/85500156/9dee0fa1-7a2a-4cbc-824e-67cb97f49fec">

You need to add 3 host: api.${NM_DOMAIN} -> netmaker:8081, broker.${NM_DOMAIN} -> http://mq:1883, and dashboard.${NM_DOMAIN} -> http://netmaker-ui:80 as following

<img width="1220" alt="Screenshot 2024-04-11 at 2 24 27 PM" src="https://github.com/SMUEric1127/netmaker-nginx-proxy-manager/assets/85500156/d9af017c-4b8e-4864-a1a0-d08402e0e337">

Then here's the tricky part - <b><u>CORS</u></b>

# 4.  (Important) CORS in Nginx Proxy Manager:

This is the part that took me the most of time, including finding all methods, header. In short, just add the following lines to the section advance setting of the api.${NM_DOMAIN}

<img width="523" alt="Screenshot 2024-04-11 at 2 26 26 PM" src="https://github.com/SMUEric1127/netmaker-nginx-proxy-manager/assets/85500156/12694aac-14d2-4a39-88a9-286c1c9b15b3">

the lines are:

```
location / {
    if ($request_method = OPTIONS) {
        return 204;
    }

    add_header Access-Control-Allow-Origin 'https://dashboard.nmk.chatspace.info';
    add_header 'Access-Control-Expose-Headers' 'Content-Length, Content-Range';
    add_header 'Access-Control-Max-Age' 1728000;
    add_header 'Access-Control-Allow-Headers' 'User-Agent, Keep-Alive, Content-Type, from-ui, Authorization, Origin, X-Requested-With, Content-Type, Accept';
    add_header "Access-Control-Allow-Methods" "GET, POST, OPTIONS, HEAD, DELETE, PUT";
    add_header 'Access-Control-Allow-Credentials' 'true';

    # Pass requests to the backend
    proxy_pass http://netmaker:8081;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-Scheme $scheme;
    proxy_set_header X-Forwarded-For $remote_addr;
}
```

After that, save it, and voilà! 🎨🖌️ 

# 5. (Additional) Setup Nginx Proxy Manager:

Pull the docker-compose file from this repo

```
curl -O https://raw.githubusercontent.com/SMUEric1127/netmaker-nginx-proxy-manager/main/docker-compose.nginx.yaml
```

Then deploy it:

```
docker-compose up -d
```

Then you can access through <IP>:81 and start from there. More information can be found at: [Nginx Proxy Manager](https://github.com/NginxProxyManager/nginx-proxy-manager)

# Contribution:

Please create any pull request to improve the readme or anything else. Hope this help you all!
