# Info Grabber Attack

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

### 3.1 In other terminal (let's call terminal 2), subscribe to the topic '$SYS/broker/version'

```sh
mosquitto_sub  -t '$SYS/broker/version'  -u client1 -P Tijolo200! -v
```

TIP: check [here](https://mosquitto.org/man/mosquitto-8.html) all the `$SYS` topics on mosquitto.

You'll receive a message similar of following:

```sh
$SYS/broker/version mosquitto version 2.0.18
```

## 4 Set up monitoring using Suricata

Make sure the backend is stopped:

```sh
cd backend/mosquitto-suricata
docker-compose stop  # or ctrl + c on previous step terminal
```

### 4.1 Configure suricata rule 

Add the following [suricata rule](https://docs.suricata.io/en/suricata-6.0.0/rules/intro.html) to the `backend/mosquitto-suricata/suricata-files/rules/my.rules` file:

`my.rules`:  
```sh
alert tcp $HOME_NET any -> any 1883 (msg:"Information Grabber alert - Grab version"; mqtt.subscribe.topic; content:"SYS/broker/version"; endswith; classtype:information-grabber; sid:20; rev:1;)
```

### 4.2 Start Backend

```sh
cd backend/mosquitto-suricata
docker-compose up
```

### 4.3 Subscribe to the topic '$SYS/broker/version'

Terminal 2:  
```sh
mosquitto_sub  -t '$SYS/broker/version'  -u client1 -P Tijolo200! -v
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

After subscribing, we'll see in terminal 3 (suricata terminal) the following alert:

```sh
1/13/2023-22:39:27.852231  [**] [1:20:1] Information Grabber alert - Grab version [**] [Classification: Someone has tried to get internal information from broker] [Priority: 1] {TCP} 172.21.0.1:40812 -> 172.21.0.2:1883
```

## 5 Set up prevention using Suricata

Make sure the backend is stopped:

```sh
cd backend/mosquitto-suricata
docker-compose stop  # or ctrl + c on previous step terminal
```

### 5.1 Configure suricata rule for dropping the publish messages om this topic

Add the following [suricata rule](https://docs.suricata.io/en/suricata-6.0.0/rules/intro.html) to the `backend/mosquitto-suricata/suricata-files/rules/my.rules` file:

`my.rules`:  
```sh
drop tcp $HOME_NET any -> any 1883 (msg:"Information Grabber drop - Grab version"; mqtt.subscribe.topic; content:"SYS/broker/version"; endswith; classtype:information-grabber; sid:20; rev:1;)

```

### 5.2 Start Backend

```sh
cd backend/mosquitto-suricata
docker-compose up
```

### 5.3 Subscribe to tthe topic '$SYS/broker/version'

Terminal 2:  
```sh
mosquitto_sub  -t '$SYS/broker/version'  -u client1 -P Tijolo200! -v
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

After subscribing, we'll see in terminal 3 (suricata terminal) the following drop message (i.e. the message didn't get to the topic):

```sh
11/13/2023-22:45:28.702538  [Drop] [**] [1:20:1] Information Grabber drop - Grab version [**] [Classification: Someone has tried to get internal information from broker] [Priority: 1] {TCP} 172.21.0.1:42162 -> 172.21.0.2:1883
```

See also in terminal 2, the mosquitto client didn't receive the message content.
