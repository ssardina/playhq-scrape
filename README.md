# PlayHQ Scrapper and TeamApp Game Schedule Builder

This repo contains Jupyter notebooks to scrape [PlayHQ](https://www.playhq.com) fixtures via its [REST API](https://support.playhq.com/hc/en-au/sections/4405422358297-PlayHQ-APIs) to:

1. Extract teams and their fixture in a competition, and save them into JSON files.
2. Produce CSV files for import as Schedule in a club's [TeamApp](https://www.teamapp.com) account. Using this, one can populate the schedule section for the next round or many rounds of games in minutes and with almost no manual work.

> [!NOTE]
> Before July 2022, the CSV file was built using the [`cba2csv`](cba2csv/) form spreadsheets provided by CBA (Basketball Association). Using PlayHQ REST API provides a much easier and robust solution, so this is not used anymore.

The script was originally done to support the [Brunswick Magic Basketball Club](https://www.brunswickmagic.com/) (Melbourne, Australia), from [Coburg Basketball Association (CBA)](https://coburgbasketball.org.au/). However, any club competing in CBA, or more generally under the [PlayHQ](https://bv.playhq.com/) system (even for other sports or countries!), should be able to use it.

- [PlayHQ Scrapper and TeamApp Game Schedule Builder](#playhq-scrapper-and-teamapp-game-schedule-builder)
  - [Notebooks available](#notebooks-available)
  - [Pre-requisites \& Setup](#pre-requisites--setup)
    - [Club Configuration File](#club-configuration-file)
  - [Scrape team fixtures from PlayHQ](#scrape-team-fixtures-from-playhq)
  - [TeamApps](#teamapps)
    - [Team Names and Access Groups](#team-names-and-access-groups)
    - [How to use it](#how-to-use-it)
  - [PlayHQ REST API via the shell](#playhq-rest-api-via-the-shell)
  - [Contact](#contact)

## Notebooks available

There are three notebooks in this repo:

1. `playhq_scrape_seasons.ipynb` - to extract the seasons available for a club, and get the season id from its name.
2. `playhq_scrape_fixture` **[MAIN NOTEBOOK!]**- to extract all teams in a season competition, and their fixtures. Each team fixture is saved into a separate JSON file. This can be used as the season progresses or after the season is finished, to get all the results of the games played.
3. `playhq2teamapp.ipynb` - to extract all teams and their fixtures in a competition, and produce a CSV file ready to be imported into TeamApp as Schedule.
    - This is usually used for a particular Club to handle all its teams in a competition.

## Pre-requisites & Setup

Tools run on Python 3.8+ as a Jupyter notebook, just run the following to get dependencies:

```shell
$ pip install -r requirements.txt
```

### Club Configuration File

The first thing to set-up, once per season, is the club configuration file, which defines a few variables. This configuration file will be designed at the start of the season and remained fixed.

Make a copy of the template [config_template.py](config_template.py), and complete it according to the Club's details.

Of particular importance are PlayHQ keys to access the Public API:

- Organization id. (e.g., "`8c4d5431-eaa5-4644-82ac-992abe224b88`")
- `x-api-key`: provided by PlayHQ support.
- `x-tenant`: the id of the sport (e.g., `bv` for Basketball Victoria).

There are also variables to set regarding the season name (as per PlayHQ), and text templates to use to populate descriptions fields in TeamApp.

To access the games in the PlayHQ admin system use:

```shell
https://bv.playhq.com/org/{$ORG_ID}/games?date=<YY_MM_DD>
```

> [!NOTE]
> If you want to use the `playhq2teamapp.ipynb` notebook, you will need to complete the TeamApp configuration section, which includes the text templates for the game description, and the mapping of PlayHQ team names to TeamApp team names.

## Scrape team fixtures from PlayHQ

The main notebook provided is [`playhq_scrape_fixture`](playhq_scrape_fixture.ipynb), which will extract all teams in a season competition, and their fixtures. Each team fixture is saved into a separate JSON file.


## TeamApps

The notebook to translate PlayHQ fixtures to TeamApp CSV file, `playhq2teamapp.ipynb`, will generate a CSV file ready to be imported into TeamApp as Schedule. This is done by using the [TeamApp](https://www.teamapp.com/) web interface, in the Schedule section.

> [!NOTE]
> Our Brunswick Magic club stopped using it in 2025, as fixtures can now be communicated via [MyHoops](https://get.playhq.com/apps/myhoops) app (which is the official mobile app for basketball in Australia.).

### Team Names and Access Groups

The system assumes _team names_ match _access groups_ names in the TeamApp account of the club. This is important because the CSV file generated will include such team and access group names.

These names generally be very similar to the ones registered in PlayHQ. For example, PlayHQ may have a team "`Magics U14 Boys Gold`", and the club may just use "`U14 Boys Gold`" for the corresponding team AND access group names. A function `tapp_team_name(team_name)` will translate PlayHQ team names to those names in TeamApp (read below).

### How to use it

To run the system, use notebook [playhq2teamapp.ipynb](playhq2teamapp.ipynb) and change/configure the first section, with the name of the configuration file of the club and the interval dates you want to extract games from. You can get the season id at the start of each season using notebook [playhq_scrape_seasons.ipynb](playhq_scrape_seasons.ipynb).

Then, run the system cell by cell, checking on the way all is consistent.

In the last steps, it will generate a `CSV` file ready to be up imported to TeamApp in the Schedule section.

## PlayHQ REST API via the shell

This system uses PlayHQ public REST API:

- [PlayHQ API documentation](https://support.playhq.com/hc/en-au/sections/4405422358297-PlayHQ-APIs).
- [PlayHQ API's technical documentation](https://docs.playhq.com/tech).

As stated above, one needs the organization club's id (public), together with the `x-api-key` (private) and `x-tenant` (the id of the sport, `bv` for Basketball Victoria). In addition, to get the fixtures one would need the season name as it appears in PlayHQ (e.g., "Winter 2022").

First set-up your API key and ORG id in environment variables:

```shell
export API_KEY="........."
export ORG_ID="......."
```

We start by **extracting the active seasons for the club**.

We start by **extracting the active seasons for the club** (using its organisation id). This can be done using the [playhq_scrape_seasons.ipynb](playhq_scrape_seasons.ipynb) notebook.

```shell
$ curl -H "x-phq-tenant: bv" -H "x-api-key: $API_KEY" https://api.playhq.com/v1/organisations/$ORG_ID/seasons

{
    "data": [
        {
            "id": "a94981b4-75b7-429f-9005-915182ab6153",
            "name": "Summer 2022/23",
            "status": "UPCOMING",
            "association": {
                "id": "e18085db-c6de-512f-b78b-53d253c65b32",
                "name": "Coburg Basketball Association",
                "url": "https://www.playhq.com/basketball-victoria/org/coburg-basketball-association/e18085db",
                "logo": {
                    "sizes": [
                        {
                            "url": "https://res.cloudinary.com/playhq/image/upload/h_32,w_32/v1/production/bv/e18085db-c6de-512f-b78b-53d253c65b32/1645069189211/logo.jpg",
                            "dimensions": {
                                "width": 32,
                                "height": 32
                            }
                        },
                        {
                            "url": "https://res.cloudinary.com/playhq/image/upload/h_48,w_48/v1/production/bv/e18085db-c6de-512f-b78b-53d253c65b32/1645069189211/logo.jpg",
                            "dimensions": {
                                "width": 48,
                                "height": 48
                            }
                        },
                        {
                            "url": "https://res.cloudinary.com/playhq/image/upload/h_128,w_128/v1/production/bv/e18085db-c6de-512f-b78b-53d253c65b32/1645069189211/logo.jpg",
                            "dimensions": {
                                "width": 128,
                                "height": 128
                            }
                        },
                        {
                            "url": "https://res.cloudinary.com/playhq/image/upload/h_64,w_64/v1/production/bv/e18085db-c6de-512f-b78b-53d253c65b32/1645069189211/logo.jpg",
                            "dimensions": {
                                "width": 64,
                                "height": 64
                            }
                        },
                        {
                            "url": "https://res.cloudinary.com/playhq/image/upload/h_96,w_96/v1/production/bv/e18085db-c6de-512f-b78b-53d253c65b32/1645069189211/logo.jpg",
                            "dimensions": {
                                "width": 96,
                                "height": 96
                            }
                        },
                        {
                            "url": "https://res.cloudinary.com/playhq/image/upload/h_256,w_256/v1/production/bv/e18085db-c6de-512f-b78b-53d253c65b32/1645069189211/logo.jpg",
                            "dimensions": {
                                "width": 256,
                                "height": 256
                            }
                        }
                    ]
                }
            },
            "competition": {
                "id": "81fe8d81-d087-4a63-b4fb-87bc85e9d8ac",
                "name": "Junior Domestic"
            },
            "createdAt": null,
            "updatedAt": null
        }
    ],
    "metadata": {
        "hasMore": false,
        "nextCursor": null
    }
}
```

Season Summer 22/23 has id `a94981b4-75b7-429f-9005-915182ab6153`. We can then **extract all the teams** registered for that season:

```shell
$ curl -H "x-phq-tenant: bv" -H "x-api-key: $API_KEY" https://api.playhq.com/v1/seasons/a94981b4-75b7-429f-9005-915182ab6153/teams
```

This will yield all the teams in the season. We can extract the club's teams by matching their orgasnisation id. Each team will have an id, for example, team "Magic U10 Boys Gold" has id `bce059fc-67a0-4167-80ee-c2a8fcf4faad`.

We can then **extract all the team fixture** published so far (this may include past, and future games):

```shell
$ curl -H "x-phq-tenant: bv" -H "x-api-key: $API_KEY" https://api.playhq.com/v1/teams/bce059fc-67a0-4167-80ee-c2a8fcf4faad/fixture

{
    "data": [
        {
            "id": "c8d97a11-a603-43bf-b6c7-ea16157d505f",
            "status": "UPCOMING",
            "url": "https://www.playhq.com/basketball-victoria/org/coburg-basketball-association/junior-domestic-summer-2022-23/saturday-u-10-boys-division-4/game-centre/c8d97a11",
            "createdAt": "2022-10-03T01:00:35.000Z",
            "updatedAt": "2022-10-03T01:00:35.000Z",
            "grade": {
                "id": "196be9f8-d7ff-44d2-8989-7f0a90291a5a",
                "name": "Saturday U10 Boys Division 4",
                "url": "https://www.playhq.com/basketball-victoria/org/coburg-basketball-association/junior-domestic-summer-2022-23/saturday-u-10-boys-division-4/196be9f8"
            },
            "round": {
                "id": "334cb7c9-e49e-4096-aace-4bece285d9a6",
                "name": "Round 1",
                "abbreviatedName": "R1",
                "isFinalRound": false
            },
            "pool": null,
            "schedule": {
                "date": "2022-10-08",
                "time": "09:15:00",
                "timezone": "Australia/Melbourne"
            },
            "competitors": [
                {
                    "id": "bce059fc-67a0-4167-80ee-c2a8fcf4faad",
                    "name": "Magic U10 Boys Gold",
                    "isHomeTeam": false
                },
                {
                    "id": "29a5de69-13fe-41e5-99f7-3d1c65cd1828",
                    "name": "Warriors U10 Boys Gold",
                    "isHomeTeam": true
                }
            ],
            "venue": {
                "id": "84783227-07b3-4e74-906e-f318b26b5ea3",
                "name": "Coburg Basketball Stadium",
                "surfaceName": "Court 3",
                "surfaceAbbreviation": "Crt3",
                "address": {
                    "line1": "25 Outlook Road",
                    "postcode": "3058",
                    "suburb": "Coburg North",
                    "state": "VIC",
                    "country": "Australia",
                    "latitude": -37.73315,
                    "longitude": 144.97684
                }
            }
        }
    ],
    "metadata": {
        "hasMore": true,
        "nextCursor": "MaMx"
    }
}
```

The above JSON reply has been formatted for better legibility.

The last `metadata` information of each JSON reply, states whether there are more follow-up pages using a cursor parameter. To get the next page:

```shell
$ curl -H "x-phq-tenant: bv" -H "x-api-key: $API_KEY" https://api.playhq.com/v1/seasons/a94981b4-75b7-429f-9005-915182ab6153/teams?cursor=MaMx
```

The above interaction is done in Python via function `get_json(self, key)`, which provides an iterator with the various response pages, one by one.

## Contact

All scripts have been developed by Sebastian Sardina (ssardina@gmail.com).
