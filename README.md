# SQL Murder Mistery

A crime has taken place and the detective needs your help. The detective gave you the crime scene report, but you somehow lost it. You vaguely remember that the crime was a ​**murder**​ that occurred sometime on **​Jan.15, 2018**​ and that it took place in **​SQL City​**. Start by retrieving the corresponding crime scene report from the police department’s database.

![Schema](./schema.png "SQL Schemas")

So, we need to search for a murder on 2018-01-15 in SQL City, right?

```sql
select * from crime_scene_report where date = 20180115 and city = 'SQL City';
```


| date     | type   | description                                                                                                                                                                               | city     |
| -------- | ------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| ... | ... | ... | ... |
| 20180115 | murder | Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave". | SQL City |

Now follow the path, we need to looking for witnesses...

```sql
select * from person where address_street_name = 'Northwestern Dr'
union
select * from person where name like '%Annabel%' and address_street_name = 'Franklin Ave'
order by address_street_name, address_number DESC;
```


| id | name | license_id | address_number | address_street_name | ssn |
| -- | ---- | ---------- | -------------- | ------------------- | --- |
| 16371 | Annabel Miller | 490173 | 103 | Franklin Ave | 318771143 |
| 14887 | Morty Schapiro | 118009 | 4919 | Northwestern Dr | 111564949 |
| ... | ... | ... | ... | ... | ... |


Okay, now we have witnesses, searching for their interviews...

```sql
select * from interview where person_id in (16371, 14887);
```


| person_id | transcript                                                                                                                                                                                                                      |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 14887     | I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W". |
| 16371     | I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th.                                                                                                           |

The answer is here, murderer is a gym member of Get Fit Now Gym, he had gold member card, drive a car with that license plate, and go to the gym on 2018-01-09

```sql
select
	c.check_in_date as inDate,
	c.check_in_time as inTime,
	c.check_out_time as outTime,
	m.membership_start_date as memStartDate,
	m.membership_status as memStatus,
	p.id as perId,
	p.name as perName,
	d.plate_number as plateNum
from get_fit_now_check_in as c
	join get_fit_now_member as m on c.membership_id = m.id
	join person as p on m.person_id = p.id
	join drivers_license as d on p.license_id = d.id
where
	m.membership_status = 'gold'
	and d.plate_number like '%H42W%'
	and c.check_in_date = 20180109;
```


| inDate   | inTime | outTime | memStartDate | memStatus | perId | perName       | plateNum |
| -------- | ------ | ------- | ------------ | --------- | ----- | ------------- | -------- |
| 20180109 | 1530   | 1700    | 20160101     | gold      | 67318 | Jeremy Bowers | 0H42W2   |

We only have 1 man. He is the murderer.

Congrats, you found the murderer! But wait, there's more... If you think you're up for a challenge, try querying the interview transcript of the murderer to find the real villain behind this crime. If you feel especially confident in your SQL skills, try to complete this final step with no more than 2 queries. Use this same INSERT statement with your new suspect to check your answer.

```sql
select * from interview where person_id = 67318;
```


| person_id | transcript                                                                                                                                                                                                                                       |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 67318     | I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017. |

Okay, so now we are looking for a woman, height from 65" to 67", red hair, drives a Tesla Model S, rich, came to SQL Symphony Concert 3 times in last December...

```sql
select
	d.gender,
	d.hair_color as hair,
	d.height,
	f.event_name as event,
	f.date,
	p.id,
	p.name,
	i.annual_income as income,
	d.car_make,
	d.car_model as model
from facebook_event_checkin as f
	join person as p on f.person_id = p.id
	join income as i on p.ssn = i.ssn
	join drivers_license as d on p.license_id = d.id
where
	d.gender = 'female'
	and d.hair_color = 'red'
	and d.height >= 65
	and d.height <= 67
	and f.date >= 20171201
	and f.date <= 20171231
	and d.car_make = 'Tesla'
	and d.car_model = 'Model S'
order by
	d.gender,
	d.hair_color,
	d.height DESC;
```


| gender | hair | height | event                | date     | id    | name             | income | car_make | model   |
| ------ | ---- | ------ | -------------------- | -------- | ----- | ---------------- | ------ | -------- | ------- |
| female | red  | 66     | SQL Symphony Concert | 20171206 | 99716 | Miranda Priestly | 310000 | Tesla    | Model S |
| female | red  | 66     | SQL Symphony Concert | 20171212 | 99716 | Miranda Priestly | 310000 | Tesla    | Model S |
| female | red  | 66     | SQL Symphony Concert | 20171229 | 99716 | Miranda Priestly | 310000 | Tesla    | Model S |

Now we know the name who hired this gymmer to do the dirty work...

Congrats, you found the brains behind the murder! Everyone in SQL City hails you as the greatest SQL detective of all time. Time to break out the champagne!
