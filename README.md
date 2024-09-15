# UniFi Webhook Integration with Home Assistant
This page describes how to use UniFi Protect motion detection events, or alarms, to trigger automations in Home Assistant.

Written by Patrick Felstead, September 2024, after extensive trial and error configuring UniFi Protect to turn on outside lights—controlled by my home automation system, Home Assistant—when the cameras detect people using "AI Smart Detections."

## Overview
In UniFi Protect's Alarm Manager, each motion event is configured to **send** (push) a webhook to Home Assistant (HA). Home Assistant passively listens for the incoming webhook and triggers an automation when it receives it. Since a webhook can trigger only one automation, I use the automation to turn on an `input_boolean` toggle switch for 10 seconds.

### Reasons to use `input_boolean` switches:
- **Flexibility**: By using an `input_boolean`, you can decouple the webhook from the actual action, allowing multiple automations to be triggered based on the same event.
- **State history**: Home Assistant stores the history of the `input_boolean` in a form that's easier to view than an automation's history.
- **Conditional actions**: You can use the `input_boolean` in multiple automations to perform different actions based on the same event. For example, turning on lights only when it is dark outside or triggering other devices based on the motion event.
- **Reusability**: Once the `input_boolean` is set up, it can be reused in various parts of your Home Assistant setup, saving time and effort for future automations.
- **Timing control**: By turning on the `input_boolean` for a set duration (like 10 seconds), you can display the `input_boolean` on the front end and observe the toggle changing state when the webhook is received.
- **Prevent multiple triggers**: It helps prevent multiple automations from being triggered in rapid succession from repeated motion events, as the `input_boolean` serves as a short-term gate.
