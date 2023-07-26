# Docker-Planefence

## What is it?

Planefence is an ADS-B alert system and proximity monitor, meant to compliment  `readsb`, `dump1090`, or `dump1090-fa`.

Planefence creates a log of aircraft heard by your feederstation that are within a defined area or "fence". Fencing exists for both ground distance from a declared point as well as altitude. This log is compiled and formatted in both HTML and CSV, and made available on localhost.

Planefence also features integration with Twitter and Mastodon, among other active development.

Planefence is deployed as a Docker container and is pre-built for the following architectures:
- linux/ARMv6 (armel): older Raspberry Pi's
- linux/ARMv7 (armhf): Raspberry Pi 3B+ / 4B with the standard 32 bits Raspberry OS (tested on Busted, may work but untested on Stretch or Jessie)
- linux/ARM64: Raspberry Pi 4B with Ubuntu 64 bits OS
- linux/AMD64: 64-bits PC architecture (Intel or AMD) running Debian Linux (incl. Ubuntu)
- linux/i386: 32-bits PC architecture (Intel or AMD) running Debian Linux (incl. Ubuntu)

The Docker container can be accessed on [Dockerhub (kx1t/planefence)](https://hub.docker.com/repository/docker/kx1t/planefence) and can be pulled directy using this Docker command: `docker pull kx1t/planefence`.

## Who is it for?
Planefence is NOT a standalone image. You'll need to have a working ADS-B feed of some form to feed Planefence. Experience in these areas will help:

- The `dump1090` family of ADS-B software (for example, `readsb`, `tar1090`, `dump1090`, or `dump1090-fa`), how to deploy it, and the hardware needed. Ideally, you have your ADS-B station already up and running.
- Deploying Docker images to your machine. If you don't -- it's actually quite simple! [Mikenye's excellent Gitbook](https://mikenye.gitbook.io/ads-b/) contains a step-by-step guide.
- Using Docker via `docker-compose`. This README has been written assuming `docker-compose`. If you don't have it, feel free to `apt-get install` it. It should be easy to convert the `docker-compose.yml` instructions to a command-line `docker run` string, but support is not offered for this.
- Further support is provided at the #planefence channel at the [SDR Enthusiasts Discord Server](https://discord.gg/VDT25xNZzV). If you need immediate help, please add "@k1xt" to your message. Note that, if you're reading this, you are using a FORK of k1xt's work. If you do engage with this person, be sure to make them aware that you are using a FORKED version.

## Install PlaneFence - Prerequisites

This guide assumes that `/home/pi` is your home directory. If it is not, please change all mentions of `/home/pi` to the applicable home directory path. If you execute `cd ~` and the directory listed is different, you need to use that directory.

You must already be an instance of `tar1090`, `dump1090[-fa]`, or `readsb` connected to a SDR somewhere in reach of your Planefence machine, either on the machine or remotely.
- It is imperative to enable SBS data on port 30003 on whatever instance feeds ADS-B data to PlaneFence. PlaneFence will use this to get its data. See the Troubleshooting section for help to get this done

### Getting ready (existing stack of Docker containers)

1. You can add the information from this project to your existing `docker-compose.yml`.
2. Get the template Docker-compose.yml file from here:
```
curl -s https://raw.githubusercontent.com/cullyRSG/docker-planefence/main/docker-compose.yml > docker-compose.yml
```
3. If you're looking to copy-and-paste, [this link](https://github.com/cullyRSG/docker-planefence/blob/main/docker-compose.yml) will pull up this repository's docker-compose.yml file.
4. Be sure to remove any containers that you already have running. If you already have a container running that is listed here, ensure the settings for the running container is compatible with PlaneFence.
### Getting ready (Creating a new stack of Docker containers)

1. Create a project directory: `sudo mkdir -p /opt/planefence && sudo chmod a+rwx /opt/planefence && cd /opt/planefence`
2. Add a new `docker-compose.yml` to it:
```
curl -s https://raw.githubusercontent.com/kx1t/docker-planefence/main/docker-compose.yml > docker-compose.yml
```

### Planefence Configuration

#### Initial Docker rollout
In the `docker-compose.yml` file, you should configure the following:
- IMPORTANT: Update `TZ=America/New_York` to your time zone. Note that this variable is case sensitive. A list of time zone codes can be found [here](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones), take data from the "TZ identifier" column.
- There are 2 defined volumes. With the exception of changing /home/pi/.planefence to /home/username/.planefence as necessary, do NOT to change these volumes. If you have to, you can map the HTML directory to some other location. ONLY change what is to the LEFT of the colons. Changing data on the right of the colons will prohibit function.
- 
- Exit the editor and start the container with `docker-compose up -d`. The first time you do this, it can take a minute or so to build the images. 
- Monitor the container (`docker logs -f planefence`). At first start-up, it should be complaining about not being configure. That is expected behavior.
- Once you see the warnings about `planefence.config` not being available, press CTRL-C to get the command prompt.
- Sidenote: The image in this docker-compose.yml file, by default, points at the release image. For the DEV version, change the file to this: `image: kx1t/planefence:dev`

#### Building planefence.config
After you start the container for the first time, it will create a few directories with setup files. At this point in the tutorial, nothing should work. You MUST edit these setup files before things will work. Bar none, ~/.planefence/planefence.config is the most vital. Ensure this file is built well.
- MANDATORY: First -- copy the template config file in place: `cp ~/.planefence/planefence.config-RENAME-and-EDIT-me ~/.planefence/planefence.config`
  - ALTERNATIVE - If you have used PlaneFence before and created a `.env` file, you can use this file as a basis for your `planefence.config` file. You can copy it with `sudo cp /opt/planefence/.env ~/.planefence/planefence.config`. However, there are many new features and setting described in the `planefence.config-RENAME-and-EDIT-me file`. You should take notice and copy these in! There are some items (like the setup for the different feeders) that are not needed by `planefence.config`.
  - Note -- placing the full parameter set in `.env` is OBSOLETE and support for this will be withdrawn in the next version of PlaneFence
- MANDATORY: `sudo nano ~/.planefence/planefence.config` Go through all parameters - their function is explained in this file. Edit to your liking and save/exit.

#### Preparing other PlaneFence settings
These settings are all optional, but will help provide the best experience for you.
- Customizing aircraft for PlaneFence to ignore: `sudo nano ~/.planefence/planefence-ignore.txt`. If there are specific planes that fly too often over your home, add them here. Use 1 line per entry, and the entry can be a ICAO, flight number, etc. You can even use regular expressions if you want. Be careful -- we use this file as an input to a "grep" filter. If you put something that is broad (`.*` for example), then ALL PLANES will be filtered out!
- Help us map airline codes: `sudo nano ~/.planefence/airlinecodes.txt`. This file maps the first 3 characters of the flight number to the names of the airlines. We scraped this list from a Wikipedia page, and it is by no means complete. Feel free to add more to them -- please add an issue at https://github.com/kx1t/planefence/issues so we can add your changes to the default file.
- Backup Twitter TWURLRC: If you configured Twitter support before, `sudo nano ~/.planefence/.twurlrc`. You can add your back-up TWURLRC file here, if you want.
- Use Twitter and configure Tweets: Configure tweets to be sent. For details, see these instructions: https://github.com/kx1t/docker-planefence/blob/main/README-planetweet.md
- Plane Alert database: `sudo nano ~/.planefence/plane-alert-db.txt`. This is the list of tracking aircraft of Plane-Alert. It is prefilled with the planes of a number of "interesting" political players. Feel free to add your own, delete what you don't want to see, etc. Just follow the same format.
- If you have multiple containers running on different web port, and you would like to consolidate them all under a single host name, then you should consider installing a "reverse web proxy". This can be done quickly and easily - see instructions [here](https://github.com/kx1t/docker-planefence/README-nginx-rev-proxy.md).
- If you have a soundcard and microphone, adding NoiseCapt is as easy as hooking up the hardware and running another container. You can add this to your existing `docker-compose.yml` file, or run it on a different machine on the same subnet. Instructions are [here](https://github.com/kx1t/docker-noisecapt/).
- For Plane-Alert: You can add custom fields, that are displayed on the Plane-Alert list. See [this discussion](https://github.com/kx1t/docker-planefence/issues/38) on how to do that.
- The website will apply background pictures if you provide them. Save your .jpg pictures as `~/.planefence/pf_background.jpg` for Planefence and `~/.planefence/pa_background.jpg` for Plane-Alert. (You may have to restart the container or do `touch ~/.planefence/planefence.config` in order for these backgrounds to become effective.)
- Add images of tar1090 to your Tweets in Planefence and Plane-Alert. In order to enable this, simply add the `pf-screenshot` section to your `Docker-compose.yml` file as per the example in this repo's [`Docker-compose.yml`](https://github.com/kx1t/docker-planefence/blob/main/docker-compose.yml) file. Note - to simplify configuration, Planefence assumes that the hostname of the screenshotting image is called `pf-screenshot` and that it's reachable under that name from the Planefence container stack.
- OPTIONAL: Show [OpenAIP](http://map.openaip.net) overlay on Planefence web page heatmap. Enable this by setting the option `PF_OPENAIP_LAYER=ON` in `~/.planefence/planefence.config`


#### Applying your setup
- If you made a bunch of changes for the first time, you should restart the container. Most updates to `~/.planefence/planefence.config` will be picked up automatically, but Docker containers are meant to be disposable. Simply `cd` to your docker-compose.yml file and run `docker-compose up -d` to restart the containers.
- You can restart the Planefence container by doing: `pushd /opt/planefence && docker stop planefence && docker-compose up -d && popd`

## What does it look like when it's running?
- Planefence deployment example: https://planefence.com/planefence
- Plane-Alert deployment example: https://planefence.com/plane-alert
- Planefence tweets: https://twitter.com/planeboston

## API access to your data
### Introduction
Planefence and Plane-Alert keep a limited amount of data available. By default, PlaneFence keeps 2 weeks of data around, while Plane-Alert isn't time limited. This data is accessible using a REST interface that makes use of HTTP GET. You can access this API from the directory where your Planefence or Plane-Alert web pages are deployed. For example:
- If Planefence is available at https://planefence.com/planefence, then you can reach the Planefence API at https://planefence.com/planefence/pf-query.php
- If Plane-Alert is available at https://planefence.com/plane-alert, then you can reach the Plane-Alert API at https://planefence.com/plane-alert/pa-query.php
### API parameters and usage examples
The Planefence and Plane-Alert APIs accept awk-style Regular Expressions as arguments. For example, a tail number starting with N, followed by 1 digit, followed by 1 or more digits or letters would be represented by this RegEx: `n[0-9][0-9A-Z]*` .  Querie arguments are case-insensitive: looking for `n` or for `N` yield the same results.
Each query must contain at least one of the parameters listed below. Optionally, the `type` parameter indicates the output type. Accepted values are `json` or `csv`; if omitted, `json` is the default value. (These argument values must be provided in lowercase.)
Note that the `call` parameter (see below) will start with `@` followed by the call (tail number or flight number as reported via ADS-B/MLAT/UAT) if the entry was tweeted. So make sure to start your `call` query with `^@?` to include both tweeted an non-tweeted calls.
#### Planefence Query parameters
| Parameter | Description | Example |
|---|---|---|
| `hex` | Hex ID to return | https://planeboston.com/planefence/pf_query.php?hex=^A[AB][A-F0-9]*&type=csv returns a CSV with any Planefence records of which the Hex IDs that start with A, followed by A or B, followed by 0 or more hexadecimal digits |
| `tail` | Call sign (flight number or tail) to return | https://planeboston.com/planefence/pf_query.php?call=^@?AAL[0-9]*&type=json returns any flights of which the call starts with "AAL" or "@AAL" followed by only numbers. (Note - the call value will start with `@` if the entry was tweeted, in which case the `tweet_url` field contains a link to the tweet.) |
| `start` | Start time, format `yyyy/MM/dd hh:mm:ss` | https://planeboston.com/planefence/pf_query.php?start=2021/12/19.*&type=csv returns all entries that started on Dec 19, 2021. |
| `end` | End time, format `yyyy/MM/dd hh:mm:ss` | https://planeboston.com/planefence/pf_query.php?end=2021/12/19.*&type=csv returns all entries that ended on Dec 19, 2021. |

#### Plane-Alert Query parameters
| Parameter | Description | Example |
|---|---|---|
| `hex` | Hex ID to return | https://planeboston.com/plane-alert/pa_query.php?hex=^A[EF][A-F0-9]*&type=csv returns a CSV with any Planefence records of which the Hex IDs that start with A, followed by E or F, followed by 0 or more hexadecimal digits. (Note - this query returns most US military planes!) |
| `tail` | Tail number of the aircraft | https://planeboston.com/plane-alert/pa_query.php?tail=N14[0-9]NE&type=csv returns any records of which the tail starts with "N14", followed by 1 digit, followed by "NE". |
| `name` | Aircraft owner's name | https://planeboston.com/plane-alert/pa_query.php?name=%20Life\|%20MedFlight&type=csv returns any records that have " Life" or " MedFlight" in the owner's name. |
| `equipment` | Equipment make and model | https://planeboston.com/plane-alert/pa_query.php?equipment=EuroCopter returns any records of which the equipment contains the word "EuroCopter" |
| `timestamp` | Time first seen, format `yyyy/MM/dd hh:mm:ss` | https://planeboston.com/plane-alert/pa_query.php?timestamp=2022/01/03 returns any records from Jan 3, 2022. |
| `call` | Callsign as reported by aircraft | https://planeboston.com/plane-alert/pa_query.php?call=SAM returns any records of which the callsign contains "SAM". |
| `lat` | Latitude first observation, in decimal degrees | https://planeboston.com/plane-alert/pa_query.php?lat=^43 returns any records of which the latitude starts with "43" (i.e., 43 deg N) |
| `lon` | Longitude first observation, in decimal degrees | https://planeboston.com/plane-alert/pa_query.php?lon=^-68 returns any records of which the longitude starts with "-68" (i.e., 68 deg W) |


## Troubleshooting
- Be patient. Some of the files won't get initialized until the first "event" happens: a plane is in PlaneFence range or is detected by Plane-Alert. This includes the planes table and the heatmap.
- You may have to run `sudo chmod -R 777 /opt/planefence/Volumes`. I'm sure there's a better way to do this.
- If your system doesn't behave as expected: check, check, double-check. Did you configure the correct container in `docker-compose.yml`? Did you edit the `planefence.config` file?
- Check the logs: `docker logs -f planefence`. Some "complaining" about lost connections or files not found is normal, and will correct itself after a few minutes of operation. The logs will be quite explicit if it wants you to take action
- Check the website: http://myip:8088 should update every 80 seconds (starting about 80 seconds after the initial startup). The top of the website shows a last-updated time and the number of messages received from the feeder station.
- Plane-alert will appear at http://myip:8088/plane-alert
- If you are getting a 403 error at http://myip:8088/ but at http://myip:8088/plane-alert serves a page, it likely means you are set up correctly and need data to meet the criteria set in planefence.config. Consider setting `PF_MAXALT=40000 & PF_MAXDIST=100` temporarily to jumpstart the data flow and generate an `index.html`
- Twitter setup is complex. [Here](https://github.com/kx1t/docker-planefence#setting-up-tweeting)'s a description on what to do.
- Error "We cannot reach {host} on port 30003". This could be caused by a few things:
    - Did you set the correct hostname or IP address in `PF_SOCK30003HOST` in `~/.planefence/planefence.config`? This can be either an IP address, or an external hostname, or the name of another container in the same stack (in which case you use your machine's IP address).
    - Did you enable SBS (BaseStation -- *not* Beast!) output? Here are some hints on how to enable this:
      - For non-dockerized `dump1090[-fa]`/`readsb`/`tar1090`: add command line option `--net-sbs-port 30003`
      - For dockerized `readsb-protobuf`:
      add to the `environment:` section of your `docker-compose.yml` file:
      ```
            - READSB_NET_SBS_OUTPUT_PORT=30003
            - READSB_EXTRA_ARGS=--net-beast-reduce-interval 2 --net-sbs-reduce
      ```
      if you are using a different container stack, then you should also add `- 30003:30003` to the `ports:` section


## Getting help
- If you need further support, please join the #planefence channel at the [SDR Enthusiasts Discord Server](https://discord.gg/VDT25xNZzV) and look for "@kx1t" to your message. Alternatively, email me at kx1t@amsat.org.

That's all!

![](https://media.giphy.com/media/3oKHWikxKFJhjArSXm/giphy.gif)
