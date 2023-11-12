# MQTT-PWN

MQTT is a machine-to-machine connectivity protocol designed as an extremely lightweight publish/subscribe 
messaging transport and widely used by millions of IoT devices worldwide. MQTT-PWN intends to be a one-stop-shop 
for IoT Broker penetration-testing and security assessment operations, as it combines enumeration, 
supportive functions and exploitation modules while packing it all within command-line-interface 
with an easy-to-use and extensible shell-like environment.

# Running

```sh
docker-compose build
```

```sh
docker-compose run cli
```

connect -o 172.21.0.2
disconnect

connect -o 172.21.0.2 -u admin -w admin1

# Attacks

## Information Grabber
The MQTT brokers (specifically mosquitto), tend to send some metadata about the broker itself, the clients connected and more.

- MQTT PWND SHELL
```sh
system_info
```


Broker Status
The information (metadata) we grab from the broker can be grabbed through a successful subscription to certain special topics. Those topics are located within the $SYS hierarchy. There are quite a lot of them, but we mainly focus on 9 important topics.

https://manpages.ubuntu.com/manpages/xenial/man8/mosquitto.8.html

## Credentials Brute Force
MQTT protocol uses a centralized broker to communicate between entities (device, sensor, etc.). Those brokers can define a basic authentication mechanism in the form of username / password pair. MQTT-PWN provides a credential brute force module that with a given set of usernames and passwords tries to authenticate to the broker in order to find valid credentials.

```sh
bruteforce --host 172.21.0.2
# bruteforce --host 172.21.0.2  -u admin -pf /mqtt_pwn/resources/wordlists/passwords.txt
```


mosquitto_sub -h localhost -t '$SYS/broker/clients/connected'
mosquitto_sub -h localhost -t '$SYS/broker/version'

# Topic Enumeration
The MQTT protocol allows by design to every entity (device, sensor etc.) to subscribe to any topic it wishes (as long the broker hasn’t enabled any security measures, which by default are off). Using this method, we developed what we called - the discovery plugin, which subscribes for a certain amount of time, to all topics (using wildcard notation) by that enumerating all available topics at a certain time.

## Command & Control   
MQTT can be used for more than connecting your smart home to the cloud. This plugins harnesses the nature of the protocol (publish/subscribe) to create a bot-net like network where the infected clients communicate not to a self owned server directly (traditionally), but to a publicly open broker. By that, masquerading the identity of the bot-net operator and utilizing the broker to handle the vast amount of clients available.


# Feature Support

- Credential Brute-Forcer - configurable brute force password cracking to bypass authentication controls
- Topic Enumerator - establishing comprehensive topic list via continuous sampling over time
- Useful Information Grabber - obtaining and labeling data from an extensible predefined list containing known topics of interest
- GPS tracker - plotting routes from devices using OwnTracks app and collecting published coordinates
- Sonoff Exploiter – design to extract passwords and other sensitive information
- Extensibility - the framework was designed to add new custom plugins with ease
- Shodan - search through `Shodan.io` API for available vulnerable MQTT brokers

# Documentation   

Documentation is available at https://mqtt-pwn.readthedocs.io/.
