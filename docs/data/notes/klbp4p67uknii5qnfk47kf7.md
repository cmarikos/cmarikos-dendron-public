On Thu, Aug 21, 2025 at 8:10 AM Erin Zamora <[erin@youthengagementfund.org](mailto:erin@youthengagementfund.org)> wrote:  

> Dear Sherri and the Rural Arizona Engagement Team,
> 
>   
> 
> Thank you for the timely submission of your application. The YEF Team appreciates the effort and care that your team is putting into this submission process.
> 
>   
> To ensure your application is processed in a timely manner, please complete the [Updated Diversity Table](https://docs.google.com/spreadsheets/d/1-_KKROw2fRfQl93w3h48KviC6K3N4YKLR7Rdd7Ohxns/edit?usp=sharing). Once updated, please change the date on the document to reflect the new submission date.  
>   
> Our team asks that you keep in mind these instructions as you review the diversity table.  
> 
> - Please ensure that the totals in each category, gender, race + ethnicity and age are equal to the total number of staff at the top of each column (Row 2, Columns B-H). 
> 
> - While we are trying to collect as complete of a dataset as possible, if you don’t have complete data for some of these categories or do not want to disclose, please put those numbers in the No Data/Prefer Not to Disclose row. Including the ‘No Data’ selections, the total in each category should still equate to the total number of staff in Row 2, Columns B-H.
> 
>   
> The deadline for edits to the [Updated Diversity Table](https://docs.google.com/spreadsheets/d/1-_KKROw2fRfQl93w3h48KviC6K3N4YKLR7Rdd7Ohxns/edit?usp=sharing) is **August 28, 2025 at 5:00 pm PT / 8:00 pm ET.**  
>   
> While the YEF team will be on our annual summer break from August 25-29, Cassandra Stafford will be available to answer any questions you may have about the diversity table. I will return on September 2 to review your submission.   
>   
> 
> Thank you very much in advance and please reach out if you have any questions.
> 
>   
> 
> With gratitude,
> 
> Erin Zamora

YEF 2024 Diversity Table: https://docs.google.com/document/d/1j_NTZtoUVJAEykwgJvsSBLNdPNc1jYPajXvScOfztEQ/edit?tab=t.0

YEF 2025 Application with Narrative: https://docs.google.com/document/d/1G3qsdUzUAq3nCq9QCeEaYh2bGilkGPRxugXE36837Gw/edit?tab=t.0#heading=h.4j36sw7swjrt

YEF Diversity Table: https://docs.google.com/spreadsheets/d/1-_KKROw2fRfQl93w3h48KviC6K3N4YKLR7Rdd7Ohxns/edit?gid=1663386223#gid=1663386223

### Demographic Diversity Table
[Staff Demographic Form Responses](https://docs.google.com/spreadsheets/d/1rU69LZuIE63d1zIaTqouxBrhqz51c9mvUhYDyOP8-5Q/edit?gid=1315867243#gid=1315867243)

Filter existing form responses in new tab
```
=ARRAYFORMULA(FILTER('Form Responses 1'!1:159,'Form Responses 1'!D:D = "Active",'Form Responses 1'!J:J <> "Canvasser"))
```
 
##### Use last year's YEF Edit Me! tab
Create a namehelper column with first and last name as a search key
```
=CONCAT(E2,F2)
```

Vlookup using the name helper to autofill previously edited categories, manually update the rest
```
=VLOOKUP(A2,'YEF Edit Me!'!D2:K,5,False)
```

Pull in dob and race (in YEF categories like this, then manually edit blanks)

[DOB comes from this sheet](https://docs.google.com/spreadsheets/d/1HaktJ7y0eehGYeWU9Cf1nbunPPC6KZYP0mJycZYUcuU/edit?gid=0#gid=0) which is pulled from Paychex

Pull age from DOB
```
=DATEDIF(C2, TODAY(), "Y")
```

Create age buckets to match YEF chart
```
=IF(B2<18,"Under 18",IF(B2<=21,"18 - 21",IF(B2<=25,"22 - 25",IF(B2<=29,"26 - 29",IF(B2<=35,"30 - 35",IF(B2<=44,"36 - 44","45+"))))))
```

Have to get Antonio, Lindsay, Bryce, Tara Clayton, and Anne to fill our [Staff Demo Form](https://docs.google.com/forms/d/e/1FAIpQLSeB2u5ZXdfoNzotx2r3iR3dXp5lxL1N0pqyXIK8pU3XbQO4yA/viewform?usp=sf_link). For now I've added them in data not captured.