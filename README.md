# UniFi Protect Integration with Home Assistant using Webhooks
This page describes how to use UniFi Protect motion detection events, or alarms, to trigger automations in Home Assistant.

Written by Patrick Felstead, September 2024, after extensive trial and error configuring UniFi Protect to turn on outside lights—controlled by my home automation system, Home Assistant—when the cameras detect people using "AI Smart Detections."

## Overview
In UniFi Protect's Alarm Manager, each motion event is configured to **send** (push) a webhook to Home Assistant (HA). Home Assistant passively listens for the incoming webhook and triggers an automation when it receives it. Since a webhook can trigger only one automation, I use the automation to turn on an `input_boolean` toggle switch for 10 seconds.

Since I currently have five cameras, I've set up five alarms, where each alarm is for detection of a person.

### Reasons to use `input_boolean` switches:
- **Flexibility**: By using an `input_boolean`, you can decouple the webhook from the actual action, allowing multiple automations to be triggered based on the same event.
- **State history**: Home Assistant stores the history of the `input_boolean` in a form that's easier to view than an automation's history.
- **Conditional actions**: You can use the `input_boolean` in multiple automations to perform different actions based on the same event. For example, turning on lights only when it is dark outside or triggering other devices based on the motion event.
- **Reusability**: Once the `input_boolean` is set up, it can be reused in various parts of your Home Assistant setup, saving time and effort for future automations.
- **Timing control**: By turning on the `input_boolean` for a set duration (like 10 seconds), you can display the `input_boolean` on the front end and observe the toggle changing state when the webhook is received.
- **Prevent multiple triggers**: It helps prevent multiple automations from being triggered in rapid succession from repeated motion events, as the `input_boolean` serves as a short-term gate.

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

