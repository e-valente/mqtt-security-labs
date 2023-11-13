# Info Grabber Attack

This section provides a comprehensive guide on the essential steps required to replicate the conducted experiments successfully. Additionally, it presents the detailed results obtained from these experiments.


- Start backend (mosquitto & suricatta):

```sh
cd backend/mosquitto-suricata
docker-compose up
```

- On malicious device: ESP32 or a Desktop machine (linux, macOS, or windows), subscribe to any internal broker topic, for instance:

```sh
mosquitto_sub  -t '$SYS/broker/version'  -u client1 -P Tijolo200! -v
```

- Checking the alert on suricata stdout

```sh
docker exec -it suricata /bin/bash

tail -f /var/log/suricata/fast.log
```

You'll see a similar message:

```sh
07/02/2023-16:02:10.409761  [**] [1:20:1] Information Grabber alert - Grab version [**] [Classification: Someone has tried to get internal information from broker] [Priority: 1] {TCP} 172.21.0.1:56882 -> 172.21.0.2:1883
```

You can now change the rule to `DROP` mode. To do this, follow these steps:

Open the `backend/mosquitto-suricata/rules/information-grabber.rules` file.

Change the rule:

```sh
alert tcp $HOME_NET any -> any 1883 (msg:"Information Grabber alert - Grab version"; mqtt.subscribe.topic; content:"SYS/broker/version"; endswith; classtype:information-grabber; sid:20; rev:1;)
``` 

to

```sh
drop tcp $HOME_NET any -> any 1883 (msg:"Information Grabber DROP - Grab version"; mqtt.subscribe.topic; content:"SYS/broker/version"; endswith; classtype:information-grabber; sid:20; rev:1;)
```  

Next, re-rerun the `docker-compose up` command and redo the experiment.

