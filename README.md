# UniFi Protect Integration with Home Assistant using Webhooks
This page describes how to use UniFi Protect motion detection events, or alarms, to trigger automations in Home Assistant.

Written by Patrick Felstead, September 2024, after some trial and error configuring UniFi Protect to turn on outside lights—controlled by my home automation system, Home Assistant—when the cameras detect people using "AI Smart Detections."

## Overview
In UniFi Protect's Alarm Manager, each motion event is configured to **send** (push) a webhook to Home Assistant (HA). Home Assistant passively listens for the incoming webhook and triggers an automation when it receives it. Since a webhook can trigger only one automation, I use the automation to turn on an `input_boolean` toggle switch for 10 seconds. The `input_boolean` is then used to trigger other automations, including multiple automations if needed.

Since I currently have five cameras, I've set up five alarms in Protect, where each alarm is for detection of a person. This triggers five separate webhook automations, which set five `input_boolean` helpers.

### Reasons to use `input_boolean` switches:
- **Flexibility**: By using an `input_boolean`, you can decouple the webhook from the automations that do the real work, allowing multiple automations to be triggered based on the same event.
- **State history**: Home Assistant stores the history of the `input_boolean` in a form that's easier to view than an automation's history.
- **Timing control**: By turning on the `input_boolean` for a set duration (like 10 seconds), you can display the `input_boolean` on the front end and observe the toggle changing state when the webhook is received.
- **Prevent multiple triggers**: It helps prevent multiple automations from being triggered in rapid succession from repeated motion events, as the `input_boolean` serves as a short-term (10 second) gate.


## HA Automations
Home Assistant Automations are shown below (the icons under the name are just HA labels to help identify the automations in the long list of automations). They are also undera a HA category "Unifi camera webhooks", which is like a folder for HA automations).

**HA Automataions triggered by webhooks**

<img width="1423" alt="image" src="https://github.com/user-attachments/assets/c83c0e9a-3954-4d43-97d8-65aea1335043">

**Content of one HA automation**
<img width="1268" alt="image" src="https://github.com/user-attachments/assets/d34a61dc-b503-4505-9d1e-9cf384283355">

As you can see, after the automation is triggered ("When") by the webhook, there are no conditions ("And if"). The actions ("Then do") are to:
- Turn ON the `input_boolean`
- Wait 10 seconds
- Turn OFF the `input_boolean`

While developing the automation, you can test it by sending your phone a notification. I've disabled that action, but left it in the automation in case I need to check something later on.

Here's what I sent to my phone, this displays the `triggers : key` part of webhook payload json data (see below for an example)
```yaml
action: notify.mobile_app_patricks_iphone_13
metadata: {}
data:
  title: Unifi CCTV Camera
  message: >
    Back Yard: {% set triggers = trigger.json.alarm.triggers %}
    {% for trigger in triggers -%}
      {{ trigger.key }}
    {% endfor %}
enabled: true
```
So on your phone you'll see a notification with title `Unifi CCTV Camera` and content `Back Yard: person`.

If you'd rather just display the complete json text (as shown below), you'd write the notification as:
```yaml
action: notify.mobile_app_patricks_iphone_13
metadata: {}
data:
  title: Unifi CCTV Camera
  message: >
    Back Yard: {{ trigger.json }}
enabled: true
```

An example of the webhook json data content received by HA from Unifi Protect when a person is detected. For a line crossing, `person` will be replaced with `line_crossing`
```json
{
  "alarm": {
    "name": "Pool Alfresco Person",
    "sources": [{ "device": "F4E2C60E6104", "type": "include" }],
    "conditions": [
      { "condition": { "type": "is", "source": "person" } },
      { "condition": { "type": "is", "source": "animal" } }
    ],
    "triggers": [
      { "key": "person", "device": "F4E2C60E6104" },
      { "key": "person", "device": "F4E2C60E6104" }
    ]
  },
  "timestamp": 1725883107267
}
```

**Details of the webhook trigger**

The image below shows the yaml version of the webhook automation trigger. Note that I've changed the webhook ID for this readme as the IDs should be treated like passwords.

<img width="1266" alt="image" src="https://github.com/user-attachments/assets/c1d3c1eb-1445-4d9f-86b3-3965f7653d74">


Change the HA automation mode to `single` to prevent additional triggers if another webhook is pushed during the 10 second delay.

<img width="317" alt="image" src="https://github.com/user-attachments/assets/a2a4806b-9bed-4049-a0fa-4e2f4925bb5a">

<img width="406" alt="image" src="https://github.com/user-attachments/assets/18c308d3-7c4c-4c3b-aebb-943e1cd2c1a6">




**Input Booleans as displayed on HA front end**
`input_boolean` are actually created using the HA UI under the "Helpers" menu (i.e. you don't need to define them in yaml in your config.yaml file).

<img width="495" alt="image" src="https://github.com/user-attachments/assets/a3e6c59d-0998-4fa6-9824-033deb3b3d12">

Like I said previously, you can then use the `input_boolean` to trigger your actual HA automations that do something useful. You can write as many automations as you want to be triggered by each `input_boolean`.

## Unifi Protect Settings
### Cameras
<img width="644" alt="image" src="https://github.com/user-attachments/assets/32a7b300-c482-4f79-acfc-b1e718ab4838">

### Smart Detection Settinngs - Per Camera

<img width="393" alt="image" src="https://github.com/user-attachments/assets/97e8515f-caf7-47ec-af3b-69bdc0d31eb1">

<img width="393" alt="github readme unifi 1" src="https://github.com/user-attachments/assets/767652e9-36b8-4e22-ba0a-14504d7789bd">


I also use crossing lines to trigger the same alarm (webhook) because UniFi Protect does not generate an alarm until a motion event has fully completed. If a person enters the smart zone and stays there, or walks within the zone without leaving, the webhook isn't sent until the "ongoing" motion event ends.

However, by adding crossing lines, the alarm (webhook) is triggered immediately when the person crosses the line, even if they remain in the smart zone and the motion detection is still "ongoing."

**Smart Detection Zones editor**
<img width="1267" alt="image" src="https://github.com/user-attachments/assets/1331fabc-299e-4407-92ea-703540222969">

**Crossing Lines editor**
<img width="1267" alt="image" src="https://github.com/user-attachments/assets/6fea967f-b1de-43b7-95ae-5aad381b738b">


### Alarm Manager
![image](https://github.com/user-attachments/assets/954fb21d-736e-49f5-86bc-dcb0572e79b3)


**Alarm showing webhook configuration**

Alarm manager editor

<img width="389" alt="image" src="https://github.com/user-attachments/assets/8191a6b3-f38a-4dd6-8e63-30be3f9eeb90">

