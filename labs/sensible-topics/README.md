# Sensible Topics 

## Requirements

- mosquitto (host machine): follow the installation instructions [here](https://mosquitto.org/download/)
- docker + docker-compose: both are present in the [docker-desktop](https://www.docker.com/products/docker-desktop/) package (recommended)

## 1 Clone the Repo

```sh
git clone git@github.com:e-valente/mqtt-security-labs.git
```

## 2 Build & Start Backend

```sh
cd backend/mosquitto-suricata
docker-compose up
```

## 3 Test pub & sub

### 3.1 In other terminal (let's call terminal 2), Subscribe to the topic 'sensible'

```sh
mosquitto_sub  -t 'sensible'  -u client1 -P Tijolo200! -v
```

### 3.2 Publish the message to the topic 'sensible'

Terminal 3:

```sh
mosquitto_pub  -t 'sensible' -m "Testing sensible"  -u client1 -P Tijolo200!
```

## 4 Set up monitoring using Suricata

Make sure the backend is stopped:

```sh
cd backend/mosquitto-suricata
docker-compose stop  # or ctrl + c on previous step terminal
```

### 4.1 Configure suricata rule 

Add the following [suricata rule](https://docs.suricata.io/en/suricata-6.0.0/rules/intro.html) to the `backend/mosquitto-suricata/rules/my.rules` file:

`my.rules`:  
```sh
alert tcp $HOME_NET any -> any 1883 (msg:"Sensible topic alert"; mqtt.publish.topic; content:"sensible"; classtype:sensible-topic; sid:10; rev:3;)
```

### 4.2 Start Backend

```sh
cd backend/mosquitto-suricata
docker-compose up
```

### 4.3 Subscribe to the topic 'sensible'

Terminal 2:  
```sh
mosquitto_sub  -t 'sensible'  -u client1 -P Tijolo200! -v
```

### 4.4 In other terminal (let's call terminal 3), get a shell on Suricata container

Terminal 3:
```sh
docker exec -it suricata /bin/bash
```

### 4.5 In suricata shell (terminal 3), tail the alert file

Terminal 3:
```sh
tail -f /var/log/suricata/fast.log
```

### 4.6 In other terminal (let's call terminal 4), publish the message to the topic 'sensible'

Terminal 4:
```sh
mosquitto_pub  -t 'sensible' -m "Testing sensible"  -u client1 -P Tijolo200!
```

After publishing, we'll see in terminal 3 (suricata terminal) the following alert:

```sh
1/10/2023-20:57:01.080197  [**] [1:10:3] Sensible topic alert [**] [Classification: Sensible Topic Publishing Activity was Detected] [Priority: 1] {TCP} 172.21.0.1:39630 -> 172.21.0.2:1883
```

## 5 Set up prevention using Suricata

Make sure the backend is stopped:

```sh
cd backend/mosquitto-suricata
docker-compose stop  # or ctrl + c on previous step terminal
```

### 5.1 Configure suricata rule for dropping the publish messages om this topic

Add the following [suricata rule](https://docs.suricata.io/en/suricata-6.0.0/rules/intro.html) to the `backend/mosquitto-suricata/rules/my.rules` file:

`my.rules`:  
```sh
drop tcp $HOME_NET any -> any 1883 (msg:"Sensible topic preventing"; mqtt.publish.topic; content:"sensible"; classtype:sensible-topic; sid:10; rev:3;)
```

### 5.2 Start Backend

```sh
cd backend/mosquitto-suricata
docker-compose up
```

### 5.3 Subscribe to the topic 'sensible'

Terminal 2:  
```sh
mosquitto_sub  -t 'sensible'  -u client1 -P Tijolo200! -v
```

### 5.4 In other terminal, get a shell on the Suricata container

Terminal 3:
```sh
docker exec -it suricata /bin/bash
```

### 5.5 In suricata shell (terminal 3), tail the alert file

Terminal 3:
```sh
tail -f /var/log/suricata/fast.log
```

### 5.6 In other terminal, publish the message to the topic 'sensible'

Terminal 4:
```sh
mosquitto_pub  -t 'sensible' -m "Testing sensible"  -u client1 -P Tijolo200!
```

After publishing, we'll see in terminal 3 (suricata terminal) the following drop message (i.e. the message didn't get to the topic):

```sh
[Drop] [**] [1:10:3] Sensible topic preventing [**] [Classification: Sensible Topic Publishing Activity was Detected] [Priority: 1] {TCP} 172.21.0.1:53096 -> 172.21.0.2:1883
```

See also in terminal 2, the mosquitto client didn't receive the message content.
