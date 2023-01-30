---
layout: post
title: Sheet Scraper
date: 2023-1-13 12:00:00
categories: [python, FRC]
---

# Project Overview

I wrote this program to help the FRC shirt trading community by making a spreadsheet that is a combination of members' spreadsheets. If you are not familiar with FRC or shirt trading you probably did not understand a word of that sentence so I will give a brief overview.

The code shown here has been modified to make it easier to explain. If you would like to check it out, the full code is available on GitHub: [Sheet Scraper](https://github.com/SwervyK/Sheet-Scraper){:target="\_blank"}. If you would like to see the final result here is a link to the [spreadsheet](https://docs.google.com/spreadsheets/d/1xDtfRs81EcdFHOiG-Nua_mBz2xAD4tFVVtNt6SjwPoM/edit#gid=341185536){:target="\_blank"}.

<details>
<summary><b>Table of Contents</b></summary>
<ul>
<li><a href="#project-overview">Project Overview</a></li>
<li><a href="#requirements">Requirements</a></li>
<li><a href="#getting-links-from-discord">Getting Links From Discord</a></li>
<li><a href="#getting-the-spreadsheets">Getting the Spreadsheets</a></li>
<li><a href="#parsing-the-spreadsheet">Parsing the Spreadsheet</a></li>
<li><a href="#parsing-the-inventories">Parsing the Inventories</a></li>
<li><a href="#sorting">Sorting</a></li>
<li><a href="#saving-results">Saving Results</a></li>
<li><a href="#final-thoughts">Final Thoughts</a></li>
<li><a href="#bonus-counting-the-number-of-occurrences-each-shirt-has">Bonus</a></li>
</ul>
</details>

## FRC

First Robotics Competition is a high school competitive robotics league. It has around +3500 teams across +25 countries. Teams will design, build, wire, and program a robot to compete in 3v3 matches against an opposing team. The way you score points each year is different (Here is an example of the [2022 game](https://www.youtube.com/watch?v=LgniEjI9cCM)).

## Shirt Trading

FRC is modeled like conventional sports, teams have mascots and team shirts. FRC does differ from conventional sports because it promotes cooperation and gracious professionalism at competitions which results in collaboration between teams. One example of this collaboration is shirt trading. Teams will make extra shirts to trade at competitions or online. Shirts have different values depending on the team/s that are on them and how cool they are. There is a Discord server for shirt trading with channels for posting spreadsheets of shirt inventories.

## Requirements

Now let's talk about the requirements to run this scraper:

- [Blue alliance API key (optional)](https://www.thebluealliance.com/apidocs){:target="\_blank"}

Blue alliance is used to get the name of a team when only the number has been provided
- [Discord bot token](https://docs.discordbotstudio.org/setting-up-dbs/finding-your-bot-token){:target="\_blank"}

A discord bot is used to scrape the spreadsheet data from the FRC Shirt Trading discord server
- [Account credentials from a google cloud project with access to google sheets API](https://developers.google.com/sheets/api){:target="\_blank"}

Used to get data from the spreadsheets and to write data into the combined spreadsheet

## Getting Links From Discord

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

Let's start with `retrieve_messages`. We pass in the discord channel we want to get the messages of and using requests we get the channel data from the Discord API. Next we send all of the channel data to `get_sheet_id` where we loop though every message and extract the spreadsheet id if it exists.

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

# Getting the Spreadsheets

We use the google sheets api to get spreadsheet data.

```python
def get_sheet(id):
    result = service.spreadsheets().get(spreadsheetId=id, includeGridData=True).execute()
    result = service.spreadsheets().values().get(spreadsheetId=id, range=result['sheets'][0]['properties']['title'], majorDimension="ROWS").execute()
    if "values" in result:
        return result["values"]
    else:
        return -1
```

We use the first request to get the title of the first spreadsheet and the second request to get the contents with a check to make sure that the contents exist.

# Parsing The Spreadsheet

Our first step is to take a spreadsheet and locate the different category's.

<img src="/assets/icons/sheet-scraper/example_spreadsheet.PNG" style="border:5px solid black;display: block;margin-left: auto;margin-right: auto;width: 80%;">

In this example we can see we have categories for team number, team name, shirt type, shirt size, and how tradable it is. Now lets find each of those categories:

```python
start = -1
for _ in range(10):
    category_locations = {category:-1 for category in CATEGORIES}
    for row in range(start + 1, len(sheet)):
        for col in range(len(sheet[row])):
            if check_category(sheet[row][col]) != "none" and category_locations[check_category(sheet[row][col])] == -1:
                category_locations[check_category(sheet[row][col])] = col
                start = row
        if start == row: # Category's need to be in a row
            break
    categoriesFound = sum(1 for _, category in category_locations.items() if category != -1)
    if categoriesFound >= MIN_CATEGORIES:
        break
```

Lets break it down. First we initialize a `start` variable that we will use to keep track of where the spreadsheet starts.

```python
category_locations = {category:-1 for category in CATEGORIES}
for row in range(start + 1, len(sheet)):
    for col in range(len(sheet[row])):
```

We create an empty dictionary to store the colum numbers for each category. Next we start to loop through every row of the spreadsheet but first we need to figure out how to find out if a current index contains a category.

```python
def check_category(val):
    if val == "":
        return "none"
    val = val.lower()
    if "name" in val and "username" not in val:
        return "Name"
    if "number" in val or ("#" in val and "team" in val):
        return "Number"
    if "size" in val:
        return "Size"
    if "type" in val or "item" in val:
        return "Type"
    if "year" in val:
        return "Year"
    if "description" in val or "info" in val or "details" in val or "decription" in val:
        return "Description"
    if "status" in val or "avail" in val:
        return "Availability"
    if "trad" in val or "rarity" in val or "likeliness" in val:
        return "Tradability"
    if "notes" in val or "other" in val or "comments" in val:
        return "Notes"
    if "team" in val:
        return "Number"
    return "none"
```

Here is the `checkCategory` method we use to look for keywords in each index of the spreadsheet. Now lets look at how we loop through each row.

```python
if check_category(sheet[row][col]) != "none" and category_locations[check_category(sheet[row][col])] == -1:
    category_locations[check_category(sheet[row][col])] = col
    start = row
```

At each index we check to see if it contains a category by passing it into our `check_category` method as long as it does not return `"none"` it is a valid category. We also check to make sure that this category has not already been found. After those conditions have been met we save the category's colum number and the current row number.

```python
for row in range(start + 1, len(sheet)):
    for col in range(len(sheet[row])):
    if start == row:
        break
```

After we are done looping through the row we check to see if start has been reassigned which means that a row containing categories has been found. In every spreadsheet the categories are always in a row so there is no reason to continue looking. But what if the sheet has a header that accidentally contains a "category". 

```python
categoriesFound = sum(1 for _, category in category_locations.items() if category != -1)
if categoriesFound >= MIN_CATEGORIES:
    break
```

We use this to make sure that the category row we have found is the correct one. We add up all of the categories that don't have a default colum and check to make sure we have found enough `MIN_CATEGORIES = 3`.

```python
for _ in range(10):
```

We loop through this 10 times so that we don't waste time on sheets that don't contain any categories. After we loop through it 10 times we rotate the sheet 90 degrees `sheet = list(zip(*sheet[::-1]))` and loop through it another 10 times because some spreadsheets have their categories horizontal.

```python
end = -1
for row in range(len(sheet)):
    for col in range(len(sheet[row])):
        val = str(sheet[row][col]).lower()
        if "wish" in val or "wants" in val or "looking" in val:
            end = row if all([col <= index for _, index in category_locations.items()]) else -1
            break
    if end != -1:
        break
```

Finally we want to locate the end of the spreadsheet. Sometimes users will put a wishlist beneath their inventory so we use this to make sure we don't get the inventory shirts mixed up with the wishlist. `all([col <= index for _, index in category_locations.items()]) else -1` is used because sometimes the wishlist is to the right of the regular inventory and we don't want to end too soon. Wishlists are also part of the reason why we check if the category has not all ready been found `category_locations[check_category(sheet[row][col])] == -1`.

# Parsing the Inventories

Now that we know where each category is we can get all of the shirt data.

```python
def parse_sheet(sheet, start, end, user, id, category_locations):
    shirts = []
    empty = 0
    for row in range(start+1, len(sheet) if end == -1 else end):
        shirt = {category:"" for category in CATEGORIES}
        shirt["User"] = user
        shirt["ID"] = id
        for category, col in category_locations.items():
            if col == -1 or col >= len(sheet[row]):
                continue
            shirt[category] = sheet[row][col]
        if shirt["Name"] == "" and shirt["Number"] != "":
            try:
                shirt["Name"] = get_team_name(int(shirt["Number"]))
            except:
                ...
        numNotEmpty = sum(1 for _, col in shirt.items() if col != '')
        if numNotEmpty >= MIN_SHIRT_DATA:
            shirts.append(shirt)
            empty = 0
        empty += 1
        if empty > MAX_EMPTY:
            break
    return shirts
```

First we initialize `shirts` and `empty` which will keep track of each shirt we find and the number of consecutive empty rows respectively.

```python
for row in range(start+1, len(sheet) if end == -1 else end):
    shirt = {category:"" for category in CATEGORIES}
    shirt["User"] = user
    shirt["ID"] = id
```

Next we loop through each row of the sheet from `start` to `end` (if it is not -1). We also initialize shirt to be a dictionary of each category and we set the user and id categories.

```python
for category, col in category_locations.items():
    if col == -1 or col >= len(sheet[row]):
        continue
    shirt[category] = sheet[row][col]
```

In this section we are looping through the `category_locations` dictionary which contains each category and what column it is in. We check to make sure that the category has been found and that it is not out of range. Then we get the category value and assign it to the `shirt` dictionary at the category index.

```python
if shirt["Name"] == "" and shirt["Number"] != "":
    try:
        shirt["Name"] = get_team_name(int(shirt["Number"]))
    except:
        ...
```

Here we check to see if we have a team number but no team name. We then try to typecast the shirt number to make sure it is of type `int`. If shirt number is an int then we can find the team name using the Blue Alliance API. The Blue Alliance API is an API from a website that displays FRC teams info, rankings, and match record. We can use this to fill in the blanks when we are missing a team name.

```python
def get_team_name(number):
    try:
        name = team_names[str(number)]
        return name
    except:
        ...
    site = "https://www.thebluealliance.com/api/v3/team/frc"
    api = {"X-TBA-Auth-Key": keys.BLUE_ALLIANCE}
    request = requests.get(url=site+str(number), headers=api)
    team_names[str(number)] = request.json()['nickname']
    return request.json()['nickname']
```

Here we use a try catch loop to see if we already have the name cached. If we don't then we make a call to the API using the team number and then access the nickname from the json.

```python
numNotEmpty = sum(1 for _, col in shirt.items() if col != '')
if numNotEmpty >= MIN_SHIRT_DATA:
    shirts.append(shirt)
    empty = 0
empty += 1
if empty > MAX_EMPTY:
    break
```

Finally before we append the `shirt` dictionary to `shirts` we need to verify it. We count up how many empty categories we have and if it i greater than the min `MIN_SHIRT_DATA = 5` we add it to `shirts`. If it is empty we increment `empty` and if there are too many empty shirts in a row then we stop parsing the spreadsheet `MAX_EMPTY = 10`.

# Sorting

Once we have a complete list of all of the shirts we, need to sort them.

```python
def sort_sheet(shirts):
    nonInt = {}
    currentInt = 999999999
    for row in shirts:
        try:
            row[1] = int(row[1])
        except:
            nonInt[str(currentInt)] = row[1]  
            row[1] = currentInt
            currentInt += 1          
    
    shirts.sort(key=lambda x: int(x[1]))
    
    for row in shirts:
        try:
            if row[1] >= 999999999:
                row[1] = nonInt[str(row[1])]
        except Exception as e:
            print(str(e))
    return shirts
```

Because there is no guarantee that the number colum of the spreadsheet is a valid integer we need to reassign it so we can sort it numerically by team number.

```python
for row in shirts:
    try:
        row[1] = int(row[1])
    except:
        nonInt[str(currentInt)] = row[1]  
        row[1] = currentInt
        currentInt += 1
```

Here we check to see if the current index is an integer, if not we assign it a unique value so we can sort everything later.

```python
shirts.sort(key=lambda x: int(x[1]))

for row in shirts:
    try:
        if row[1] >= 999999999:
            row[1] = nonInt[str(row[1])]
    except Exception as e:
        print(str(e))
```

Now we sort the shirts and reassign them using the placeholder vales. 

# Saving Results

Now that we have a complete sorted list of every shirt we can save them to our results spreadsheet.

```python
def write_result(sheet, id):
    CATEGORIES.append("User")
    CATEGORIES.append("ID")
    sheet.insert(0, CATEGORIES)
    values = {"majorDimension": "ROWS", "range": "Sheet1", "values": sheet}
    result = service.spreadsheets().values().update(spreadsheetId=id, range="Sheet1", valueInputOption="RAW", body=values).execute()
    format = {'requests': [
                {'updateSheetProperties': {
                    'properties': {'gridProperties': {'frozenRowCount': 1}},
                    'fields': 'gridProperties.frozenRowCount',
                }},
                {'repeatCell': {
                    'range': {'endRowIndex': 1},
                    'cell': {'userEnteredFormat': {'textFormat': {'bold': True}}},
                    'fields': 'userEnteredFormat.textFormat.bold',
                }},
            ]}
    service.spreadsheets().batchUpdate(spreadsheetId=id, body=format).execute()
    return result
```

Here we add `User` and `ID` to the categories and update the results sheet. We then format the sheet so the category row is frozen and bold.

# Final Thoughts

This was a fun project. There was definitely some difficulty with creating algorithms to work for any spreadsheet as there was a lot of variety between them. Overall I had fun creating the algorithms and I hope you had fun reading about it.

# BONUS counting the number of occurrences each shirt has

Were not done yet. Lets calculate the number of times each shirt appears in the result spreadsheet.

```python
sheet = get_sheet(RESULT)
numberOfShirts = {}
del sheet[0]
for shirt in sheet:
    if str(shirt[1]) in numberOfShirts:
        numberOfShirts[str(shirt[1])] += 1
    else:
        numberOfShirts[str(shirt[1])] = 1
sheet = [[team, num] for team, num in numberOfShirts.items()]
    
values = {"majorDimension": "ROWS", "range": "Count", "values": sheet}
result = service.spreadsheets().values().update(spreadsheetId=RESULT, range="Count", valueInputOption="RAW", body=values).execute()
```

First we get the data from the result spreadsheet. Next loop through each index counting the number of times each shirt occurs. Then we convert it to a list and add the result to the `Count` page of the final spreadsheet.

Full code: [Sheet Scraper](https://github.com/SwervyK/Sheet-Scraper){:target="\_blank"} Result: [Spreadsheet](https://docs.google.com/spreadsheets/d/1xDtfRs81EcdFHOiG-Nua_mBz2xAD4tFVVtNt6SjwPoM/edit#gid=341185536){:target="\_blank"}
