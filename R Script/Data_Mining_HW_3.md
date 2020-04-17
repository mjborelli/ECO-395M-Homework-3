Predictive Model Building
=========================

Introduction
------------

There has been an increased focus on creating environmentally conscious
buildings, also known as green buildings, which adds a wrinkle into the
already complex decision-making process for commercial real estate firms
on what types of buildings they should construct. There are a variety of
potential benefits that could make investing in constructing green
buildings a worthwhile decision:

-   Lower operational costs (water, climate control, waste management,
    etc.).
-   Better indoor environments (natural sunlight for instance) could
    encourage better productivity and lead to happier and more motivated
    employees, increasing the incentives for a business to rent out
    those spaces.
-   Increased PR for both the real estate firm and the business renting
    the space due to positive public perceptions of green buildings.
-   Green buildings potentially have longer lives of operation. They are
    both physically constructed to last longer and less susceptible to
    energy market shocks.

While this list of benefits seems to make an airtight case for
commercial real estate firms to fully switch to constructing green
buildings, there is a major issue. Green buildings are generally more
costly for these firms to construct as a result of the standards that
must be met in order to achieve green building certification from LEED
or EnergyStar. However, as described above, it is possibly true that
business would be willing to pay hire rents for office space in green
buildings. This “Green Premium” would increase the profit incentives for
commercial real estate firms to construct green buildings, which could
be considered a societal good. However, we can’t be certain that the
“Green Premium” exists, or if so to what degree having a green certified
building would increase rent prices. This analysis has two main goals:

1.  Create the best predictive model possible for rent prices.
2.  Use said model to quantify the average change in rental income per
    sq. ft. associated with attaining green certification, holding any
    other features of the building constant.

Completing these two goals will give us a tool that we can employ to
predict rent prices for buildings given certain features, as well as
demonstrate to commercial real estate firms whether attempting to focus
future constructions projects on green buildings is financially worth
it. Given the uncertainty of the potential positive features listed
above, this is a good quantifiable method for providing evidence for or
against the impact of green certification on these companies’ decision
making processes.

Data and Model
--------------

For this analysis, we are using using a data set of 7894 commercial
rental properties in the U.S. Of these, 695 are certified as green
buildings through either LEED or EnergyStar. In this data set, each
green building is matched with a cluster of nearby non-green commercial
buildings. Data is collected for a variety of features about the
buildings, including rent in dollars per square foot, total square
footage, age of building, etc. We will use these features in order to
best predict building prices and quantify how much of a “Green Premium”
exists. To start, we want to first look at a summary of the differences
between different green classifications to give us a reference point for
future results.

    ##   LEED Energystar    N     Rent Leasing Rate      Age Class A % Class B %
    ## 1    0          0 7209 28.26678     81.97206 49.46733     36.22     48.48
    ## 2    0          1  631 30.04304     89.39591 23.20761     80.51     18.54
    ## 3    1          0   47 29.21043     87.42787 31.63830     68.09     29.79
    ## 4    1          1    7 32.99000     91.45286 29.00000     85.71     14.29

    ##   LEED Energystar Temp Control Days % w/ Amenities Precipitation
    ## 1    0          0          4703.402          50.76      31.26063
    ## 2    0          1          4110.742          74.64      28.46602
    ## 3    1          0          5488.660          44.68      38.54489
    ## 4    1          1          5484.857          85.71      32.04571

    ##   LEED Energystar  Gas Costs Electricity Costs
    ## 1    0          0 0.01135852        0.03089946
    ## 2    0          1 0.01097718        0.03191236
    ## 3    1          0 0.01253191        0.02771702
    ## 4    1          1 0.01204286        0.02772857

There are a lot of observations to make from these summary tables. A
simple glance at the rent column shows that rent appears to be higher in
green certified buildings than in non-green certified buildings.
However, there also seems to be a big difference in the rental rate
between EnergyStar certified buildings and LEED certified buildings,
0.8326173$ per square foot. Looking closer at these tables, there are
large differences in feature variables amongst the different populations
of building types. The starkest difference is in days of temperature
control, defined as the number of days where the building needs heating
or cooling. EnergyStar buildings need 592.6603176 days fewer of heating
or cooling than buildings with no green certification, but LEED
buildings actually need 785.257577 days more.

