### New Concept Learn :

**- option("overwriteSchema", "true")**

This option is used when our new dataframe contain different columns or schema than existing table have
this option simple overwrite our existing schema with new dataframe shcema , if we have not used this then we faces issue like column mismatched in table.

eg of error:
A metadata mismatch was detected when writing to the Delta table. SQLSTATE: 42KDG

eg : df_monthly_sales.write.mode("overwrite").option("overwriteSchema", "true").saveAsTable("associate_assignment.default.dim_monthly_sales")



