# HA-NR-MYCovidStats

Home Assistant &amp; Node Red Implementation of Malaysia Covid Stats

Original sharing in FB Home Assistant Malaysia group by [Jimmy93 (FB:A Jim)](https://github.com/jimmy93/Malaysia-Daily-Covid-19-Home-Assistant) which implements Malaysia COVID-19 Statistic for use in [Home Assistant(HA)](https://www.home-assistant.io/) with HA configuration.yaml triggering command line call of python script. 

![Node-Red Flow of HA-NR-MYCovidStats](https://github.com/anasothman-myy/HA-NR-MYCovidStats/blob/main/Node-Red%20Flow%20-%20COVID19%20Stats.PNG)

This reimplements similar functions but instead uses much more easier (to me) Node-Red to callout and retrieve statistic from [repo](https://github.com/wnarifin/covid-19-malaysia) which you should read out in length how the data scarping is done and automated. *sigh -  statistic availability*  

![HA Lovelace](https://github.com/anasothman-myy/HA-NR-MYCovidStats/blob/main/lovelace-ha-nr-mycovidstats.PNG)

![Entities Statistics](https://github.com/anasothman-myy/HA-NR-MYCovidStats/blob/main/Sensor%20and%20Attributes-ha-nr-mycovidstats.PNG)

![Telegram sample](https://github.com/anasothman-myy/HA-NR-MYCovidStats/blob/main/telegram-request-ha-nr-mycovidstats.PNG)

In addition to sending back for HA for lovelace; also included in node red function for Telegram request and reporting. 


## How It Works (Extension to Node-Red)
1.  Node-Red pulls CSV from [@wnarifin github](https://github.com/wnarifin/covid-19-malaysia) and returns payload as CSV array.
2.  Array pop() on function node returns last row; which would correspond to latest data and what likely we want to see anyway.
3.  Send to HA entities defined as Sensors. Defined 4 most useful information while the rest information is nested as attributes under the sensor. 
4.  Telegram node `/getcovidstats`  for manual call request and sets flags to identify at output to only send Telegram update when requested. 

## Pre-requisites 
1.  Home Assistant with Node-Red. Ada banyak tutorial/videos on this with difficulty level as easy. [This is one example](http://https://www.juanmtech.com/get-started-with-node-red-and-home-assistant/). Test that you have enabled and can load Node-red on side bar. Make sure to also install [Node-Red companion integration](https://github.com/zachowj/hass-node-red).
2.  Telegram bot and chat ids. I followed this [tutorial](https://www.thesmarthomebook.com/2020/10/13/a-guide-to-using-telegram-with-node-red-and-home-assistant/) which is clear and easy to follow. 
> **Tip:** Follow the steps to get botid/chatid only but you do not need to setup in Home Assistant Notify/Telegram platform. Use Node-Red fully for Telegram.
4.  In Node-red the following additional nodes may be required:
-  `node-red-contrib-home-assistant-websocket` - Comes pre-installed if using default HA Node-Red Docker from Supervisor store. 
 - `node-red-contrib-telegrambot` - For Telegrambot. Setup as guide above.
5.  For Home Assistant, the following will be required:
 - [Node-Red companion integration](https://github.com/zachowj/hass-node-red); install via [HACS](https://hacs.xyz/). This allows entities to be setup from node-red instead of manual sensor entities in configuration.yaml or helpers. 
> **Tip:** Define name of entity node first before clicking 'deploy' to be able to register in HACS the exact sensor name you want instead of random numbers.
6.  For Home Assistant Lovelace to mimic my view, additional lovelace needed - mostly installed via HACS lovelace:
- [Vertical stack in card](https://github.com/ofekashery/vertical-stack-in-card) to combine cards together without border.
- [Mini graph card](https://github.com/kalkih/mini-graph-card)
    

## Installation
1. Node-Red.
- Import [Flow](https://github.com/anasothman-myy/HA-NR-MYCovidStats/blob/main/ha-nr-mycovidstats.json) into Node Red (Upper Right burger stack -> Import).
- Ensure Telegrambots and Home Assistants node Servers are configured before clicking deploy.
- Click on `inject` node to test you have data.
> **Tip:** Use `link in`/`link out` to simplify and link repetitive task i.e. send to Telegram. 
- Additional: The missing `link-in` Telegram node can be imported [here](https://github.com/anasothman-myy/HA-NR-MYCovidStats/blob/main/telegram_output.json).
2. Home Assistant
- Under`Configuration -> Integrations -> Node-Red`  following created entities should now be available:
-`sensor.covid19_malaysia_daily_stats`
-`sensor.kes_baharu`
-`sensor.kes_sembuh`
-`sensor.kes_kematian`

3. Lovelace - Import and customize to your liking.

```text
type: custom:vertical-stack-in-card
    style: |
      ha-card {
        background-color: var(--primary-background-color);
        border-radius: 15px;
        margin: 10px;
        font-size: 6 px
        box-shadow:
          {% if is_state('sun.sun', 'above_horizon') %}
            -4px -4px 8px rgba(255, 255, 255, .5), 5px 5px 8px rgba(0, 0, 0, .03);
          {% elif is_state('sun.sun', 'below_horizon') %}
            -5px -5px 8px rgba(50, 50, 50, .2), 5px 5px 8px rgba(0, 0, 0, .08);
          {% endif %}
       }
        .card-header {
        font-size: 6 px
      }
    cards:
      - type: entity
        entity: sensor.covid19_malaysia_daily_stats
        name: 'Statistic COVID-19 Malaysia '
      - type: horizontal-stack
        cards:
          - type: custom:mini-graph-card
            entities:
              - color: red
                entity: sensor.kes_baharu
                unit: kes
            name: Baru
            hours_to_show: 168
            group_by: date
            show:
              state: true
              fill: false
          - type: custom:mini-graph-card
            entities:
              - entity: sensor.kes_sembuh
                unit: kes
            name: Sembuh
            hours_to_show: 168
            group_by: date
            show:
              state: true
              fill: false
              units: false
          - type: custom:mini-graph-card
            entities:
              - color: black
                entity: sensor.kes_kematian
                unit: kes
            name: Kematian
            hours_to_show: 168
            group_by: date
            show:
              state: true
              fill: false

```
        
