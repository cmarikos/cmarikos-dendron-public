In 2024 RAZE registered 5,352 voters

Query here:
```SQL
SELECT 
  COUNT(Voter_File_VANID) 
FROM `prod-organize-arizon-4e1c0a83.viewers_dataset.RAZE_2024` 
```

When we match to the Targetsmart file, we lose 1,422 people, and come out with a total of 3,930 registered voters that we can find.

Here is a query to decipher 2024 general election vote history
```SQL
SELECT  
  CASE
      WHEN vhsyn_vf_g2024_synthetic IN ('A','E','Y') THEN 'Voted'
      ELSE 'Did not vote'
    END AS vote_history
  ,COUNT(vb.voterbase_id) AS voter_count
FROM `prod-organize-arizon-4e1c0a83.viewers_dataset.RAZE_2024` AS vr

LEFT JOIN `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest` AS vb
  ON vr.Voter_File_VANID = vb.vb_smartvan_id

LEFT JOIN `prod-organize-arizon-4e1c0a83.targetsmart_AZ.vote_history_synthetics_latest` AS vh
  ON vb.voterbase_id = vh.voterbase_id

GROUP BY 1
```

Results:

| vote_history | voter_count |
| ------------ | ----------- |
| Did not vote | 2014        |
| Voted        | 1916        |

Doing a quick/rough tsmr_race breakdown
```SQL
SELECT  
  vb.ts_tsmr_race
  , COUNT(vb.voterbase_id) AS voter_count
FROM `prod-organize-arizon-4e1c0a83.viewers_dataset.RAZE_2024` AS vr

LEFT JOIN `prod-organize-arizon-4e1c0a83.targetsmart_AZ.voter_base_latest` AS vb
  ON vr.Voter_File_VANID = vb.vb_smartvan_id

LEFT JOIN `prod-organize-arizon-4e1c0a83.targetsmart_AZ.vote_history_synthetics_latest` AS vh
  ON vb.voterbase_id = vh.voterbase_id

WHERE vb.ts_tsmr_race IS NOT NULL

GROUP BY 1
```

| ts_tsmr_race | voter_count |
| ------------ | ----------- |
| N            | 13          |
| H            | 2748        |
| W            | 1072        |
| B            | 72          |
| A            | 14          |
| U            | 11          |
Wondering if we're running into the voter missingness reported by Miriam in the following. Would love to dive deeper. It would be helpful to know if there is missingness vs purges/people not successfully getting registered -- how to investigate this?

![[Surfacing Missing Voters 2.12.24.pdf]]

Clarification from Hal on the match issue
"So the issues with the Blocks to VAN sync is that it goes through two points of transfer as well.Once the OCR digitizes the forms, it goes back into blocks then State Voices would send a blocks file to VVN who is the only org nationwide who can bulk apply ACs via API to the VRT, then from the VRT it matches over to MyV via those codes we make every year, so that’s the last two kind of drop off points.So a couple things could be happening:  

1. Yes, voters may not have made it all the way into the voter file, our coalition averages about an 80% match rate
2. if you’re pulling based on those activists codes Folks could have dropped off.  What we could do is take your blocks list and have TargetSmart match via their voter base ID and bulk upload those people and retag them with the activist code

Hal Boyles

So it’s likely a combo of those two things rather than one option being solely responsible for that large of a percent of the blocks list"
