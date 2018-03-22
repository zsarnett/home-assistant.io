---
layout: page
title: "Snips"
description: "Instructions on how to integrate Snips within Home Assistant."
date: 2018-03-22 12:00
sidebar: true
comments: false
sharing: true
footer: true
logo: snips.png
ha_category: Voice
ha_release: 0.65
---

The [Snips Voice Platform](https://www.snips.ai) allows users to add powerful voice assistants to their Raspberry Pi devices without compromising on privacy. It runs 100% on-device, and does not require an internet connection. It features Hotword Detection, Automatic Speech Recognition (ASR), Natural Language Understanding (NLU) and Dialog Management.

The latest documentation can be found here: [Snips Platform Documentation](https://github.com/snipsco/snips-platform-documentation/wiki).

![Snips Modules](/images/screenshots/snips_modules.png)

Snips takes voice or text as input and produces *intents* as output, which are explicit representations of an intention behind an utterance and which can subsequently be used by Home Assistant to perform appropriate actions.

![Snips Modules](/images/screenshots/snips_nlu.png)


## {% linkable_title The Snips Voice Platform %}

### {% linkable_title Installation %}

All of the Snips installation documentation can be found at [

[Raspberry Pi](https://github.com/snipsco/snips-platform-documentation/wiki/1.-Setup-the-Snips-Voice-Platform#1-snips-platform-for-raspberry-pi-3-with-raspbian-stretch)

Unfortunately, only Debian Jessie is supported at this time.
[Debian/AMD64](https://github.com/snipsco/snips-platform-documentation/wiki/1.-Setup-the-Snips-Voice-Platform#2-snips-platform-installation-on-debianamd64)

Docker supports both AMD64 and ARM architechtures
[Docker](https://github.com/snipsco/snips-platform-documentation/wiki/6.--Miscellaneous#using-docker)

### {% linkable_title Creating an assistant %}

<div class='videoWrapper'>
<iframe src="https://player.vimeo.com/video/223255884" width="700" height="380" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
</div>

Head over to the [Snips Console](https://console.snips.ai) to create your assistant. Launch the training and download by clicking on the "Download Assistant" button.

The next step is to get the assistant to work on your device. Assuming your downloaded assistant folder is on your desktop (your assistant name will vary), just run:

```bash
$ scp -r ~/Desktop/assistant.zip pi@<raspi_hostname.local_or_IP>:/home/pi/.
```

Now ssh into your Raspberry Pi:

```bash
$ ssh pi@<raspi_hostname.local_or_IP>
unzip -o -d /usr/share/snips /home/pi/assistant.zip
```

### {% linkable_title Running Snips %}

Make sure that a microphone is plugged to the Raspberry Pi. If you are having trouble setting up audio, we have written a guide on [Raspberry Pi Audio Configuration](https://github.com/snipsco/snips-platform-documentation/wiki/1.-Setup-the-Snips-Voice-Platform-on-your-Raspberry-Pi#configuring-the-audio).

Start the Snips Voice Platform by starting the `snips-*` services:

```bash
$ sudo systemctl start 'snips-*'
```

Snips is now ready to take voice commands from the microphone. To trigger the listening, simply say

> Hey Snips

followed by a command, e.g.

> Turn on the living room lights

As the Snips Platform parses this query into an intent, it will be published on MQTT, on the `hermes/intent/<intentName>` topic. The Snips Home Assistant component subscribes to this topic, and handles the intent according to the rules defined in `configuration.yaml`, as explained below.

#### {% linkable_title Optional: specifying an external MQTT broker %}

By default, Snips runs its own MQTT broker. But we can also tell Snips to use an external broker by specifying this when launching Snips. In this case, we need to specify this in the `/etc/snips.toml` configuration file. For more information on configuring this, see the [Using an external MQTT broker](https://github.com/snipsco/snips-platform-documentation/wiki/6.--Miscellaneous#using-external-mqtt-broker) article.

## {% linkable_title Home Assistant configuration %}

### {% linkable_title Specifying the MQTT broker %}

Messages between Snips and Home Assistant are passed via MQTT. We can either point Snips to the MQTT broker used by Home Assistant, as explained above, or tell Home Assistant which [MQTT broker](/docs/mqtt/) to use by adding the following entry to the `configuration.yaml` file:

```yaml
mqtt:
  broker: MQTT_BROKER_IP
  port: MQTT_BROKER_PORT
```

By default, Snips runs a mosquitto broker on port 1883. So if we wish to use this broker, and if Snips and Home Assistant run on the same device, the entry will look as follows:

```yaml
mqtt:
  broker: 127.0.0.1
  port: 1883
```

Alternatively, MQTT can be configured to bridge messages between servers if using a custom MQTT broker such as [mosquitto](https://mosquitto.org/).

### {% linkable_title Triggering actions %}

In Home Assistant, we trigger actions based on intents produced by Snips using the [`intent_script`](/components/intent_script) component. For instance, the following block handles a `ActivateLightColor` intent to change light colors:

{% raw %}
```yaml
snips:

intent_script:
  TurnOn:
    action:
      - service: light.turn_on
        data_template:
          entity_id: light.{{ objectLocation | replace(" ","_") }}
```
{% endraw %}

In the `data_template` block, we have access to special variables, corresponding to the slot names for the intent. In the present case, the `ActivateLightColor` has two slots, `objectLocation` and `objectColor`.

### Sending TTS Notifications

You can send TTS notifications to Snips using the snips.say and snips.say_action services. Say_action starts a session and waits for user response, "Would you like me to close the garage door?", "Yes, close the garage door".

#### {% linkable_title Service `snips/say` %}

| Service data attribute | Optional | Description                                            |
|------------------------|----------|--------------------------------------------------------|
| `text`                 |       no | Text to say.                                           |
| `site_id`              |      yes | Site to use to start session.                          |
| `custom_data`          |      yes | custom data that will be included with all messages in this session. |

#### {% linkable_title Service `snips/say_action` %}

| Service data attribute | Optional | Description                                            |
|------------------------|----------|--------------------------------------------------------|
| `text`                 |       no | Text to say.                                           |
| `site_id`              |      yes | Site to use to start session.                          |
| `custom_data`          |      yes | custom data that will be included with all messages in this session. |
| `can_be_enqueued`      |      yes | If True, session waits for an open session to end, if False session is dropped if one is running. |
| `intent_filter`        |      yes | Array of Strings - A list of intents names to restrict the NLU resolution to on the first query. |

#### Configuration Examples

```yaml
script:
  turn_on_light:
    sequence:
      service: script.turn_on_light
      service: snips.say
        data:
          text: 'OK, the light is now on'

automation:
  query_garage_door:
    trigger:
     - platform: state
        entity_id: binary_sensor.my_garage_door_sensor
        from: 'off'
        to: 'on'
        for:
          minutes: 10
    sequence:
      service: snips.say_action
        data:
          text: 'Garage door has been open 10 minutes, would you like me to close it?'
          intentFilter:
            - closeGarageDoor

# This intent is fired if the user responds with the appropriate intent after the above notification
intent_script:
  closeGarageDoor:
    speech:
      type: plain
      text: 'OK, closing the garage door'
    action:
      - service: script.garage_door_close
```
