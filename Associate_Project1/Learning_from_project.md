**scenario: suppose we have below type of data and we need to merge this data in main table based on latest updated datetime , how we will do this?**

customer_id	name	 city	    status	  operation	  updated_datetime
1003	Mike Brown	 Dallas	    INACTIVE	D	      7/15/2026 10:02
1014	Tom Walker	 Houston	Actve	    U	      7/15/2026 10:13
1016	Henry Adams	 Denver	    INACTIVE	D	      7/15/2026 10:15
1021	John Smith	 New York	ACTIVE	    I	      7/15/2026 10:26
1018	Sophia Allen Miami	    active	    U	      7/15/2026 10:17
1020	Olivia Young Chicago	inactive	U	      7/15/2026 10:19
1021	John Smith	 New York	ACTIVE	    I	      7/15/2026 10:20
1021	John Smith	 New York	ACTIVE	    I	      7/15/2026 10:21

ANS:
first we will store above data in some intermediate table , then using row_number() we will store only latest record in temp view like

CREATE OR REPLACE TEMP VIEW latest_cdc AS
SELECT *
FROM (
    SELECT *,
           ROW_NUMBER() OVER (
               PARTITION BY customer_id
               ORDER BY CAST(updated_datetime AS TIMESTAMP) DESC
           ) AS rn
    FROM associate_assignment.default.dim_customer_cdc
) sub
WHERE rn = 1;

temp_view:
customer_id	name	 city	    status	  operation	  updated_datetime rn
1003	Mike Brown	 Dallas	    INACTIVE	D	      7/15/2026 10:15  1
1003	Henry Adams	 Denver	    INACTIVE	D	      7/15/2026 10:02  2
1021	John Smith	 New York	ACTIVE	    I	      7/15/2026 10:26  1
1021	John Smith	 New York	ACTIVE	    I	      7/15/2026 10:20  2
1021	John Smith	 New York	ACTIVE	    I	      7/15/2026 10:01  3
1034	Sophia Allen Miami	    active	    U	      7/15/2026 10:39  1
1034	Olivia Young Chicago	inactive	U	      7/15/2026 10:19  2
1034	Tom Walker	 Houston	Actve	    U	      7/15/2026 10:13  3

