# Format specification

This document is a work in progress. Our aim is to define a format to allow users to edit the data about contributors in an easy way. The use case below contains an example of the file that will be used to feed the dashboards.

( brief specification of the sections of the file will be included here ) 

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
      start: 2013-01-01T00:00:00
  name:
    - J. Manrique Lopez de la Fuente
    - Manrique Lopez
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
  name:
    - Luis Cañas Díaz
    - Luis Cañas-Díaz
  github:
    - sanacl
  email:
    - lcanas@bitergia.com
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