These differences necessitate more formal evaluations of the data.
Particularly, we will evaluate the effect of certification separately
for LEED and EnergyStar, as buildings that meet their respective
certifications seem to have different standards. To predict building
rent prices we will utilize random forest regression First, we split the
data into training and testing data, then use random forests to pick the
model with the lowest error. For robustness, we will test the error for
different numbers of trees in the random forest. However, we can’t use
random forests to quantify the effect of green certification on rent
prices, as large tree and forests are generally not interpretable in the
way we want. Therefore, we will utilizeTo check robustness, we will
compare the simple model of rent regressed onto green certification with
the model identified through our selection process on out-of-sample
performance.

Results
-------

### Random Forest Model Errors

The model errors are high at low amounts of trees, flattening out at
around 100 trees, so we will use 100 trees in our random forest model.

### Variable Importance Plot

Random Forests, as well as other large tree regression methods, are
generally not interpretable. However, we can create measures of variable
importance to help understand which feature variables by looking at
which variables best improve error within the aggregated trees. Below is
a variable importance plot, ranked in order of importance.

    varImpPlot(forest_green)

![](Data_Mining_HW_3_files/figure-markdown_strict/importance-1.png)

The most important variables are closely related since electricity
costs, total days of heating or cooling required, precipitation amounts,
and gas costs are all related to the running costs of a building.
Interestingly, the least important variables are the green certification
dummy variables, with EnergyStar being slightly more important than
LEED. This is evidence that having certification as a green building
doesn’t have a large impact on rental rates, but to be sure we will now
check using stepwise regression model selection.

### Green Premium

To start the stepwise selection process, we introduce a null model of
rent regressed onto the green building certification variables.

    simple_green = glm(Rent ~ LEED + Energystar + Energystar*LEED, data=green)
    stargazer(simple_green, type="html")

    ## 
    ## <table style="text-align:center"><tr><td colspan="2" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td><em>Dependent variable:</em></td></tr>
    ## <tr><td></td><td colspan="1" style="border-bottom: 1px solid black"></td></tr>
    ## <tr><td style="text-align:left"></td><td>Rent</td></tr>
    ## <tr><td colspan="2" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">LEED</td><td>0.944</td></tr>
    ## <tr><td style="text-align:left"></td><td>(2.205)</td></tr>
    ## <tr><td style="text-align:left"></td><td></td></tr>
    ## <tr><td style="text-align:left">Energystar</td><td>1.776<sup>***</sup></td></tr>
    ## <tr><td style="text-align:left"></td><td>(0.626)</td></tr>
    ## <tr><td style="text-align:left"></td><td></td></tr>
    ## <tr><td style="text-align:left">LEED:Energystar</td><td>2.003</td></tr>
    ## <tr><td style="text-align:left"></td><td>(6.137)</td></tr>
    ## <tr><td style="text-align:left"></td><td></td></tr>
    ## <tr><td style="text-align:left">Constant</td><td>28.267<sup>***</sup></td></tr>
    ## <tr><td style="text-align:left"></td><td>(0.177)</td></tr>
    ## <tr><td style="text-align:left"></td><td></td></tr>
    ## <tr><td colspan="2" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>7,894</td></tr>
    ## <tr><td style="text-align:left">Log Likelihood</td><td>-32,614.150</td></tr>
    ## <tr><td style="text-align:left">Akaike Inf. Crit.</td><td>65,236.310</td></tr>
    ## <tr><td colspan="2" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
    ## </table>

Conclusion
----------

In the future, it would be prudent to access data that distinctifies the
specific rating levels for LEED and EnergyStar green certification.

What Causes What?
=================

1.
--

2.
--

3.
--

4.
--

Clustering and PCA
==================

Introduction
------------

Data and Model
--------------

Results
-------

Conclusion
----------

Market Segmentation
===================

Introduction
------------

Data and Model
--------------

Results
-------

Conclusion
----------
