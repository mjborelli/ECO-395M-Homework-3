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
between to give us a reference point for future results.

    ##   LEED Energystar    N     Rent Leasing Rate      Age Class A Class B
    ## 1    0          0 7209 28.26678     81.97206 49.46733   36.22   48.48
    ## 2    0          1  631 30.04304     89.39591 23.20761   80.51   18.54
    ## 3    1          0   47 29.21043     87.42787 31.63830   68.09   29.79
    ## 4    1          1    7 32.99000     91.45286 29.00000   85.71   14.29
    ##   Temp Control Days
    ## 1          4703.402
    ## 2          4110.742
    ## 3          5488.660
    ## 4          5484.857

For modelling purposes, we will be utilizing bagged random forests.

Results
-------

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