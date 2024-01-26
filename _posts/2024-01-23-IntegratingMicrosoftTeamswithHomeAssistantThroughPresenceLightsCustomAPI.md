---
layout: post
title: Integrating Microsoft Teams with Home Assistant Through PresenceLight's Custom API
categories: [API, Home Assistant, Microsoft Teams, Automation]
tags: [API, Home Assistant, Microsoft Teams, Automation]
comments: true
---

- [Introduction](#introduction)
- [How to get started?](#how-to-get-started)
  - [Ulanzi Pixel Clock](#ulanzi-pixel-clock)
  - [Home Assistant](#home-assistant)
- [References:](#references)


# Introduction

I was recently introduced to [PresenceLight](https://github.com/isaacrlevin/presencelight) via my Microsoft [Home Assistant](https://www.home-assistant.io/) friends.

PresenceLight is a solution to broadcast your various statuses to various kinds of smart lights. Some statuses you can broadcast are: your availability in Microsoft Teams or color of your choosing. There are other solutions that do something similar to sending Teams Availability to a light, but they require a tethered solution (plugging a light into a computer via USB). What PresenceLight does is leverage the [Presence Api](https://docs.microsoft.com/graph/api/presence-get), which is available in [Microsoft Graph](https://docs.microsoft.com/graph/overview), allowing to retrieve your presence without having to be tethered. This could potentially allow someone to update the light bulb from a remote machine they do not use.

When I investigated PresenceLight I noticed that it didn't supported my [IKEA Smart Lights](https://www.ikea.com/nl/en/cat/smart-lighting-kits-36815/). But it does have a Custom API which I could use.

After further investigation I found out that this Custom API feature did not have the option to provide a JSON body to the REST API calls made to configured endpoints 😒

<img src="/assets/2024-01-23-01.png" alt="Image of configuration screen of Custom API settings from Presence Light without the option to provide a body" width="1000"/>

This was a small disappointment because most of the REST API calls also require a body to be send. So I contacted the developer of PresenceLight, Isaac Levin via a [Github issue](https://github.com/isaacrlevin/presencelight/issues/848) and asked if he could add support for providing a body to the Custom API REST API call functionality.

Isaac responded that he was happy to review a PR for this new functionality to be added.  
  
 ![Meme with a male person, and the text "That's not to e unexpected"](/assets/2024-01-23-02.gif)

The only thing left to do was accept this challenge, especially because I'm not a real developer. I call myself a wannabee Developer. But with the help of [Github Copilot](https://github.com/features/copilot) I was able to add this functionality to PresenceLight🎉🙏

In the rest of this blog post I'm going to explain how you can use this improved Custom API feature of PresenceLight with Home Assistant and the [Ulanzi Pixel Clock](https://www.ulanzi.com/collections/clock).

# How to get started?

You first need to install the latest version of the PresenceLight solution, from one of the <a href="https://github.com/isaacrlevin/presencelight?tab=readme-ov-file#get-presencelight" target="_blank">locations documented on the website of PresenceLight.</a> The easiest installation is probably from the <a href="https://apps.microsoft.com/detail/9NFFKD8GZNL7?hl=en-us&gl=US" target="_blank">Microsoft Store</a>.

After the installation you need to figure out how the REST API call from PresenceLight needs to look like to your Home Assistant, Ulanzi Pixel Clock or whatever service you want to trigger with your Microsoft Teams status from PresenceLight.

## Ulanzi Pixel Clock

I already installed <a href="https://github.com/Blueforcer/awtrix-light" target="_blank">Awtrix-Light</a> (AWL) on my Ulanzi Pixel clock to do more fun things with it.

AWL meant to be a companion for your smarthome like HomeAssistant, IOBroker, FHEM, NodeRed and so on.

AWL supports both <a href="https://blueforcer.github.io/awtrix-light/#/api" target="_blank">MQTT and HTTP API</a>. For the usage together with PresenceLight I used the HTTP API functionality.

If we want to send the Microsoft Team Status using PresenceLight to the Ulanzi Pixel clock we can use the following REST API call.

| HTTP URL               | Payload/Body | HTTP Method |
|------------------------|--------------|-------------|
| http://[IP]/api/notify | JSON         | POST        |

Example JSON body:

```json
{   "text": "Teams Status is Available",   "rainbow": true,   "duration": 10 }
```

This will display the text "Teams Status is Available" in rainbow colors for 10 seconds.

The PresenceLight Custom API also supports the usage of the following variables in the Custom API JSON body.

| Variables | Value |
|-|-|
| `{`{availability`}`} | Graph API Teams Availability Status |
| `{`{activity`}`} | Graph API Teams Activity Status |

```json
{   "text": "Teams Status is \/{\/{availability\/}\/}'",   "rainbow": true,   "duration": 10 }
```

The next step is configure the Custom API settings of PresenceLight with the Uri and JSON Body information.

<img src="/assets/2024-01-23-03.png" alt="PresenceLight Custom API setting pane, showing the configuration of the REST API call to the Ulanzi Pixel Clock" width="1000"/>

In PresenceLight Custom API setting you need to enter the following information:

| Method | Uri            | Body |
|--------|----------------|------|
| POST   | http://[IP]/api/notify|    {   "text": "Teams Status is {{availability}}",   "rainbow": true,   "duration": 10 }  |


And how can use this with Home Assistant?

## Home Assistant

To use PresenceLight with Home Assistant you can use the Custom API functionality as follows:

In Home Assistant you can use <a href="https://www.home-assistant.io/docs/automation/trigger/#webhook-trigger" target="_blank">Webhooks triggers</a> to trigger an Automation Action, like turning on a light bulb.

Example Automation for turning on a light bulb based on the Teams status send using the Custom API functionality of PresenceLight.

```yaml
alias: Teams presence - IKEA Light Bulb Living Room
description: >-
  Show the Microsoft Teams status via a color of the Light Bulb in the Living
  Room
trigger:
  - platform: webhook
    allowed_methods:
      - POST
    local_only: true
    webhook_id: "<enter secret webhook id here>"
condition: []
action:
  - choose:
      - conditions:
          - condition: template
            value_template: "{{ trigger.json.presence_status == 'Busy' }}"
        sequence:
          - service: light.turn_on
            metadata: {}
            data:
              color_name: red
            target:
              entity_id: light.ikea_bulb
      - conditions:
          - condition: template
            value_template: "{{ trigger.json.presence_status == 'Available' }}"
        sequence:
          - service: light.turn_on
            metadata: {}
            data:
              color_name: green
            target:
              entity_id: light.ikea_bulb
      - conditions:
          - condition: template
            value_template: "{{ trigger.json.presence_status == 'Away' }}"
        sequence:
          - service: light.turn_on
            metadata: {}
            data:
              color_name: yellow
            target:
              entity_id: light.ikea_bulb
      - conditions: null
        sequence:
          - service: light.turn_off
            metadata: {}
            target:
              entity_id: light.ikea_bulb
            data: {}
mode: single
```

In PresenceLight Custom API setting you need to enter the following information:

| Method | Uri            | Body |
|--------|----------------|------|
| POST   | http://homeassistant.local:8123/api/webhook/webhook_id |    {   "presence_status":"Away" }  |

This is how simple it now is to use the PresenceLight Custom API and a JSON Body.

![Meme with the text "Oh Yeah, Oh Yeah"](/assets/2024-01-23-04.gif)

# References:

- [PresenceLight](https://github.com/isaacrlevin/presencelight)
- [Home Assistant](https://www.home-assistant.io/) 
- [Awtrix-Light](https://github.com/Blueforcer/awtrix-light)