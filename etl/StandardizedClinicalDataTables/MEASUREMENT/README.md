# Link to CommonDataModel
- [MEASUREMENT](https://github.com/OHDSI/CommonDataModel/wiki/MEASUREMENT)

# You must filter your queries with data types  
- ie the `measurement_type_concept_id` column!

# Source Tables (mimic)

- when measures are mapped to standard concepts (SMOMED) `measurement_concept_id
 != 0`, and you may use the non-omop mimic itemid
- else `measurement_concept_id` = 0, and you should select measures with: `measurement_source_concept_id`
- you can determine the `measurement_source_concept_id` by looking in the `concept` table, where `concept_code` is the MIMIC `itemid`

- when `visit_detail_id` is assigned, this is a calculated value!

## [charteevents](https://mimic.physionet.org/mimictables/chartevents/)

- rows in error have not been exported from mimic

## [labevents](https://mimic.physionet.org/mimictables/labevents/)

- logical Observation Identifiers Names and Codes (LOINC) = database and universal standard for identifying medical laboratory observations loinc.csv
- abnormal values from labevents do not have any equivalent in OMOP. The ranges should be prefered, but are not yet provided in MIMICt 

## [microbiologyevents](https://mimic.physionet.org/mimictables/microbiologyevents/)

## [outputevents](https://mimic.physionet.org/mimictables/outputevents/)

## [inputevents_mv](https://mimic.physionet.org/mimictables/inputevents_mv/)

- the patient weigth has a `measurement_type_concept_id` = 44818701 (from physical examination) and a `concept_id` = 3025315 (body weight)
- this is similar to the weight comming from chartevents

# Mapping used for `measurement_concept_id`

## charteevents

- https://github.com/MIT-LCP/mimic-omop/blob/master/extras/concept/chart_label_to_concept.csv

## labevents

- https://github.com/MIT-LCP/mimic-omop/blob/master/extras/concept/lab_label_to_concept.csv
- https://github.com/MIT-LCP/mimic-omop/blob/master/extras/concept/lab_unit_to_concept.csv
- https://github.com/MIT-LCP/mimic-omop/blob/master/extras/concept/lab_value_to_concept.csv

## microbiologyevents

- https://github.com/MIT-LCP/mimic-omop/blob/master/extras/concept/atb_to_concept.csv

## outputevents

- https://github.com/MIT-LCP/mimic-omop/blob/master/extras/concept/output_label_to_concept.csv

# Derived value

- derived values, should be populated with time
- the following have already been done 
    - corrected calcemia, kaliemia, P/F ratio, corrected osmolarity from chartevents and labevents
    - Weight, height, LVEF from noteevents
    - BMI from chartevents and noteevents
- [derived_values](https://github.com/MIT-LCP/mimic-omop/blob/master/extras/concept/derived_to_concept.csv)

# Examples

## Introduction

### different date types

``` sql
-- you must filter your queries with data types !
SELECT concept_name, concept_id, COUNT(1)
FROM measurement
JOIN concept ON measurement_type_concept_id = concept_id
GROUP BY concept_name, concept_id ORDER BY COUNT(1) DESC;
```
|        concept_name       | concept_id |   count |
|---------------------------|------------|-----------|
|From physical examination  |   44818701 | 257715284|
|Labs - Chemistry           | 2000000011 |   7526424|
|Output Event               | 2000000003 |   4349218|
|Labs - Hemato              | 2000000009 |   3023959|
|Labs - Blood Gaz           | 2000000010 |   1161954|
|Derived value              |   45754907 |   1020678|
|Labs - Culture Organisms   | 2000000007 |    363506|
|Labs - Culture Sensitivity | 2000000008 |    267350|
|Lab result                 |   44818702 |      5032|

## different operator types

``` sql
SELECT distinct operator_concept_id, concept_name
FROM measurement
JOIN concept on operator_concept_id = concept_id;
```
| operator_concept_id | concept_name|
|---------------------|--------------|
|             4172704 | >|
|             4171754 | <=|
|             4171756 | <|
|             4171755 | >=|
|             4172703 | =|

## From physical examination

### the most frequent items, by using omop concept_id

``` sql
SELECT concept_name, concept_id, count (1)
FROM measurement
JOIN concept ON measurement_concept_id = concept_id
WHERE measurement_type_concept_id = 44818701                      		-- concept_name = 'From physical examination'
GROUP BY concept_name, concept_id ORDER BY count(concept_name) DESC
LIMIT 10;
```
|            concept_name             | concept_id |   count|
|-------------------------------------|------------|-----------|
| No matching concept                 |          0 | 181101521|
| Respiratory rate                    |    3024171 |   9474378|
| Heart rate                          |    3027018 |   7942776|
| Oxygen saturation in Arterial blood |    3016502 |   7807775|
| BP systolic                         |    3004249 |   6421226|
| BP diastolic                        |    3012888 |   6397068|
| Mean blood pressure                 |    3027598 |   6379838|
| Body temperature                    |    3020891 |   3964808|
| Body weight                         |    3025315 |   3729554|
| Heart rate rhythm                   |    3022318 |   3303151|

### the most frequent items, by using non omop mimic itemid

``` sql
SELECT concept_name, concept_code, count (1)
FROM measurement
JOIN concept ON measurement_source_concept_id = concept_id
WHERE measurement_type_concept_id = 44818701                      		-- concept_name = 'From physical examination'
GROUP BY concept_name, concept_code ORDER BY count(concept_name) DESC
LIMIT 10;
```
|            concept_name             | concept_id |   count|
|-------------------------------------|------------|--------|
| Heart Rate                          |        211 | 5180809|
| calprevflg                          |        742 | 3464326|
| SpO2                                |        646 | 3418917|
| Respiratory Rate                    |        618 | 3386719|
| Heart Rhythm                        |        212 | 3303151|
| Ectopy Type                         |        161 | 3236350|
| Code Status                         |        128 | 3216866|
| Precautions                         |        550 | 3205052|
| Service Type                        |       1125 | 2955851|
| Heart Rate                          |     220045 | 2762225|

### weights, by using omop `measurement_concept_id`

``` sql
SELECT MIN(value_as_number), AVG(value_as_number), MAX(value_as_number)
FROM measurement
WHERE measurement_concept_id = 3025315                                          -- concept.concept_name = 'Body weight'
AND measurement_type_concept_id = 44818701;                                     -- concept_name = 'From physical examination'
```
|    min     |         avg         | max |
|------------|---------------------|-----|
| 44.1798608 | 83.8148454007945714 | 170|

### weights, by using non omop mimic items
``` sql
SELECT MIN(value_as_number), AVG(value_as_number), MAX(value_as_number)
FROM measurement JOIN concept ON measurement_source_concept_id = concept_id
WHERE concept_code = '762' or concept_code = '226531' or concept_code = '226512'
AND measurement_type_concept_id = 44818701;                                     -- concept_name = 'From physical examination'
```
|    min     |         avg         | max |
|------------|---------------------|-----|
| 44.1798608 | 81.5634084520612165 | 170|

## From labs

### the most frequent items
``` sql
SELECT measurement_source_value, count (1)
FROM measurement
WHERE measurement_type_concept_id = 2000000009                                  -- concept_name = 'Labs - Hemato'
GROUP BY measurement_source_value ORDER BY count(1) desc 
LIMIT 10;
```
| measurement_source_value | count |
|--------------------------|-------|
| Hematocrit               |  2710|
| Hemoglobin               |  2559|
| Platelet Count           |  2096|
| White Blood Cells        |  1736|
| MCHC                     |  1728|
| MCH                      |  1727|
| Red Blood Cells          |  1727|
| MCV                      |  1727|
| RDW                      |  1726|
| PTT                      |  1315|

### white blood cells values by using LOINC classification, ie 804-5 or 26464-8

``` sql
SELECT MIN(value_as_number), AVG(value_as_number), MAX(value_as_number)
FROM measurement 
JOIN concept ON measurement_source_concept_id = concept_id
WHERE concept_name ILIKE '%loinc:[804-5]' OR concept_name ILIKE '%loinc:[26464-8]'
WHERE measurement_type_concept_id = 2000000009                                  -- concept_name = 'Labs - Hemato'
```
| min |        avg         | max  |
|-----|--------------------|------|
| 2.2 | 8.2142857142857143 | 23.5|

### white blood cells value by non omop mimic items

``` sql
SELECT MIN(value_as_number), AVG(value_as_number), MAX(value_as_number)
FROM measurement JOIN concept on measurement_source_concept_id = concept_id
WHERE concept_code = '51300' or concept_code = '51301'
WHERE measurement_type_concept_id = 2000000009                                  -- concept_name = 'Labs - Hemato'
```
| min |        avg         | max  |
|-----|--------------------|------|
| 2.2 | 8.2142857142857143 | 23.5|

## From cultures

### Nb of negative blood cultures

```sql
SELECT count(1)
FROM measurement
where measurement_type_concept_id = 2000000007   				-- concept.concept_name = 'Labs - Culture Organisms'
and value_as_concept_id = 9189;                  				-- concept.concept_name = 'Negative'
```
 count
--------
 287523

### Main place of collection

``` sql
SELECT concept_name, concept_id
FROM concept
WHERE concept_id IN
	 (SELECT distinct measurement_concept_id
         FROM measurement
         WHERE measurement_type_concept_id = 2000000007)                       -- concept.concept_name = 'Labs - Culture Organisms'
LIMIT 10;
```
|                                                            concept_name                                                            | concept_id |
|------------------------------------------------------------------------------------------------------------------------------------|------------|
| Bacteria identified in Unspecified specimen by Culture                                                                             |    3002619|
| Virus identified in Unspecified specimen by Immunofluorescence                                                                     |    3006753|
| Bacteria identified in Synovial fluid by Culture                                                                                   |    3006761|
| Fungus identified in Blood by Culture                                                                                              |    3009171|
| Bacteria identified in Catheter tip by Culture                                                                                     |    3009986|
| Toxoplasma gondii Ab [Presence] in Serum                                                                                           |    3011392|
| Bacteria identified in Bronchial specimen by Aerobe culture                                                                        |    3015532|
| Bacteria identified in Body fluid by Culture                                                                                       |    3016727|
| Cytomegalovirus IgG Ab [Presence] in Serum                                                                                         |    3016816|
| Bacteria identified in Cerebral spinal fluid by Culture                                                                            |    3016914|
| Bacteria identified in Sputum by Culture                                                                                           |    3023419|

###  most frequent microorganisms in blood culture

``` sql
SELECT concept_name, value_as_concept_id
FROM measurement m
JOIN concept c on c.concept_id = m.value_as_concept_id
WHERE measurement_type_concept_id = 2000000007        				-- concept.concept_name = 'Labs - Culture Organisms'
And value_as_concept_id != 9189                       				-- concept.concept_name = 'Negative'
AND measurement_concept_id = 46235217                 				-- concept.concept_name = 'Bacteria identified in Blood product unit.autologous by Culture'
GROUP BY concept_name, value_as_concept_id order by count(1) desc;
```
|            concept_name            | value_as_concept_id |
|------------------------------------|---------------------|
| Staphylococcus, coagulase negative |             4020318 |
| Klebsiella pneumoniae              |             4209452 |
| Staphylococcus aureus              |             4149419 |
| Pseudomonas aeruginosa             |             4198675 |
| Enterococcus raffinosus            |             4011481 |
| Acinetobacter baumannii            |             4235587 |
| Bacteroides fragilis group         |             4213634 |
| Corynebacterium                    |             4299363 |

## resistance profile for Staph. Aureus

``` sql
-- fact_id_1 is always the organisms and fact_id_2 the antibiotics tested
SELECT measurement_source_value, value_as_concept_id, concept_name
FROM measurement
JOIN concept resistance ON value_as_concept_id = concept_id
JOIN fact_relationship ON measurement_id =  fact_id_2
JOIN
(
	SELECT measurement_id AS id_is_staph
	FROM measurement m
	WHERE measurement_type_concept_id = 2000000007        			-- concept.concept_name = 'Labs - Culture Organisms'
	AND value_as_concept_id = 4149419                     			-- concept.concept_name = 'staph aureus coag +'
	AND measurement_concept_id = 46235217               			-- concept.concept_name = 'Bacteria identified in Blood product unit.autologous by Culture';
) staph ON id_is_staph = fact_id_1;
WHERE measurement_type_concept_id = 2000000008        			        -- concept.concept_name = 'Labs - Culture Sensitivity'
```
| measurement_source_value | value_as_concept_id | concept_name |
|--------------------------|---------------------|--------------|
| ERYTHROMYCIN             |             4148441 | Resistant|
| ERYTHROMYCIN             |             4148441 | Resistant|
| LEVOFLOXACIN             |             4148441 | Resistant|
| OXACILLIN                |             4148441 | Resistant|
| PENICILLIN               |             4148441 | Resistant|
| PENICILLIN               |             4148441 | Resistant|
| OXACILLIN                |             4038110 | Susceptible|
| RIFAMPIN                 |             4038110 | Susceptible|
| TETRACYCLINE             |             4038110 | Susceptible|
| VANCOMYCIN               |             4038110 | Susceptible|
| CLINDAMYCIN              |             4038110 | Susceptible|
| GENTAMICIN               |             4038110 | Susceptible|
| GENTAMICIN               |             4038110 | Susceptible|
| LEVOFLOXACIN             |             4038110 | Susceptible|

## organism and antibiotic resistance

``` sql
SELECT org.value_source_value, atb.measurement_source_value, resistance.concept_name
FROM
(
    SELECT fact_id_1, fact_id_2 from fact_relationship where fact_id_1 IN 
    (
        SELECT measurement_id 
        FROM measurement
        WHERE measurement_type_concept_id = 2000000007
        AND value_as_concept_id != 9189
    )
) as fact
JOIN measurement org ON org.measurement_id = fact.fact_id_1 and org.measurement_type_concept_id = 2000000007
JOIN measurement atb ON atb.measurement_id = fact.fact_id_2 and atb.measurement_type_concept_id = 2000000008
JOIN concept resistance ON resistance.concept_id = atb.value_as_concept_id
LIMIT 10;
```
