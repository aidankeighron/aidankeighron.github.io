---
layout: post
title: Group Management
date: 2023-7-29 12:00:00
categories: [java script, software]
---

# Project Overview

I often need to save large groups of tabs either for later use or for use on another device. Usually I will save every tab to a bookmarked folder, but this is tedious and unnecessary. So I created this chrome extension to make that easier.

The code shown here has been modified to make it easier to explain. If you would like to check it out, the full code is available on GitHub: [Group Extension](https://github.com/aidankeighron/Group-Extension){:target="\_blank"}

<details>
<summary><b>Table of Contents</b></summary>
<ul>
<li><a href="#project-overview">Project Overview</a></li>
<li><a href="#installation">Installation</a></li>
<li><a href="#how-to-use">How To Use</a></li>
<li><a href="#showcase">Showcase</a></li>
<li><a href="#code">Code</a></li>
</ul>
</details>

# Installation

The easiest way is to install it through the [chrome web store](https://chrome.google.com/webstore/detail/group-management/mipeplimdkiijcfjjkdgkhemfcpoaied?hl=en&authuser=0){:target="\_blank"}. Alternatively, you can download the source code for the extension. To add it to your extensions. Go to <a href='chrome://extensions/' target='_blank'>chrome://extensions/</a> (turn on developer mode in the top right corner if you don't already have it on) and select "Load unpacked". This should open a file prompt where you can select the folder for the extension. Once you have loaded the extension, you are all good to go.
* NOTE: You will not be able to transfer group data between synced devices without installing it as a chrome extension

# How To Use
When you click on the extension, a popup appears in which you can select a group to save or load. The save table displays a list of all current groups (including groups in other windows); clicking one of the buttons saves the corresponding group. Any of the buttons in the load table will load the saved group. The X next to each load button removes the group. All saved groups will be deleted if the clear button is pressed.

# Showcase
<img src="/assets/icons/group/demo_image.png" style="border:5px solid black;display: block;margin-left: auto;margin-right: auto;width: 90%;">

# Code

NOTE: JavaScript is not my best language and I am still in the process of learning

```python
function saveGroup(groupName) {
  console.log("Saving Group: " + groupName);
  chrome.tabGroups.query({ title: groupName }, function (group) {
    chrome.tabs.query({ groupId: group[0].id }, function (tabs) {
      chrome.storage.sync.remove(groupName);
      chrome.storage.sync.set({ [groupName]: tabs.map((site) => site.url)});
    });
  });

  setTimeout(refreshUI, 100);
}
```

This is how I save groups. First I get all of the current active groups filtering them by title to get a specific group name. Then I grab the first one (I am assuming that their are no duplicates) and save all of the site urls to chrome sync storage. I use sync storage because it saves across devices. Finally I wait for sync to update and refresh the ui.

```python
function loadGroup(groupName) {
  console.log("Loading Group:" + groupName);
  chrome.storage.sync.get(groupName).then((savedGroups) => {
    Object.keys(savedGroups).forEach((savedGroup) => {
      const sites = savedGroups[savedGroup];
      chrome.tabs.create(
        { url: sites.shift(), active: false},
        function (firstTab) {
          chrome.tabs.group({ tabIds: [firstTab.id] }, function (group) {
            chrome.tabGroups.update(group, { title: groupName, collapsed: true});
            sites.forEach(function (site) {
              chrome.tabs.create(
                { url: site, active: false },
                function (tab) {
                  chrome.tabs.group({tabIds: [tab.id],groupId: group});
                }
              );
          });
        }
      );
      });
    })
  })
}
```

First we get all of the saved urls for the given group name from chrome storage. Then we create the first tab and add it to a new group, creating it in the process. This step is needed because we need a group id to add tabs too, so we create the group first using the first url. Next we loop through the other urls adding each of them.

Full code: [Group Extension](https://github.com/aidankeighron/Group-Extension){:target="\_blank"}
Store link: [Chrome Web Store](https://chrome.google.com/webstore/detail/group-management/mipeplimdkiijcfjjkdgkhemfcpoaied?hl=en&authuser=0){:target="\_blank"}