---
layout: post
title: Creating a Birthday Teams Chat Bot with Power Automate
categories: [Automate, Teams, Chat, Bot]
tags: [Automate, Teams, Chat, Bot]
comments: true
---

For those following me on <a href="https://twitter.com/sstranger/status/1502744581707911172" target="_blank">Twitter</a> I promised to write a blog post about creating a Birthday Teams Chat Bot with Power Automate when I got more than 15 likes and that happened pretty fast, so here is the blog post. Hope you enjoy it.

<!-- TOC start -->
- [Scenario](#scenario)
- [Automation Flow](#automation-flow)
  - [1. When keywords are mentioned](#1-when-keywords-are-mentioned)
  - [1 & 2 Configuring variables](#1--2-configuring-variables)
  - [4. Compose - Get random Giphy Birthday url](#4-compose---get-random-giphy-birthday-url)
  - [5. Compose - Output Giphy Birthday url](#5-compose---output-giphy-birthday-url)
  - [6. HTTP - Get What happened today REST API](#6-http---get-what-happened-today-rest-api)
  - [6. Parse JSON - Parse content from What happened today REST API](#6-parse-json---parse-content-from-what-happened-today-rest-api)
  - [8. Apply to each - Find Dutch people birthdays](#8-apply-to-each---find-dutch-people-birthdays)
  - [9. Compose - Get messageid property from first Teams Chat message](#9-compose---get-messageid-property-from-first-teams-chat-message)
<!-- TOC end -->


# Scenario

I don't know about you but within our Microsoft team we have a Microsoft Teams chat group where we communicate within our team about all kind of topics but it's also used to congratulate people on their birthdays. Often one congratulates a collegueue and then the rest of the team also congratulates that team member. 

One time it was jokingly mentioned that the initial birthday congratulation was automated, and that got me thinking. Why not try to automate my birthday congratulation Teams Group Chat message when someone congratulates someone within our Microsoft Teams Chat Group?

# Automation Flow

When someone congratulates someone using some keywords like "birthday" or "bday", I want to create an automated birthday congratulation Teams Group Chat message using a <a href="https://powerautomate.microsoft.com/" target="_blank">Power Automate Flow</a>.

![Example Power Automate Flow](/assets/03-13-2022-01.png)

To get above results I created the following Power Automate Flow.

![PowerAutomate Flow diagram](/assets/03-13-2022-02.png)

Let's go step by step through each of the steps in the Flow.

## 1. When keywords are mentioned

The trigger for this Flow is an <a href="https://docs.microsoft.com/connectors/teams/#when-keywords-are-mentioned" target="_blank">action that creates a webhook for keyword mentions in chats or channels</a>.

I configured the following for the trigger of this Flow.

![Step 1 in the Flow](/assets/03-13-2022-03.png)

Configure the keywords to have this Flow being triggered on. In my case I choose the keywords: "birthday" and "bday".

## 1 & 2 Configuring variables

Steps 1 & 2 are the initialization of 2 variables.

The first variable is storing an list (array) of Giphy birthday animated urls. Each time the Flow is being triggered a random object (url) is selected from this list.

You can create your own list of Giphy urls or use below one I created.

![Initalize variable action](/assets/03-13-2022-04.png)

```json
[
  "https://media.giphy.com/media/l4KibWpBGWchSqCRy/giphy.gif",
  "https://media.giphy.com/media/WRL7YgP42OKns22wRD/giphy.gif",
  "https://media.giphy.com/media/26BRtW4zppWWjrsPu/giphy.gif",
  "https://media.giphy.com/media/VX5pqkR3E6EmlKFlmq/giphy.gif",
  "https://media.giphy.com/media/eDSnmeQ4MWmB2/giphy.gif",
  "https://media.giphy.com/media/Kg2tFStNdUsOmxv2GC/giphy.gif",
  "https://media.giphy.com/media/kaBuCyQLuCANfmoFMC/giphy.gif",
  "https://media.giphy.com/media/Dn5nT3gtXXqiRysvK2/giphy.gif"
]
```


The second variable being initialized is used for storing a string containing all the Dutch people who are born on a the same day and month as the the person being mentioned in the happy birthday Teams chat message.

![Initialize variable action](/assets/03-13-2022-05.png)

## 4. Compose - Get random Giphy Birthday url

Within this action I'm selecting a random Giphy url from one of the urls configured in the variable called varBirthDayGifUrls.

![Get random url expression](/assets/03-13-2022-06.png)

The following Expression is used to get a random url.

```bash
variables('varBirthdayGifUrls')?[rand(0,length(variables('varBirthdayGifUrls')))]
```
## 5. Compose - Output Giphy Birthday url

This action creates an url object that is being used in step 16 to create the Teams Group chat message content.

![Compose - Output Giphy Birthday url](/assets/03-13-2022-07.png)

## 6. HTTP - Get What happened today REST API

![HTTP - Get What happened today REST API Action](/assets/03-13-2022-08.png)

The <a href="https://history.muffinlabs.com/" target="_blank">Today in History REST API</a> returns some basic historical information for any given day. This data is coming from <a href="https://en.wikipedia.org/wiki/List_of_days_of_the_year" target="_blank">Wikipedia's list of historical anniversaries</a>.

This list also contains some birthday information which I want to parse for Dutch people.

When filtered the following json result is returned for 13th of March.

```json
[
  {
    "year": "1560",
    "text": "William Louis, Count of Nassau-Dillenburg, Dutch count (d. 1620)",
    "html": "1560 - <a href=\"https://wikipedia.org/wiki/William_Louis,_Count_of_Nassau-Dillenburg\" title=\"William Louis, Count of Nassau-Dillenburg\">William Louis, Count of Nassau-Dillenburg</a>, Dutch count (d. 1620)",
    "no_year_html": "<a href=\"https://wikipedia.org/wiki/William_Louis,_Count_of_Nassau-Dillenburg\" title=\"William Louis, Count of Nassau-Dillenburg\">William Louis, Count of Nassau-Dillenburg</a>, Dutch count (d. 1620)",
    "links": [
      "@{title=William Louis, Count of Nassau-Dillenburg; link=https://wikipedia.org/wiki/William_Louis,_Count_of_Nassau-Dillenburg}"
    ]
  },
  {
    "year": "1967",
    "text": "Pieter Vink, Dutch footballer and referee",
    "html": "1967 - <a href=\"https://wikipedia.org/wiki/Pieter_Vink\" title=\"Pieter Vink\">Pieter Vink</a>, Dutch footballer and referee",
    "no_year_html": "<a href=\"https://wikipedia.org/wiki/Pieter_Vink\" title=\"Pieter Vink\">Pieter Vink</a>, Dutch footballer and referee",
    "links": [
      "@{title=Pieter Vink; link=https://wikipedia.org/wiki/Pieter_Vink}"
    ]
  },
  {
    "year": "1973",
    "text": "Edgar Davids, Surinamese born Dutch international footballer midfielder and manager",
    "html": "1973 - <a href=\"https://wikipedia.org/wiki/Edgar_Davids\" title=\"Edgar Davids\">Edgar Davids</a>, Surinamese born Dutch international footballer midfielder and manager",
    "no_year_html": "<a href=\"https://wikipedia.org/wiki/Edgar_Davids\" title=\"Edgar Davids\">Edgar Davids</a>, Surinamese born Dutch international footballer midfielder and manager",
    "links": [
      "@{title=Edgar Davids; link=https://wikipedia.org/wiki/Edgar_Davids}"
    ]
  },
  {
    "year": "1988",
    "text": "Furdjel Narsingh, Dutch footballer",
    "html": "1988 - <a href=\"https://wikipedia.org/wiki/Furdjel_Narsingh\" title=\"Furdjel Narsingh\">Furdjel Narsingh</a>, Dutch footballer",
    "no_year_html": "<a href=\"https://wikipedia.org/wiki/Furdjel_Narsingh\" title=\"Furdjel Narsingh\">Furdjel Narsingh</a>, Dutch footballer",
    "links": [
      "@{title=Furdjel Narsingh; link=https://wikipedia.org/wiki/Furdjel_Narsingh}"
    ]
  },
  {
    "year": "1998",
    "text": "Jay-Roy Grot, Dutch footballer",
    "html": "1998 - <a href=\"https://wikipedia.org/wiki/Jay-Roy_Grot\" title=\"Jay-Roy Grot\">Jay-Roy Grot</a>, Dutch footballer",
    "no_year_html": "<a href=\"https://wikipedia.org/wiki/Jay-Roy_Grot\" title=\"Jay-Roy Grot\">Jay-Roy Grot</a>, Dutch footballer",
    "links": [
      "@{title=Jay-Roy Grot; link=https://wikipedia.org/wiki/Jay-Roy_Grot}"
    ]
  }
]
```

But back to the action, at this time the complete list of events for today is being retrieved. Filtering happens later in the Flow.

## 6. Parse JSON - Parse content from What happened today REST API

This action parses the JSON content retrieved in step 5.

![Parse JSON - Parse content from What happened today REST API action](/assets/03-13-2022-09.png)

You can easily create the schema using the Generate from sample option, after copying the <a href="https://history.muffinlabs.com/date" target="_blank">results from the REST API</a>.

## 8. Apply to each - Find Dutch people birthdays

In this step we want to filter the parsed JSON results for Dutch people, and when this is true add the text result to the variable varDutchPeopleBirthdays.

![Apply to each - Find Dutch people birthdays action](/assets/03-13-2022-10.png)

With the following expression we are filtering on all people who are born on today's date for the keyword "Dutch"

```bash
contains(items('Apply_to_each_-_Find_Dutch_people_birthdays').text,'Dutch')
```

If the condition is true then we want to store the text properties in the variable varDutchPeopleBirthdays with the following expression:

```bash
concat('<li>',items('Apply_to_each_-_Find_Dutch_people_birthdays').text,decodeUriComponent('%0A'),'</li>')
```

This generates a string html list of people which can be used in the Team Group Chat message action later on.

## 9. Compose - Get messageid property from first Teams Chat message

When the Flow is being triggered a list of Teams Group chat messages is being retrieved for the keywords selected. Even when only one Teams Group Chat message is being retrieved this is stored as an array.

In this action we need to retrieve the messageid for the Chat where the keyword(s) are used.

Example output of When keywords are mentioned trigger:

```json
{
    "value": [
      {
        "subscriptionId": "d9e5d70c-f932-4c0f-8b35-ca68514dcabe",
        "changeType": "created",
        "clientState": "secretClientValue",
        "subscriptionExpirationDateTime": "2022-03-13T14:17:07.9411301+00:00",
        "resource": "chats('19:70e0bb050123486cbf100a5bac7a937d@thread.v2')/messages('1647178904469')",
        "resourceData": {
          "id": "1647178904463",
          "@odata.type": "#Microsoft.Graph.chatMessage",
          "@odata.id": "chats('19:70e0bb050123486cbf100a5bac7a937d@thread.v2')/messages('1647178904469')"
        },
        "tenantId": "9fbf4a30-64b3-4f54-81fa-b71f3a6971b1",
        "conversationId": "19:70e0bb050123486cbf100a5bac7a937d@thread.v2",
        "messageId": "1647178904463",
        "linkToMessage": "https://teams.microsoft.com/l/message/19:70e0bb050123486cbf100a5bac7a937d@thread.v2/1647178904463?tenantId=9fbf4a30-64b3-4f54-81fa-b71f3a6971b1&context=%7B%22contextType%22:%22chat%22%7D"
      }
    ]
  }
```

![Compose - Get messageid property from first Teams Chat message action](/assets/03-13-2022-11.png)

Expression being used:

```bash
first(triggerOutputs()?['body/value'])?['messageId']
````

## 10. Apply to each

In this action we are finally sending the Teams Group chat message.

First we need to get the details of the Teams Group chat message containing the keyword(s) "birthday" or "bday".

Then we have a condition to check if this chat message is being send by someone. We don't want to respond on everyone who is sending a birthday message in the Teams Group Chat.

![Apply to each Action](/assets/03-13-2022-12.png)

If the condition is meet we want to send the Chat message.

![Apply to each Action. Part 2](/assets/03-13-2022-13.png)

Action number 14 Compose - Get Mention Id action is being used to retrieve the mention id (who is congratulated with their Birthday?) so that we can get a mention token (action 15) in the next step to be able to mention someone in the final Teams Group Chat message.

The <a href="https://docs.microsoft.com/connectors/teams/#post-message-in-a-chat-or-channel" target="_blank">Post message in a chat or channel </a>action (number 16) posts a message to a chat or a channel.

```html
@{outputs('Get_an_@mention_token_for_a_user')?['body/atMention']} Congratulations with your birthday!
<p>
<img src="@{outputs('Compose_-_Output_Giphy_Birtday_url')}" width="275" height="250" alt="Gihpy birtday image (GIF Image)" style="padding-top:5px">
<p>
<p>Interesting fact.</p>
<p>Did you know, that the following Dutch people are also celebrating their birthdays on exact this date,</p>
@{variables('varDutchPeopleBirthdays')}
<p>
<p>Have a great day!</p>
<p>
<p>Stefan Stranger</p>
</p>
```


I hope this is sufficient information to build your own Power Automate Flow using some of the logic I described in this blog post.

I would like to thank <a href="https://twitter.com/JanVidarElven" target="_blank">Jan Vidar Elven</a> for helping me some of the questions I had during the creation of the Power Automate Flow.




