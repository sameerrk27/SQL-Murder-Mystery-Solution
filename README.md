# SQL Murder Mystery — Solution

A walkthrough of my solution to the [SQL Murder Mystery](https://mystery.knightlab.com/), using SQL queries to investigate the crime, identify the murderer, and uncover the mastermind.

> **Spoilers ahead:** This README contains the solution.

## The case

On January 15, 2018, a murder took place in SQL City. I used the crime scene report, witness interviews, gym records, driver's license data, and event attendance records to identify the people involved.

## Part 1: Identify the murderer

### 1. Find the crime scene report

```sql
SELECT *
FROM crime_scene_report
WHERE date = 20180115
  AND type = 'murder'
  AND city = 'SQL City';
```

### 2. Find the witnesses

The report pointed to two witnesses: the person living at the last house on Northwestern Dr and Annabel on Franklin Ave.

```sql
-- People on Northwestern Dr, ordered by house number
SELECT *
FROM person
WHERE address_street_name = 'Northwestern Dr'
ORDER BY address_number;

-- Annabel on Franklin Ave
SELECT *
FROM person
WHERE address_street_name = 'Franklin Ave'
  AND name LIKE 'Annabel%';
```

### 3. Read the witness interviews

```sql
SELECT * FROM interview WHERE person_id = 14887;
SELECT * FROM interview WHERE person_id = 16371;
```

The interviews provided clues about the suspect's gym membership and vehicle.

### 4. Check gym records

I searched for members with IDs beginning with `48Z` who checked in on January 9, 2018, then looked up the matching members and people.

```sql
SELECT *
FROM get_fit_now_check_in
WHERE membership_id LIKE '48Z%'
  AND check_in_date = 20180109;

SELECT * FROM get_fit_now_member WHERE id = '48Z55';
SELECT * FROM get_fit_now_member WHERE id = '48Z7A';

SELECT * FROM person WHERE id = 28819;
SELECT * FROM person WHERE id = 67318;
```

### 5. Match the vehicle

```sql
SELECT *
FROM drivers_license
WHERE id = 423327;
```

### Result: The murderer

The murderer is **Jeremy Bowers**.

```sql
INSERT INTO solution VALUES (1, 'Jeremy Bowers');
SELECT value FROM solution;
```

## Part 2: Identify the mastermind

### 6. Read the murderer's interview

```sql
SELECT *
FROM interview
WHERE person_id = 67318;
```

The interview described a woman with red hair who drove a Tesla Model S.

### 7. Find people matching the description

```sql
SELECT *
FROM drivers_license
WHERE gender = 'female'
  AND hair_color = 'red'
  AND car_make LIKE 'Tesla%'
  AND car_model = 'Model S';

SELECT *
FROM person
WHERE license_id IN (918773, 291182, 202298);
```

### 8. Check concert attendance

```sql
SELECT *
FROM facebook_event_checkin
WHERE event_name = 'SQL Symphony Concert'
  AND person_id IN (78881, 90700, 99716);
```

### Result: The mastermind

The mastermind is **Miranda Priestly**.

```sql
INSERT INTO solution VALUES (1, 'Miranda Priestly');
SELECT value FROM solution;
```

## Combined queries

These queries combine the investigation steps to identify each person directly.

### Murderer

```sql
SELECT m.name
FROM get_fit_now_member m
JOIN get_fit_now_check_in c ON c.membership_id = m.id
JOIN person p ON p.id = m.person_id
JOIN drivers_license d ON d.id = p.license_id
WHERE m.id LIKE '48Z%'
  AND m.membership_status = 'gold'
  AND c.check_in_date = 20180109
  AND d.plate_number LIKE '%H42W%';
```

### Mastermind

```sql
SELECT p.name
FROM person p
JOIN drivers_license d ON d.id = p.license_id
JOIN facebook_event_checkin f ON f.person_id = p.id
WHERE d.gender = 'female'
  AND d.hair_color = 'red'
  AND d.car_make = 'Tesla'
  AND d.car_model = 'Model S'
  AND d.height BETWEEN 65 AND 67
  AND f.event_name = 'SQL Symphony Concert'
  AND f.date BETWEEN 20171201 AND 20171231
GROUP BY p.id
HAVING COUNT(*) = 3;
```

## Final answers

| Role | Person |
|---|---|
| Murderer | Jeremy Bowers |
| Mastermind | Miranda Priestly |
