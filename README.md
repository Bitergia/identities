# Format specification

This document is a work in progress. Its aim is to define a format to allow users to edit the data about contributors in an easy way. The use case below contains an example of the file that will be used to feed the dashboards.

The *profile* section is the fist one for each contributor. It contains:
* "name": this will be the named displayed on the dashboard
* "is_bot": a boolean flag to distinguish bot from humans. By default bots are filtered out on dashboard panels

```
- profile:
    name: Leonard Hofstadter
    is_bot: false
    avatar: http://gravatar.com/avatar/b7081d0131ad47821467b8e81434cf7a
    gender: male
    country: Spain
    city: Madrid
    lat/long: 40.4168N, 3.7038W
```

Some of the former fields could not be supported still by SortingHat, and will be ignored, but still they can be present in the profile information.

*enrollments* section includes the different companies the contributor has worked with. If you are not sure about the dates don't include them. In the example below Leonard contributions will be included in the buckets of two companies (depending on the date) and his contributions before 2013 will be assigned to the "Unknown" organization. Periods for companies should not overlap. Periods will always follow the rule:
 - start <= period, end > period
 - or stated as limits [start, end).

Therefore if the period is the whole year 2016, start could be 2016-01-01 and end could be 2017-01-01.

```
  enrollments:
    - organization: Example Company A
      start: 2013-01-01
      end: 2013-12-31
    - organization: Example Company B
      start: 2014-01-01
```

In order to group the different accounts in a single (and unified and shiny) identity include the different emails used by a contributor. Use *email* for that.
```
  email:
    - leoh@example.companyb.xyz
    - leoh@gmail.xyz
```

Due to the dashboard groups data from different data sources (github, stackoverflow, jira, git, gerrit ..) including the name used by the contributor will make the unification easier. In the example below we do it for *github*.

```
  github:
    - jsmanrique
```

You can also add this info for the current data sources: _askbot, bugzilla, confluence, discourse, gerrit, github, jira, mediawiki, meetup, phabricator, redmine, stackexchange, irc, telegram_. So just type the data source name and the account of the contributor.



# Use case.


* User sees a dashboard for Git, where there are two users for the same contributor.
** both of them belongs to Jose Manrique.
* It also sees the same contributor with a different name in (Github) Issues panel
* So, we create an entry in the identities.yml file for him grouping the info we know about thim

```
- profile:
    name: J. Manrique Lopez de la Fuente
  enrollments:
    - organization: Bitergia
      start: 2013-01-01
  github:
    - jsmanrique
  email:
    - jsmanrique@bitergia.com

# this is another entry
- profile:
    name: Luis Cañas-Díaz
  enrollments:
    - organization: Bitergia
      start: 2012-01-01T00:00:00
  github:
    - sanacl
  email:
    - lcanas@bitergia.com

# and another one for bots
- profile:
    is_bot: true
  email:
    - owlbot@bitergia.com
```

* this is how the information was stored in our database before loading the information from the identities file. We had two different unique identities for the same person

```
luis@hogaza ~/repos/mordred ±yaml_identities⚡ » sortinghat -u root --host mariadb -d grimoire_sh show --term manrique
unique identity 500338ea33b1ca700e557985167d1f71f50f0d70

Profile:
    * Name: J. Manrique Lopez de la Fuente
    * E-Mail: jsmanrique@bitergia.com
    * Bot: No
    * Country: -

Identities:
  500338ea33b1ca700e557985167d1f71f50f0d70	J. Manrique Lopez de la Fuente	jsmanrique@bitergia.com	-	git

Enrollments:
  Bitergia	1900-01-01 00:00:00	2100-01-01 00:00:00


unique identity 6ff5e662cf3165a12c4c0d779759dd030df1dca0

Profile:
    * Name: Manrique Lopez
    * E-Mail: jsmanrique@gmail.com
    * Bot: No
    * Country: -

Identities:
  6ff5e662cf3165a12c4c0d779759dd030df1dca0	Manrique Lopez	jsmanrique@gmail.com	-	git
  ad5d9099c8713f3eef3b27c7477340f076feb2b8	Manrique Lopez	-	jsmanrique	github

No enrollments
```
* and this is how it looks after we load the identities.yml file. Now the data is being refreshed on the enriched index, so the only name we see belongs to "J. Manrique Lopez de la Fuente"
* as we merged identities with overlapped affiliation for the company Bitergia, the tool chose the bigger one. This is something we are working on, in order to give priority to the information included in the file

```
file:bitergians@hogaza ~/repos/mordred ±yaml_identities⚡ » sortinghat -u root --host mariadb -d grimoire_sh show --term manrique
unique identity 48e7485e4cb3abbe60e469a858688978e4918647

Profile:
    * Name: J. Manrique Lopez de la Fuente
    * E-Mail: jsmanrique@gmail.com
    * Bot: No
    * Country: -

Identities:
  2caf6fbe3d56dfe65b44a4bf38d93e1e5a632bea	-	-	jsmanrique	file:bitergians
  48e7485e4cb3abbe60e469a858688978e4918647	J. Manrique Lopez de la Fuente	-	-	file:bitergians
  500338ea33b1ca700e557985167d1f71f50f0d70	J. Manrique Lopez de la Fuente	jsmanrique@bitergia.com	-	git
  6ff5e662cf3165a12c4c0d779759dd030df1dca0	Manrique Lopez	jsmanrique@gmail.com	-	git
  ad5d9099c8713f3eef3b27c7477340f076feb2b8	Manrique Lopez	-	jsmanrique	github
  b62c1edddc522c42773a094b68717adb819a29e2	Manrique Lopez	-	-	file:bitergians
  c1392e801694e3a9c3c3ebcf7bc277b873195883	-	jsmanrique@bitergia.com	-	file:bitergians

Enrollments:
  Bitergia	1900-01-01 00:00:00	2100-01-01 00:00:00

```
