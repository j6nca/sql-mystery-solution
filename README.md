# sql-mystery-solution
Solution for the SQL mystery https://mystery.knightlab.com/#experienced

## Prompt
A crime has taken place and the detective needs your help. The detective gave you the crime scene report, but you somehow lost it. You vaguely remember that the crime was a *murder* that occurred sometime on *Jan.15, 2018* and that it took place in *SQL City*. 

## SPOILER - DON'T REVIEW BELOW IF YOU DON'T WANT THE ANSWERS
## Solution

### Retrieving Crime Scene Report
We start with retrieving the reports of the crime using the information provided in the prompt.

`SELECT * FROM crime_scene_report WHERE date = 20180115 AND type = "murder" AND city = "SQL City"`
| date     | type   | description                                                                                                                                                                               | city     |
|----------|--------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| 20180115 | murder | Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave". | SQL City |

### Gathering Witness Information
Note to self: *last* house - is this could be the greatest or lowest when referring to last house on a street?

`SELECT * FROM person WHERE address_street_name = "Northwestern Dr" ORDER BY address_number DESC`

| id    | name            | license_id | address_number | address_street_name | ssn       |
|-------|-----------------|------------|----------------|---------------------|-----------|
| 14887 | Morty Schapiro  | 118009     | 4919           | Northwestern Dr     | 111564949 |
| 89906 | Kinsey Erickson | 510019     | 309            | Northwestern Dr     | 635287661 |

`SELECT * FROM person WHERE address_street_name = "Franklin Ave" AND name LIKE "%Annabel%"`

| id    | name           | license_id | address_number | address_street_name | ssn       |
|-------|----------------|------------|----------------|---------------------|-----------|
| 16371 | Annabel Miller | 490173     | 103            | Franklin Ave        | 318771143 |

With our witness information above we can use `person.id` to find interview transcripts and get more information.

`SELECT * FROM INTERVIEW WHERE person_id = 14887 OR person_id = 16371 OR person_id = 89906`

| person_id | transcript                                                                                                                                                                                                                      |
|-----------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 14887     | I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W". |
| 16371     | I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th.                                                                                                           |

### Investigating "Get Fit Now Gym"
We start by checking gym records that fit the descriptions given by the witnesses.

```
SELECT * FROM get_fit_now_member 
JOIN get_fit_now_check_in ON get_fit_now_member.id = get_fit_now_check_in.membership_id
WHERE id LIKE "48Z%" AND membership_status = "gold" AND check_in_date = "20180109"
```
| id    | person_id | name          | membership_start_date | membership_status | membership_id | check_in_date | check_in_time | check_out_time |
|-------|-----------|---------------|-----------------------|-------------------|---------------|---------------|---------------|----------------|
| 48Z7A | 28819     | Joe Germuska  | 20160305              | gold              | 48Z7A         | 20180109      | 1600          | 1730           |
| 48Z55 | 67318     | Jeremy Bowers | 20160101              | gold              | 48Z55         | 20180109      | 1530          | 1700           |

We get two names of interest both matching the gym criteria provided. We then cross reference this information with their `plate_number`s found in the `drivers_license` table.

```
SELECT * FROM drivers_license 
JOIN person ON drivers_license.id = person.license_id
WHERE plate_number LIKE "%H42W%" AND (person.id = 28819 OR person.id= 67318)
```
| id     | age | height | eye_color | hair_color | gender | plate_number | car_make  | car_model | id    | name          | license_id | address_number | address_street_name   | ssn       |
|--------|-----|--------|-----------|------------|--------|--------------|-----------|-----------|-------|---------------|------------|----------------|-----------------------|-----------|
| 423327 | 30  | 70     | brown     | brown      | male   | 0H42W2       | Chevrolet | Spark LS  | 67318 | Jeremy Bowers | 423327     | 530            | Washington Pl, Apt 3A | 871539279 |

There we have it, we've found our killer - it's Jeremy Bowers!

### Bonus - Who Hired Them?
Interrogating our culprit, we find out some details about their employer.

`SELECT * FROM INTERVIEW WHERE person_id = 67318`
| person_id | transcript                                                                                                                                                                                                                                       |
|-----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 67318     | I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017. |

```
SELECT * FROM drivers_license 
JOIN person ON drivers_license.id = person.license_id
JOIN facebook_event_checkin ON person.id = facebook_event_checkin.person_id
WHERE height BETWEEN 65 AND 67 AND event_name = "SQL Symphony Concert" GROUP BY person_id
HAVING COUNT(event_name) = 3
```

| id     | age | height | eye_color | hair_color | gender | plate_number | car_make  | car_model     | id    | name             | license_id | address_number | address_street_name      | ssn       | person_id | event_id | event_name           | date     |
|--------|-----|--------|-----------|------------|--------|--------------|-----------|---------------|-------|------------------|------------|----------------|--------------------------|-----------|-----------|----------|----------------------|----------|
| 391978 | 47  | 67     | green     | red        | male   | GUU6OT       | Dodge     | Caravan       | 49568 | Ramiro Matthias  | 391978     | 2032           | Hornchurch Grosvenor Way | 426605980 | 49568     | 1143     | SQL Symphony Concert | 20170520 |
| 457986 | 47  | 67     | green     | blue       | male   | H15O8I       | Chevrolet | Suburban 1500 | 99116 | Judson Morishito | 457986     | 1480           | Warehill Dr              | 723775388 | 99116     | 1143     | SQL Symphony Concert | 20180328 |
| 202298 | 68  | 66     | green     | red        | female | 500123       | Tesla     | Model S       | 99716 | Miranda Priestly | 202298     | 1883           | Golden Ave               | 987756388 | 99716     | 1143     | SQL Symphony Concert | 20171229 |

As Miranda Priestly is the only female in our result, we can conclude she hired Jeremy Bowers. Case closed!
