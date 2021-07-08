# Calculator on multiple filters

The `Calculator` filter is applied on a specific filter and therefore does not allow to use a variable from another filter in the pipeline. 

In order to compute a quantity with inputs from several branches, one can use the `Python Calculator` filter. 

Select the filters on which you want to apply the calculator with `Ctrl+click` and add the `Python Calculator` filter. 

The Python expression can access the quantities of the selected filters through the `inputs` list. 
For example `inputs[0]` refers to 1st selected filter and so on. 
It is also possible to be more specific by giving the data type (`PointData`, `CellData`) and the variable name as follows: `inputs[0].CellData["varName"]`. 

**Reference:** https://discourse.paraview.org/t/get-results-from-filter-in-another-branch-of-the-pipeline/4755/3