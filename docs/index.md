# BYOS - Bring Your Own Satellite




## Setting up your Virtual Machine
```
sudo apt update && sudo apt dist-upgrade -y
```

<figure markdown>
![alt text](image.png){ width="900" }
  <figcaption>Updating OS</figcaption>
</figure>


```
sudo apt install docker-compose -y
```
<figure markdown>
![alt text](image-1.png)){ width="900" }
  <figcaption>Installed Docker and Dependencies</figcaption>
</figure>


```
sudo groupadd docker
sudo usermod -aG docker $USER
```

Log out and Log back in...

open terminal and run: ```docker version```

<figure markdown>
![alt text](image-2.png){ width="900" }
  <figcaption>Docker Version Output</figcaption>
</figure>

