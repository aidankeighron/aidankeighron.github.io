---
layout: post
title: Sheet Scraper
date: 2022-12-7 12:00:00
categories: [python, FRC]
---

# Sheet Scraper
* The code shown here has been modified to make it easier to explain. If you will like to check it out, the full code is on GitHub: [Sheet Scraper](https://github.com/SwervyK/Sheet-Scraper){:target="\_blank"}

I wrote this program to help the FRC shirt trading community by making a spreadsheet that is a combination of members' spreadsheets. If you are not familiar with FRC or shirt trading you probably did not understand a word of that sentence so I will give you some explanation.

## FRC

First Robotics Competition is a high school competitive robotics league. It has around +3500 teams across +25 countries. Teams will design, build, wire, and program a robot to compete in 3v3 matches against an opposing team. The way you score points each year is different so here is an example of the [2022 game](https://www.youtube.com/watch?v=LgniEjI9cCM).

## Shirt Trading

FRC is modeled like conventional sports, teams have mascots and team shirts. FRC does differ from conventional sports because it promotes cooperation and gracious professionalism at competitions which results in collaboration between teams. One example of this collaboration is shirt trading. Teams will make extra shirts to trade with at competitions or online. Shirts have different values depending on the team/s that are on them and how cool they are. There is a Discord server for shirt trading with channels for posting spreadsheets of shirt inventories.

## Requirements

Now let's talk about the requirements to run this scraper:

- [Blue alliance API key (optional)](https://www.thebluealliance.com/apidocs){:target="\_blank"}

Blue alliance is used to get the name of a team when only the number has been provided
- [Discord bot token (only if channel is a forum channel)](https://docs.discordbotstudio.org/setting-up-dbs/finding-your-bot-token){:target="\_blank"}

A discord bot is used to scrape the spreadsheet data for forum channels (if it is a text channel we can used the request library)
- [Account credentials from a google cloud project with access to google sheets API](https://developers.google.com/sheets/api){:target="\_blank"}

Used to get data from the spreadsheets and to write data into the combined spreadsheet

## Getting spreadsheets from discord

### **Text Channel**

The first step in creating a combined spreadsheet is getting all of the individual sheets:

```python
def retrieve_messages(channelID):
    headers = {
        "authorization": "Bot " + str(keys.DISCORD_AUTH)
    }
    r = requests.get(f"https://discord.com/api/v9/channels/{channelID}/messages?limit=100", headers=headers)
    data = json.loads(r.text)
    return data
        
def get_sheet_id(data):
    ids = []
    user = []
    for value in data:
        try:
            id = value["content"].split("/d/")
            id[1] = id[1].split("/edit")
            ids.append(id[1][0])
            user.append(value['author']['username'])
        except:
            ...
    return (ids, user)

def get_ids():
    return get_sheet_id(retrieve_messages(keys.CHANNEL_ID))
```

Let's start with `retrieve_messages`. We pass in the discord channel we want to get the messages of and using the requests we get the channel data from the Discord API. Next we send all of the channel data to `get_sheet_id` where we loop though every message and extract the spreadsheet id if it exists.

### **Forums Channel**

Recently the FRC Trading Discord has updated their spreadsheet channel to convert it into a forums channel, so our code needs to reflect the change:

```python
client = discord.Client(intents=discord.Intents.default())
messages = []
users = []

@client.event
async def on_ready():
    firstTime = True
    for guild in client.guilds:
        for channel in guild.channels:
            if channel.name == "spreadsheets":
                if firstTime:
                    firstTime = False
                    continue

                for thread in channel.threads:
                    async for message in thread.history(limit=100):
                        if "spreadsheets/d/" in message.content:
                            id = message.content.split("/d/")
                            id[1] = id[1].split("/edit")
                            messages.append(id[1][0])
                            users.append(message.author.name)
                            break
                break
    await client.close()
                


def get_forum_ids():
    client.run(keys.DISCORD_AUTH)
    return (messages, users)
```

To get the spreadsheet id's from a forums channel we need to access it with discord's python library. Here we create the discord client and when we get the id's from `get_forum_ids` we run a discord bot to get every post from the forums channel and any spreadsheet id's they contain.

# 

Full code: [Sheet Scraper](https://github.com/SwervyK/Sheet-Scraper){:target="\_blank"}