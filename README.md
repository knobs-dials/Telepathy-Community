## A fork of Telepathy: An OSINT toolkit for investigating Telegram chats

This is a fork of [Telepathy-Community](https://github.com/prose-intelligence-ltd/Telepathy-Community), a "swiss army knife of Telegram tools".

Prose's repo this came from is at [prose-intelligence-ltd/Telepathy-Community](https://github.com/prose-intelligence-ltd/Telepathy-Community),
its further development into a web based index is at [prose.ltd](https://prose.ltd/).


This fork currently focuses on trying to make it easier to run your own from the command line, and perhaps do some minor development.


## Installation

### Development tools

* (optional, recommended) install pipx to make poetry install easier:

    `python3 -m pip install --user pipx && python3 -m pipx ensurepath`

* We use `poetry` to make the 'create virtual environment that will run this' step eaiser.
  You can get a recent version, and avoid having to do anything as admin, by installing it through pipx.
  (if you skip pipx: distro package management often have poetry, but frequently only old versions)

    `pipx install poetry`

* We also use `pyenv` to get a specific version of python to install in our user account.
  - This is technically optional, but only if you somehow already have a python 3.10, which this project currently specifically wants.
  - pyenv [has its own instructions](https://github.com/pyenv/pyenv?tab=readme-ov-file#a-getting-pyenv)
    (this installs into your specific user account, again no admin required)


### Telepathy itself
* get contents of this repo:
  
    `git clone https://github.com/knobs-dials/Telepathy-Community.git`

* we currently recommend installing python 3.10 via pyenv:  for poetry to pick up in the next step:
  
    `pyenv install 3.10` 

* now `cd` to the cloned repo and tell poetry to create a virtualenv for it with the dependencies ant telepathy itself installed:
  
     `poetry install`

The last step installed telepathy isolated within that virtualenv, 
which means you have to return to that virualenv to run it.

Perhaps the easiest is to `cd` to the project dir before doing `poetry run telepathy`
(telepathy is also a command line tool that was registerd within that virualenv)

<!--
TODO: figure out whether cryptg should be on an optional flag.
The package hand decryption by Python over to C, making media downloads in particular quicker and more efficient.
-->


## Setup

On first use, Telepathy will ask for your Telegram API details (obtained from my.telegram.org). 

Once those are set up, it will prompt you to enter your phone number again and then send an authorization code to your Telegram account. 
If you have two-factor authentication enabled, you'll be asked to input your Telegram password.


## Usage:

```
poetry run telepathy [OPTIONS]
```

### **`--target CHAT` / `-t CHAT`**

this option will identify the target of the scan. The specified chat must be public or have a private link.
To get the chat name, look for the 't.me/chatname' link, and subtract the 't.me/'.

For example:

```
$ telepathy -t durov
```

### **`--comprehensive` / `-c`**

Without this parameter we do a basic scan, 
meaning we find the 
title, 
description, 
number of participants, 
username, 
URL, 
chat type, 
chat ID, 
access hash, 
first post date and 
any applicable restrictions to the chat.
For group chats, Telepathy will also generate a memberlist (up to 5,000 members).

If you specify this option, you get a comprehensive scan, which offers the same information as the basic scan, 
and will also archive a chat's message history,
gather the number of reactions,
archive how many times a message has been forwarded, 
the number of replies to each message, 
and more.

Reaction lists are included in the archive file, including basic calculations of engagement rate. Only the most-common reactions are listed, with the total including all possible reactions. Currently, Telepathy calculates engagement rates based on forwards, comments and reactions seperately, with a calculation based on post views and one based on chat participant count. In future, Telepathy may include deeper analytics which can be cross-compared between chats based on a combination of these metrics, fixing for when comments, reactions or forwards are allowed or disallowed in a given chat.

For example:

```
$ telepathy -t durov -c
```

#### **`--forwards` / `-f`**

This flag will create an edgelist based on messages forwarded into a chat. 
It can be used alongside either a default or comprehensive scan. 
Since 2.3.0, Telepathy now formats these edgelists to maximize compatability with Gephi.

For example:

```
$ telepathy -t durov -f

$ telepathy -t durov -c -f
```


#### **`--media` / `-m`**

Use this flag to include media archiving alongside a comprehensive scan. 
This makes the process take significantly longer and should also be used with caution: 
you'll download all media content from the target chat, and it's up to you to not store illegal files on your system.

To archive media, you must run a comprehensive scan **and** add this flag:

```
$ telepathy -t durov -c -m
```

Once files have downloaded, you can run exiftool on the associated media directory to gather deeper insights on the files, their metadata, and in some cases attribute who might be behind an anonymous channel. Further details are in the "bonus investigations tips" section of this README.


#### **`--user` / `-u`**

Looks up a specified user. This will only work if your account has "encountered" the user before (for example, after archiving a group), you can specify User ID or @nickname. If looking up by username, it's not always necessary for your account to have already seen the user.

```
$ telepathy -t 0123456789 -u

$ telepathy -t @test_user -u
```


#### **`--location` / `-l`**

Finds users near to specified coordinates. Input should be longitude followed by latitude, seperated by a comma.
This feature only works if your Telegram account has a profile image which is set to be publicly viewable.
As of 2.3.4, this feature now includes channel lookups. 

While searches for multiple locations at once may work in some cases, Telegram appears to have a limit on how quickly an account can cycle through locations.
At the time of writing, this appears to be at least ten minutes. 
Further location scanning support while using multiple accounts is being explored for a future release.

```
$ telepathy -t 51.5032973,-0.1217424 -l
```


#### `--alt NUMBER` / `-a NUMBER`**

Flag for running Telepathy from an alternative number or API details.
You can use the same API key and Hash but authenticate with a different phone number.
This allows for running multiple scans at the same time. Telepathy will default to the first details you offer, and up to four others can be added.
Please see the notes at the top of this README for information regarding limitations with user IDs using this method.

```
$ telepathy -t Durov -c -a 1
```


#### **`--export` / `-e`**

Exports all chats your account is part of to a CSV file. 
In a future release, this may assist with provisioning new accounts to automatically following the listed groups.

```
$ telepathy -e
```
  

#### **`--reply` / `-r`**

Enable channel reply retrieval, which will archive replies and list users who replied to messages in the target channel. 

```
$ telepathy -t [CHANNEL] -c -r 
```


#### **`--translate` / `-tr`**

Enable auotmatic translation (currently only into English) during message retrieval.

```
$ telepathy -t [CHANNEL] -c -tr
```


## Bonus investigations tips:

 - Navigating to a media archive directory and running Exiftool may give you a whole host of useful information for further investigation. Telegram doesn't currently scrub metadata from PDF, DOCX, XLSX, MP4, MOV and some other filetypes, which offer creation and edit time metadata, often timezones, sometimes authors, and general technical information about the perosn or people who created a media file.  
 ```
$ cd ./telepathy/telepathy_files/CHATNAME/media
$ exiftool * > metadata.txt
```
 - Group and inferred channel memberlists offer a point of further investigation for usernames found. By using [Maigret](https://github.com/soxoj/maigret), you can look up where else a username has been used online. While this is not accurate in all cases, it's been proven to be helpful for identifying where a person has reused handles across platforms. In this case, remember to verify your findings to avoid false positives.


## Notes on how Telegram works

Telegram chats are organised into three key types: Channels, Megagroups/Supergroups and Gigagroups.
Each option works slightly differently depending on the chat type.
Channels can have seemingly unlimited subscribers and are where an admin will broadcast messages to an audience,
Megagroups can have up to 200,000 members, each of whom can participate (if not restricted), and 
Gigagroups sit somewhere between the two.

<!--
## Upcoming changes
In some environments (particularly Windows), Telepathy struggles to effectively manage files and can sometimes produce errors. Fixes for these errors will come in due course.

Upcoming features include:

  - [ ] Adding a time specification flag to set archiving for specific period.
  - [ ] A new method to once again gather complete memberlists (currently restricted by the API).
  - [ ] Ensuring inferred channel memberlists don't contain duplicate entries.
  - [ ] Exploration of whether channel events can be included, such as name changes.
-->

## Usage and credits

You may use Telepathy however you like, but your usecase is your responsibility.
Be safe and respectful.

Big thanks to the original creators and contributors named in the source repo:
Jordan Wildon (@jordanwildon), 
[Giacomo Giallombardo](https://github.com/aaarghhh),
[jkctech](https://github.com/jkctech/Telegram-Trilateration),
Alex Newhouse (@AlexBNewhouse), and
[Francesco Poldi](https://github.com/pielco11).

As it notes, credit for the use of this tool in published research is desired, but not required.

If your usecase is for commercial purposes, also consider Prose's enterprise options at [prose.ltd](https://prose.ltd)
