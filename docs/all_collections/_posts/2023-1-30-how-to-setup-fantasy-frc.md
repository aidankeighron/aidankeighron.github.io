---
layout: post
title: How to Setup Fantasy FRC
date: 2023-1-30 12:00:00
categories: [website, FRC]
---

# Project Overview

Fantasy FRC is fantasy football with robotics. It is a website that manages the full process of running a draft for FRC and it's pretty customizable. If you want to see how it looks check out the [showcase](#showcase).

If you would like to check it out, the full code is available on GitHub: [Fantasy FRC](https://github.com/aidankeighron/Fantasy-FRC){:target="\_blank"}.

## Updates

This project is going to receive future updates, if you would like to receive an email when these updates happen along with instructions on how to update your version, add your email below.

\* Email might get sent to spam or promotions folder

<!-- Begin Mailchimp Signup Form -->
<link href="//cdn-images.mailchimp.com/embedcode/classic-071822.css" rel="stylesheet" type="text/css">
<style type="text/css">
	#mc_embed_signup{
    background:black; clear:left;
  }
  .mce_inline_error {
    color: black !important;
    width: 96.5%;
    margin-left: 1% !important;
  }
</style>
<div id="mc_embed_signup" style="border-radius: 25px">
    <form action="https://fantasyrobotics.us21.list-manage.com/subscribe/post?u=e32ab2f618eca5c413307660c&amp;id=a56b26e9b4&amp;f_id=003de7e1f0" method="post" id="mc-embedded-subscribe-form" name="mc-embedded-subscribe-form" class="validate" target="_blank" novalidate>
    <div style="overflow-x: visible !important">
        <h2>Receive Updates</h2>
        <div class="mc-field-group">
          <label for="mce-EMAIL" style="margin-left: 1%">Email Address  <span class="asterisk">*</span></label>
          <input style="width: 97%; background-color: #A9ABB3; margin-left: 1%" type="email" value="" name="EMAIL" class="required email" id="mce-EMAIL" required>
          <span id="mce-EMAIL-HELPERTEXT" style="background-color: black; color: white;" class="helper_text"></span>
        </div>
        <div hidden="true" class="error-msg"><input type="hidden" name="tags" value="261553"></div>
        <div id="mce-responses" class="clear foot">
          <div class="response" id="mce-error-response" style="display:none"></div>
          <div class="response" id="mce-success-response" style="display:none"></div>
        </div>    <!-- real people should not fill this in and expect good things - do not remove this or risk form bot signups-->
        <div style="position: absolute; left: -5000px;" aria-hidden="true"><input type="text" name="b_e32ab2f618eca5c413307660c_a56b26e9b4" tabindex="-1" value=""></div>
        <div class="optionalParent">
            <div class="clear foot">
                <input style="background-color: #A9ABB3;margin-left: 1%" type="submit" value="Subscribe" name="subscribe" id="mc-embedded-subscribe" class="button">
                <p class="brandingLogo"><a style="border-bottom: 0px" href="http://eepurl.com/ijnwl1" title="Mailchimp - email marketing made easy and fun"><img src="https://eep.io/mc-cdn-images/template_images/branding_logo_text_light_dtp.svg"></a></p>
            </div>
        </div>
    </div>
    </form>
</div>
<script type='text/javascript' src='//s3.amazonaws.com/downloads.mailchimp.com/js/mc-validate.js'></script><script type='text/javascript'>(function($) {window.fnames = new Array(); window.ftypes = new Array();fnames[0]='EMAIL';ftypes[0]='email';fnames[1]='FNAME';ftypes[1]='text';fnames[2]='LNAME';ftypes[2]='text';fnames[3]='ADDRESS';ftypes[3]='address';fnames[4]='PHONE';ftypes[4]='phone';fnames[5]='BIRTHDAY';ftypes[5]='birthday';}(jQuery));var $mcj = jQuery.noConflict(true);</script>
<!--End mc_embed_signup-->

## Modifying

