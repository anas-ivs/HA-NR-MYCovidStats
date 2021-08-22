# HA-NR-MYCovidStats

Home Assistant &amp; Node Red Implementation of Malaysia Covid Stats
![visitors](https://visitor-badge.glitch.me/badge?page_id=anas-ivs.ha-nr-mr-covidstats.visitor-badge)

| [How it Works](#How) | [Pre-requisites](#Pre) | [Installation](#Install) | [Credits](#Credits) |

![](https://raw.githubusercontent.com/anas-ivs/HA-NR-MYCovidStats/main/images/images\banner.PNG)

Original sharing in FB Home Assistant Malaysia group by [Jimmy93 (FB:A Jim)](https://github.com/jimmy93/Malaysia-Daily-Covid-19-Home-Assistant) which implements Malaysia COVID-19 Statistic for use in [Home Assistant(HA)](https://www.home-assistant.io/) with HA configuration.yaml triggering command line call of python script.

This reimplements similar functions but instead uses Node-Red.

<u>**Update Release 2: August 2021**</u>

[Wnaarifin repo](https://github.com/wnarifin/covid-19-malaysia) stopped daily updates since 3rd August 2021. I have re-work on the flows to retrieve the more comprehensive open data made available recently by [MOH](https://github.com/MoH-Malaysia/covid19-public) AND [CITF](https://github.com/CITF-Malaysia/citf-public).


## <a name="How">How It Works </a>
1. Node-Red pulls CSV from [MOH](https://github.com/MoH-Malaysia/covid19-public) AND [CITF](https://github.com/CITF-Malaysia/citf-public).

2. Due to limitation (or my lack of know-how) backdated data or remapping of data to dates cannot be posted in Home Assistant database for now - hence retreived and posted data to HA will be timestamped (to HA's time) at time of data posted to HA. 

3. Array pop() on function node returns last row; which would correspond to latest data. 

4. For state datasets; which contains at least 16 rows per date; on first `pop()` - the date for the data is stored and checked to ensure the remaining 15 rows retrieved have the same date before it is retrieved and stored. 

5. Rather than sending each data to HA as individual sensors - each dataset corresponding individual data is send to HA entities as attributes. 

   ![](https://raw.githubusercontent.com/anas-ivs/HA-NR-MYCovidStats/main/images/Sensor and Attributes-ha-nr-mycovidstats.PNG)

6. This however makes assessing the attributes data a little bit tricky where not all `lovelace` cards can directly display attributes - hence workaround required:

   1.  Custom lovelace panels for displaying attribute data; refer sample in Lovelace and pre-requisites.
   2.  Mapping only done when limited by what lovelace can call and would be more meaningful (look at the bright side - Daily vaccination statistics!).

7. Telegram node `/getcovidstats`  for manual call request and sets flags to identify at output to only send Telegram update when requested. 

## <a name="Pre">Pre-requisites </a>
1.  Home Assistant with Node-Red including [Node-Red companion integration](https://github.com/zachowj/hass-node-red); installed via [HACS](https://hacs.xyz/) to enable sensor creation from Node-Red. Refer my other writeup guide on howto.
2.  Optional - Telegrambot and Chat ID. Also ref writeup.

## <a name="Install">Installation</a>
1. Node-Red.
- Import [Flow](https://github.com/anasothman-myy/HA-NR-MYCovidStats/blob/main/ha-nr-mycovidstats.json) into Node Red (Upper Right burger stack -> Import).
- Ensure Telegrambots and Home Assistants node Servers are configured before clicking deploy.
- Click on `inject` node to test you have data.
> **Tip:** Use `link in`/`link out` to simplify and link repetitive task i.e. send to Telegram. 
- Additional: The missing `link-in` Telegram node can be imported [here](https://github.com/anasothman-myy/HA-NR-MYCovidStats/blob/main/telegram_output.json).
2. Home Assistant - Sensors

- Under`Configuration -> Integrations -> Node-Red`  following created entities should now be made available:
  - `sensor.covid19_my_daily_stats` from `cases_state.csv`
  - `sensor.covid19_my_hosp_stats` from `hospital.csv`
  - `sensor.covid19_my_icu_stats` from `icu.sv`
  - `sensor.covid19_my_kes_baharu` also from `cases_state.csv`
  - `sensor.covid19_my_kes_kematian` from `death_state.csv`
  - `sensor.covid19_my_pkrc_stats` from `pkrc.csv`
  - `sensor.covid19_my_test_samples` from `test_malaysia.csv`
  - `sensor.covid19_my_vax` from `vaxreg_malaysia`
    - `sensor.vaksin_dose1_daily`
    - `sensor.vaksin_dose1_cumul`
    - `sensor.vaksin_dose2_daily`
    - `sensor.vaksin_dose2_cumul`
    - `sensor.vaksin_total_dose_daily`
    - `sensor.vaksin_total_dose_cumulative`



3. Home Assistant - Lovelace

With this new (and complex) data set from MOH and CITF - there is more that can be done choose/crunch/omit. Inspired by 

For lovelace custom cards - install the following from HACS.

 - [Vertical stack in card](https://github.com/ofekashery/vertical-stack-in-card) to combine cards together without border.

- [Mini graph card](https://github.com/kalkih/mini-graph-card) for graph visualization.
- [Multiple Entity Row](https://github.com/benct/lovelace-multiple-entity-row) for display of multiple attributes.

3. Lovelace - Import and customize to your liking.

- Multiple Entry Row Version

![](https://raw.githubusercontent.com/anas-ivs/HA-NR-MYCovidStats/main/images/lovelace-ha-nr-mycovidstats-ver2-multiplentryrow.PNG)

```yaml
type: custom:vertical-stack-in-card
style: |
  ha-card {
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
  - type: custom:mini-graph-card
    name: Statistic COVID-19 Malaysia
    unit: +ve cases
    icon: mdi:virus
    hours_to_show: 168
    points_per_hour: 0.1
    group_by: date
    entities:
      - entity: sensor.covid19_my_kes_baharu
        name: Kes Baru
      - entity: sensor.covid19_my_kes_kematian
        name: Kes Kematian
      - entity: sensor.covid19_my_icu_stats
        name: Kes ICU
  - type: entities
    entities:
      - entity: sensor.covid19_my_kes_baharu
        type: custom:multiple-entity-row
        name: COVID Cases
        show_state: false
        styles:
          width: 80px
        secondary_info:
          attribute: date
          styles:
            font-weight: bold
        entities:
          - attribute: sarawak
            name: Sarawak Cases
          - entity: sensor.covid19_my_kes_baharu
            name: New Cases
          - entity: sensor.covid19_my_kes_kematian
            name: Death Cases
      - type: section
      - entity: sensor.covid19_my_hosp_stats
        type: custom:multiple-entity-row
        name: Hospitals
        state_header: COVID Patients
        styles:
          width: 80px
        secondary_info:
          attribute: date
          styles:
            font-weight: bold
        entities:
          - attribute: admitted_covid
            name: Admitted
            styles:
              width: 50px
          - attribute: discharged_covid
            name: Discharged
            styles:
              width: 50px
      - entity: sensor.covid19_my_pkrc_stats
        type: custom:multiple-entity-row
        name: PKRC
        state_header: PKRC Patients
        styles:
          width: 80px
        secondary_info:
          attribute: date
          styles:
            font-weight: bold
        entities:
          - attribute: total_admitted
            name: Admitted
            styles:
              width: 50px
          - attribute: total_discharged
            name: Discharged
            styles:
              width: 50px
      - entity: sensor.covid19_my_icu_stats
        type: custom:multiple-entity-row
        name: ICU
        state_header: ICU
        styles:
          width: 70px
        secondary_info:
          attribute: date
          styles:
            font-weight: bold
        entities:
          - attribute: total_beds_for_covid
            name: Total Beds
            styles:
              width: 60px
          - attribute: total_ventilator_covid
            name: On Ventilator
            styles:
              width: 65px
      - type: section
      - entity: sensor.covid19_my_test_samples
        type: custom:multiple-entity-row
        name: COVID Tests
        state_header: Total Tests
        styles:
          width: 80px
        secondary_info:
          attribute: date
          styles:
            font-weight: bold
        entities:
          - attribute: pcr
            name: PCR
            styles:
              width: 60px
          - attribute: rtk-ag
            name: RTK
            styles:
              width: 60px
      - type: section
      - entity: sensor.covid19_my_vax
        type: custom:multiple-entity-row
        name: Vaccination
        state_header: Daily Dose
        styles:
          width: 80px
        secondary_info:
          attribute: date
          styles:
            font-weight: bold
        entities:
          - attribute: daily_1st_dose
            name: 1st Dose
            styles:
              width: 60px
          - attribute: daily_2nd_dose
            name: 2nd Dose
            styles:
              width: 60px
  - type: custom:mini-graph-card
    entities:
      - entity: sensor.vaksin_total_dose_daily
        name: Daily Doses
        unit: injections
      - entity: sensor.vaksin_dose1_daily
        name: 1st Dose
      - entity: sensor.vaksin_dose2_daily
        name: 2nd Dose
    hours_to_show: 168
    points_per_hour: 0.1
    group_by: date


```



- Mini graph version 

  ![](https://raw.githubusercontent.com/anas-ivs/HA-NR-MYCovidStats/main/images/lovelace-ha-nr-mycovidstats-ver2-minigraph.PNG)

```yaml
type: custom:vertical-stack-in-card
style: |
  ha-card {
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
  - type: custom:mini-graph-card
    name: Statistic COVID-19 Malaysia
    unit: +ve cases
    icon: mdi:virus
    hours_to_show: 168
    points_per_hour: 0.1
    group_by: date
    entities:
      - entity: sensor.covid19_my_kes_baharu
        name: Kes Baru
  - type: vertical-stack
    cards:
      - type: grid
        cards:
          - type: custom:mini-graph-card
            entities:
              - color: black
                entity: sensor.covid19_my_kes_kematian
                unit: kes
            name: Kematian
            hours_to_show: 168
            group_by: date
            show:
              state: true
              fill: false
          - type: custom:mini-graph-card
            entities:
              - color: blue
                entity: sensor.covid19_my_test_samples
                unit: Samples
            name: Tests
            hours_to_show: 168
            group_by: date
            show:
              state: true
              fill: false
          - type: custom:mini-graph-card
            entities:
              - color: Yellow
                entity: sensor.covid19_my_hosp_stats
                unit: C19 Patients
            name: Hospital
            hours_to_show: 168
            group_by: date
            show:
              state: true
              fill: false
          - type: custom:mini-graph-card
            entities:
              - color: Green
                entity: sensor.covid19_my_vax
                unit: Shots
            name: Daily Shots
            hours_to_show: 168
            group_by: date
            show:
              state: true
              fill: false
          - type: custom:mini-graph-card
            entities:
              - color: Green
                entity: sensor.covid19_my_icu_stats
                unit: Patients
            name: ICU
            hours_to_show: 168
            group_by: date
            show:
              state: true
              fill: false
          - type: custom:mini-graph-card
            entities:
              - color: gold
                entity: sensor.covid19_my_pkrc_stats
                unit: kes
            name: PKRC Patients
            hours_to_show: 168
            group_by: date
            show:
              state: true
              fill: false
        columns: 2
        square: false



```



## <a name="Credits">Credit</a>

##### Credit to @wnarifin / @ A Jim Al

##### [Home Assistant Malaysia](https://www.facebook.com/groups/homeassistantmalaysia)