The source code is protected under the MIT license so you have full rights to modify it as long as you keep the licenses and copyright intact. NOTE I am not an expert in JS and while I have done my best to weed out any bugs there might still be some out there. If you find any please open an issue on GitHub or create a pull request with a fix. 

<details>
<summary><b>Table of Contents</b></summary>
<ul>
<li><a href="#project-overview">Project Overview</a></li>
<li><a href="#updates">Updates</a></li>
<li><a href="#modifying">Modifying</a></li>
<li><a href="#installation">Installation</a></li>
<li><a href="#download-source-code-and-required-software">Download Source Code and Required Software</a></li>
<li><a href="#setup-database-and-apis">Setup Database and APIs</a></li>
<li><a href="#setup-website">Setup Website</a></li>
<li><a href="#yearly-setup">Yearly Setup</a></li>
<li><a href="#run">Run</a></li>
<li><a href="#showcase">Showcase</a></li>
</ul>
</details>

# Installation

If you run into problems during the installation process you can ask for help at [fantasyroboticshelp@gmail.com](mailto:fantasyroboticshelp@gmail.com).

## Download Source Code and Required Software

Download the source code from [the GitHub repository](https://github.com/aidankeighron/Fantasy-FRC){:target="\_blank"} and put it on whatever computer you want to run the website. I used a dedicated machine running Ubuntu Server but the website is not very intense so any machine should work.

Install Node.js ([Windows](https://nodejs.org/en/download/){:target="\_blank"} \| [Ubuntu](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-22-04/){:target="\_blank"}). If on Linux navigate to /server uninstall bcrypt and reinstall it using npm. This is needed because the Windows and Linux versions are different.

Install MySQL ([Windows](https://www.dataquest.io/blog/install-mysql-windows/){:target="\_blank"} \| [Ubuntu](https://www.dataquest.io/blog/install-mysql-windows/){:target="\_blank"}) and go through initial setup.

## Setup Database and APIs

Login to your MySQL server and create a database, which I called `fantasy`. And create two tables for users and teams.

Create users table:
```SQL
CREATE TABLE users (
  name VARCHAR(255),
  passw CHAR,
  id VARCHAR(255),
  position INT,
  score DOUBLE,
  teams TEXT
);
```

Create teams table:
```SQL
CREATE TABLE teams (
name VARCHAR(255),
number INT,
average DOUBLE,
opr DOUBLE,
score DOUBLE,
location VARCHAR(255),
owner VARCHAR(255) 
);
```

Get a Blue Alliance API key from their [website](https://www.thebluealliance.com/apidocs/v3){:target="\_blank"}.

Create server_info.ini in /server and add the required fields to it, it should look like this:

```ini
[SQL]
SQL_Passw = "password" # Password for MySQL server
SQL_IP = "192.168.1.1" # Hostname for MySQL server
SQL_User = "username" # Username for MySQL serer
SQL_Database = "fantasy" # Database name for MySQL server
[TBA]
BLUE_ALLIANCE = supercoolapikey # API key for Blue Alliance website (do not put it in quotes)
[SERVER]
SECRET = "super secret" # secret for sessions (I made mine a random string)
```


## Setup Website

Navigate to /server/server.js, and locate `app.get('/admin', (req, res) => {`. This is the code for serving the admin page. As there are no admin users we will need to disable security for the admin page to add them.

Change:
```js
app.get('/admin', (req, res) => {
  try {
    if (req.isAuthenticated()) {
      user = Object.values(JSON.parse(JSON.stringify(req.user[0])))[1];
      if (user === adminName) {
        res.sendFile('www/admin.html', { root: __dirname });
      }
      else {
        res.sendStatus(403);
      }
    }
    else {
      res.redirect('/');
    }
  }
  catch (error) {
    console.log("ERROR:");
    console.log(error);
    res.sendStatus(500);
  }
});
```

To this:
```js
app.get('/admin', (req, res) => {
  try {
    // if (req.isAuthenticated()) {
    //   user = Object.values(JSON.parse(JSON.stringify(req.user[0])))[1];
    //   if (user === adminName) {
        res.sendFile('www/admin.html', { root: __dirname });
    //   }
    //   else {
    //     res.sendStatus(403);
    //   }
    // }
    // else {
    //   res.redirect('/');
    // }
  }
  catch (error) {
    console.log("ERROR:");
    console.log(error);
    res.sendStatus(500);
  }
});
```

\* If you want to connect to the website externally you will need to [port forward](https://portforward.com/){:target="\_blank"} port 80 to your web server.

Run the website using `node server.js` (server.js in /server) and navigate to `<hostname>/admin` (the admin page is only accessible by direct link) and type in a username and password (see [showcase](#showcase) for what the admin page looks like). Once your admin has been added uncomment the lines to re-enable security. Then go to your MySQL server and get the ID and name of the account. In server.js locate `const adminId =` and set `adminId` and `adminName` to the user's ID and name respectively. Once an admin user is set up your admin go to the admin page and add the rest of your users.

## Yearly Setup

The setup for the 2023 season is already done but if you need to redo it for any reason here is the process.
- Update the year in both rolling_team_info.py and get_team_info.py
- Run /server/get_team_info.py and move the generated team_info.json to /server

## Run

To run the server you can use `node server.js` (server.js is in /server) to run the server and view the output. If you do not need to see the output and you are running the website on a dedicated machine you can use [forever](https://www.npmjs.com/package/forever){:target="\_blank"}.

## How does the draft work

The draft starts when the `Start Draft` button is pressed on the admin page. As soon as it is pressed all users will get access to the drafting page and a 5-minute countdown is started. Once the countdown ends the draft will start and users will get a chance to pick teams. Each user gets 20 seconds to pick a team and each team can only be picked once. Users can only have one team from each location (location is country or state if it is a USA team). If a user picks a team with a duplicate location the team will get removed from their queue and the 20-second timer will get reset (NOTE a new team will not be chosen until the user adds another team). For Example: 
- A user picks a team with a location they already have drafted
- The server rejects their choice and the timer is reset and the rejected team is removed
- The user removes the team at the top of their queue because they are not ready to pick them
- The user presses the `Add` button on any team and the team at the top of their queue is chosen

## How do teams get a score

Run `/server/rolling_team_info.py` and it will update all team's and user's scores. It will need to be ran regurarly to keep things up to date.

# Showcase

## Home

<img src="/assets/icons/fantasy/home.png" style="border:5px solid black;display: block;margin-left: auto;margin-right: auto;width: 80%;">

## Teams

<img src="/assets/icons/fantasy/teams.png" style="border:5px solid black;display: block;margin-left: auto;margin-right: auto;width: 80%;">

## Rankings

<img src="/assets/icons/fantasy/rankings.png" style="border:5px solid black;display: block;margin-left: auto;margin-right: auto;width: 80%;">

## Login

<img src="/assets/icons/fantasy/login.png" style="border:5px solid black;display: block;margin-left: auto;margin-right: auto;width: 80%;">

## Settings

<img src="/assets/icons/fantasy/settings.png" style="border:5px solid black;display: block;margin-left: auto;margin-right: auto;width: 80%;">

## How To

<img src="/assets/icons/fantasy/how_to.png" style="border:5px solid black;display: block;margin-left: auto;margin-right: auto;width: 80%;">

## Admin

<img src="/assets/icons/fantasy/admin.png" style="border:5px solid black;display: block;margin-left: auto;margin-right: auto;width: 80%;">

## Drafting

<img src="/assets/icons/fantasy/drafting.png" style="border:5px solid black;display: block;margin-left: auto;margin-right: auto;width: 80%;">

## Search

<img src="/assets/icons/fantasy/drafting_search.png" style="border:5px solid black;display: block;margin-left: auto;margin-right: auto;width: 80%;">

On the drafting page and teams page there is a search option that lets you search using team numbers or names.
