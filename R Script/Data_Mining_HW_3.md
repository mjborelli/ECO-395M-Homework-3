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
    stargazer(simple_green, type="text")

    ## 
    ## =============================================
    ##                       Dependent variable:    
    ##                   ---------------------------
    ##                              Rent            
    ## ---------------------------------------------
    ## LEED                         0.944           
    ##                             (2.205)          
    ##                                              
    ## Energystar                 1.776***          
    ##                             (0.626)          
    ##                                              
    ## LEED:Energystar              2.003           
    ##                             (6.137)          
    ##                                              
    ## Constant                   28.267***         
    ##                             (0.177)          
    ##                                              
    ## ---------------------------------------------
    ## Observations                 7,894           
    ## Log Likelihood            -32,614.150        
    ## Akaike Inf. Crit.         65,236.310         
    ## =============================================
    ## Note:             *p<0.1; **p<0.05; ***p<0.01

In this regression, we get positive coefficients for each of the green
certification variables, but only EnergyStar certification’s effect is
statistically significant. LEED does not have any significantly positive
effect on rent in our simple model. To compare, the following is the
model picked by stepwise selection using AIC as the loss function.

    ## Start:  AIC=65236.31
    ## Rent ~ LEED + Energystar + Energystar * LEED
    ## 
    ##                     Df Deviance   AIC
    ## + Electricity_Costs  1  1517242 63925
    ## + total_dd_07        1  1681693 64738
    ## + class_a            1  1713009 64883
    ## + leasing_rate       1  1735826 64988
    ## + size               1  1759114 65093
    ## + class_b            1  1766026 65124
    ## + renovated          1  1766322 65125
    ## + stories            1  1768202 65133
    ## + age                1  1774679 65162
    ## + Precipitation      1  1783412 65201
    ## + amenities          1  1785996 65213
    ## + net                1  1786983 65217
    ## - LEED:Energystar    1  1791861 65234
    ## <none>                  1791837 65236
    ## + Gas_Costs          1  1791831 65238
    ## 
    ## Step:  AIC=63925.17
    ## Rent ~ LEED + Energystar + Electricity_Costs + LEED:Energystar
    ## 
    ##                                Df Deviance   AIC
    ## + class_a                       1  1455979 63602
    ## + size                          1  1460734 63628
    ## + leasing_rate                  1  1471635 63686
    ## + stories                       1  1475660 63708
    ## + Precipitation                 1  1484503 63755
    ## + amenities                     1  1493295 63802
    ## + class_b                       1  1500455 63839
    ## + Gas_Costs                     1  1500603 63840
    ## + renovated                     1  1506367 63870
    ## + age                           1  1506717 63872
    ## + net                           1  1509121 63885
    ## - LEED:Energystar               1  1517285 63923
    ## <none>                             1517242 63925
    ## + LEED:Electricity_Costs        1  1516957 63926
    ## + total_dd_07                   1  1516980 63926
    ## + Energystar:Electricity_Costs  1  1517228 63927
    ## - Electricity_Costs             1  1791837 65236
    ## 
    ## Step:  AIC=63601.81
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + LEED:Energystar
    ## 
    ##                                Df Deviance   AIC
    ## + leasing_rate                  1  1427087 63446
    ## + Precipitation                 1  1430811 63466
    ## + Gas_Costs                     1  1432694 63477
    ## + size                          1  1435916 63494
    ## + net                           1  1442312 63529
    ## + stories                       1  1446016 63550
    ## + class_b                       1  1449314 63568
    ## + renovated                     1  1451214 63578
    ## + amenities                     1  1452343 63584
    ## + class_a:Electricity_Costs     1  1454149 63594
    ## - LEED:Energystar               1  1456087 63600
    ## <none>                             1455979 63602
    ## + age                           1  1455704 63602
    ## + total_dd_07                   1  1455793 63603
    ## + LEED:Electricity_Costs        1  1455818 63603
    ## + Energystar:Electricity_Costs  1  1455829 63603
    ## + LEED:class_a                  1  1455956 63604
    ## + Energystar:class_a            1  1455975 63604
    ## - class_a                       1  1517242 63925
    ## - Electricity_Costs             1  1713009 64883
    ## 
    ## Step:  AIC=63445.59
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     LEED:Energystar
    ## 
    ##                                  Df Deviance   AIC
    ## + Precipitation                   1  1402989 63313
    ## + Gas_Costs                       1  1403940 63318
    ## + size                            1  1412307 63365
    ## + net                             1  1413233 63371
    ## + stories                         1  1421204 63415
    ## + renovated                       1  1421373 63416
    ## + leasing_rate:Electricity_Costs  1  1422666 63423
    ## + class_b                         1  1423963 63430
    ## + leasing_rate:class_a            1  1424521 63433
    ## + class_a:Electricity_Costs       1  1425012 63436
    ## + amenities                       1  1425619 63439
    ## - LEED:Energystar                 1  1427200 63444
    ## + age                             1  1426714 63446
    ## <none>                               1427087 63446
    ## + Energystar:Electricity_Costs    1  1426915 63447
    ## + LEED:Electricity_Costs          1  1426945 63447
    ## + total_dd_07                     1  1427001 63447
    ## + LEED:class_a                    1  1427022 63447
    ## + Energystar:leasing_rate         1  1427073 63448
    ## + Energystar:class_a              1  1427084 63448
    ## + LEED:leasing_rate               1  1427087 63448
    ## - leasing_rate                    1  1455979 63602
    ## - class_a                         1  1471635 63686
    ## - Electricity_Costs               1  1678205 64723
    ## 
    ## Step:  AIC=63313.15
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + LEED:Energystar
    ## 
    ##                                   Df Deviance   AIC
    ## + Gas_Costs                        1  1293559 62674
    ## + net                              1  1387833 63229
    ## + size                             1  1392264 63255
    ## + renovated                        1  1395922 63275
    ## + Precipitation:Electricity_Costs  1  1396847 63281
    ## + leasing_rate:Precipitation       1  1397717 63285
    ## + class_b                          1  1398695 63291
    ## + leasing_rate:Electricity_Costs   1  1399399 63295
    ## + total_dd_07                      1  1399546 63296
    ## + stories                          1  1400195 63299
    ## + Energystar:Precipitation         1  1400746 63303
    ## + leasing_rate:class_a             1  1400793 63303
    ## + class_a:Electricity_Costs        1  1401534 63307
    ## + amenities                        1  1401694 63308
    ## - LEED:Energystar                  1  1403131 63312
    ## <none>                                1402989 63313
    ## + class_a:Precipitation            1  1402685 63313
    ## + Energystar:Electricity_Costs     1  1402741 63314
    ## + LEED:Electricity_Costs           1  1402798 63314
    ## + LEED:class_a                     1  1402882 63315
    ## + LEED:Precipitation               1  1402905 63315
    ## + Energystar:leasing_rate          1  1402968 63315
    ## + LEED:leasing_rate                1  1402989 63315
    ## + age                              1  1402989 63315
    ## + Energystar:class_a               1  1402989 63315
    ## - Precipitation                    1  1427087 63446
    ## - leasing_rate                     1  1430811 63466
    ## - class_a                          1  1441672 63526
    ## - Electricity_Costs                1  1673758 64704
    ## 
    ## Step:  AIC=62674.1
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + LEED:Energystar
    ## 
    ##                                   Df Deviance   AIC
    ## + Gas_Costs:Electricity_Costs      1  1248137 62394
    ## + Precipitation:Gas_Costs          1  1259029 62463
    ## + renovated                        1  1283773 62616
    ## + Precipitation:Electricity_Costs  1  1284761 62622
    ## + net                              1  1285322 62626
    ## + class_b                          1  1288354 62644
    ## + size                             1  1288800 62647
    ## + leasing_rate:Precipitation       1  1288978 62648
    ## + class_a:Electricity_Costs        1  1289538 62652
    ## + leasing_rate:class_a             1  1289971 62654
    ## + Energystar:Precipitation         1  1290729 62659
    ## + leasing_rate:Electricity_Costs   1  1291015 62661
    ## + class_a:Gas_Costs                1  1291337 62663
    ## + total_dd_07                      1  1291798 62665
    ## + age                              1  1292033 62667
    ## + stories                          1  1292756 62671
    ## - LEED:Energystar                  1  1293773 62673
    ## <none>                                1293559 62674
    ## + amenities                        1  1293330 62675
    ## + LEED:Electricity_Costs           1  1293435 62675
    ## + LEED:Gas_Costs                   1  1293438 62675
    ## + LEED:Precipitation               1  1293481 62676
    ## + LEED:class_a                     1  1293486 62676
    ## + Energystar:leasing_rate          1  1293496 62676
    ## + LEED:leasing_rate                1  1293503 62676
    ## + Energystar:Gas_Costs             1  1293518 62676
    ## + leasing_rate:Gas_Costs           1  1293520 62676
    ## + Energystar:Electricity_Costs     1  1293531 62676
    ## + class_a:Precipitation            1  1293554 62676
    ## + Energystar:class_a               1  1293559 62676
    ## - leasing_rate                     1  1319142 62827
    ## - class_a                          1  1338359 62941
    ## - Gas_Costs                        1  1402989 63313
    ## - Precipitation                    1  1403940 63318
    ## - Electricity_Costs                1  1667052 64674
    ## 
    ## Step:  AIC=62393.92
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + LEED:Energystar + Electricity_Costs:Gas_Costs
    ## 
    ##                                   Df Deviance   AIC
    ## + Precipitation:Electricity_Costs  1  1214049 62177
    ## + total_dd_07                      1  1230064 62281
    ## + size                             1  1236375 62321
    ## + class_a:Electricity_Costs        1  1237955 62331
    ## + renovated                        1  1240206 62346
    ## + stories                          1  1241601 62354
    ## + class_b                          1  1242800 62362
    ## + Precipitation:Gas_Costs          1  1243120 62364
    ## + net                              1  1243388 62366
    ## + leasing_rate:class_a             1  1243935 62369
    ## + leasing_rate:Precipitation       1  1244001 62370
    ## + Energystar:Precipitation         1  1245311 62378
    ## + leasing_rate:Electricity_Costs   1  1245465 62379
    ## + class_a:Gas_Costs                1  1247643 62393
    ## + age                              1  1247649 62393
    ## - LEED:Energystar                  1  1248283 62393
    ## + amenities                        1  1247664 62393
    ## <none>                                1248137 62394
    ## + Energystar:Gas_Costs             1  1247864 62394
    ## + LEED:Precipitation               1  1248011 62395
    ## + LEED:Gas_Costs                   1  1248016 62395
    ## + Energystar:leasing_rate          1  1248041 62395
    ## + Energystar:Electricity_Costs     1  1248084 62396
    ## + LEED:leasing_rate                1  1248101 62396
    ## + leasing_rate:Gas_Costs           1  1248102 62396
    ## + LEED:Electricity_Costs           1  1248116 62396
    ## + LEED:class_a                     1  1248128 62396
    ## + Energystar:class_a               1  1248134 62396
    ## + class_a:Precipitation            1  1248136 62396
    ## - leasing_rate                     1  1273369 62550
    ## - Precipitation                    1  1277830 62578
    ## - class_a                          1  1286873 62633
    ## - Electricity_Costs:Gas_Costs      1  1293559 62674
    ## 
    ## Step:  AIC=62177.33
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation
    ## 
    ##                                   Df Deviance   AIC
    ## + Precipitation:Gas_Costs          1  1203687 62112
    ## + size                             1  1205079 62121
    ## + class_a:Electricity_Costs        1  1206696 62131
    ## + net                              1  1208842 62145
    ## + renovated                        1  1209343 62149
    ## + stories                          1  1209600 62150
    ## + class_b                          1  1210379 62155
    ## + leasing_rate:class_a             1  1210513 62156
    ## + Energystar:Precipitation         1  1211094 62160
    ## + leasing_rate:Precipitation       1  1211286 62161
    ## + leasing_rate:Electricity_Costs   1  1212457 62169
    ## + total_dd_07                      1  1213220 62174
    ## + age                              1  1213330 62175
    ## + class_a:Gas_Costs                1  1213520 62176
    ## - LEED:Energystar                  1  1214169 62176
    ## + Energystar:Gas_Costs             1  1213637 62177
    ## <none>                                1214049 62177
    ## + class_a:Precipitation            1  1213762 62177
    ## + amenities                        1  1213824 62178
    ## + LEED:Gas_Costs                   1  1213954 62179
    ## + Energystar:leasing_rate          1  1213958 62179
    ## + LEED:Precipitation               1  1213961 62179
    ## + leasing_rate:Gas_Costs           1  1213989 62179
    ## + LEED:leasing_rate                1  1213989 62179
    ## + LEED:Electricity_Costs           1  1213994 62179
    ## + LEED:class_a                     1  1214034 62179
    ## + Energystar:class_a               1  1214036 62179
    ## + Energystar:Electricity_Costs     1  1214048 62179
    ## - leasing_rate                     1  1237512 62326
    ## - Electricity_Costs:Precipitation  1  1248137 62394
    ## - class_a                          1  1251699 62416
    ## - Electricity_Costs:Gas_Costs      1  1284761 62622
    ## 
    ## Step:  AIC=62111.67
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs
    ## 
    ##                                   Df Deviance   AIC
    ## + class_a:Electricity_Costs        1  1196463 62066
    ## + size                             1  1196512 62066
    ## + net                              1  1198419 62079
    ## + renovated                        1  1198773 62081
    ## + class_b                          1  1199063 62083
    ## + leasing_rate:class_a             1  1200226 62091
    ## + stories                          1  1200480 62093
    ## + total_dd_07                      1  1201034 62096
    ## + Energystar:Precipitation         1  1201109 62097
    ## + leasing_rate:Precipitation       1  1201151 62097
    ## + leasing_rate:Electricity_Costs   1  1202545 62106
    ## + age                              1  1202586 62106
    ## - LEED:Energystar                  1  1203827 62111
    ## + Energystar:Gas_Costs             1  1203221 62111
    ## + class_a:Gas_Costs                1  1203318 62111
    ## <none>                                1203687 62112
    ## + amenities                        1  1203459 62112
    ## + Energystar:leasing_rate          1  1203612 62113
    ## + LEED:Gas_Costs                   1  1203614 62113
    ## + LEED:leasing_rate                1  1203637 62113
    ## + LEED:Electricity_Costs           1  1203637 62113
    ## + LEED:Precipitation               1  1203638 62113
    ## + leasing_rate:Gas_Costs           1  1203643 62113
    ## + class_a:Precipitation            1  1203643 62113
    ## + LEED:class_a                     1  1203649 62113
    ## + Energystar:class_a               1  1203668 62114
    ## + Energystar:Electricity_Costs     1  1203684 62114
    ## - Precipitation:Gas_Costs          1  1214049 62177
    ## - leasing_rate                     1  1228006 62268
    ## - Electricity_Costs:Gas_Costs      1  1231060 62287
    ## - class_a                          1  1240672 62349
    ## - Electricity_Costs:Precipitation  1  1243120 62364
    ## 
    ## Step:  AIC=62066.15
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a
    ## 
    ##                                   Df Deviance   AIC
    ## + size                             1  1188585 62016
    ## + renovated                        1  1191086 62033
    ## + net                              1  1191685 62037
    ## + class_b                          1  1192015 62039
    ## + stories                          1  1192684 62043
    ## + leasing_rate:class_a             1  1193154 62046
    ## + leasing_rate:Precipitation       1  1193847 62051
    ## + total_dd_07                      1  1194010 62052
    ## + Energystar:Precipitation         1  1194381 62054
    ## + age                              1  1194890 62058
    ## + class_a:Gas_Costs                1  1194907 62058
    ## - LEED:Energystar                  1  1196574 62065
    ## + Energystar:Gas_Costs             1  1196103 62066
    ## + leasing_rate:Gas_Costs           1  1196130 62066
    ## <none>                                1196463 62066
    ## + Energystar:Electricity_Costs     1  1196186 62066
    ## + leasing_rate:Electricity_Costs   1  1196297 62067
    ## + amenities                        1  1196337 62067
    ## + class_a:Precipitation            1  1196386 62068
    ## + LEED:class_a                     1  1196395 62068
    ## + LEED:leasing_rate                1  1196399 62068
    ## + LEED:Electricity_Costs           1  1196403 62068
    ## + Energystar:leasing_rate          1  1196418 62068
    ## + Energystar:class_a               1  1196418 62068
    ## + LEED:Precipitation               1  1196429 62068
    ## + LEED:Gas_Costs                   1  1196441 62068
    ## - Electricity_Costs:class_a        1  1203687 62112
    ## - Precipitation:Gas_Costs          1  1206696 62131
    ## - leasing_rate                     1  1221240 62226
    ## - Electricity_Costs:Gas_Costs      1  1226697 62261
    ## - Electricity_Costs:Precipitation  1  1232850 62301
    ## 
    ## Step:  AIC=62016
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a
    ## 
    ##                                   Df Deviance   AIC
    ## + size:Electricity_Costs           1  1141696 61700
    ## + size:leasing_rate                1  1179479 61957
    ## + renovated                        1  1182385 61977
    ## + net                              1  1182869 61980
    ## + size:Precipitation               1  1182964 61981
    ## + total_dd_07                      1  1185237 61996
    ## + class_b                          1  1185239 61996
    ## + leasing_rate:class_a             1  1185944 62000
    ## + leasing_rate:Precipitation       1  1186443 62004
    ## + Energystar:Precipitation         1  1186562 62005
    ## + age                              1  1187177 62009
    ## + class_a:Gas_Costs                1  1187183 62009
    ## + size:class_a                     1  1187703 62012
    ## - LEED:Energystar                  1  1188678 62015
    ## + Energystar:Gas_Costs             1  1188231 62016
    ## + leasing_rate:Gas_Costs           1  1188233 62016
    ## <none>                                1188585 62016
    ## + leasing_rate:Electricity_Costs   1  1188383 62017
    ## + Energystar:Electricity_Costs     1  1188404 62017
    ## + size:Gas_Costs                   1  1188429 62017
    ## + Energystar:size                  1  1188503 62017
    ## + Energystar:leasing_rate          1  1188506 62017
    ## + amenities                        1  1188506 62017
    ## + stories                          1  1188508 62017
    ## + LEED:Electricity_Costs           1  1188517 62018
    ## + LEED:leasing_rate                1  1188535 62018
    ## + LEED:Precipitation               1  1188539 62018
    ## + LEED:class_a                     1  1188540 62018
    ## + class_a:Precipitation            1  1188551 62018
    ## + LEED:Gas_Costs                   1  1188569 62018
    ## + Energystar:class_a               1  1188569 62018
    ## + LEED:size                        1  1188583 62018
    ## - size                             1  1196463 62066
    ## - Electricity_Costs:class_a        1  1196512 62066
    ## - Precipitation:Gas_Costs          1  1196952 62069
    ## - leasing_rate                     1  1209661 62153
    ## - Electricity_Costs:Precipitation  1  1221499 62230
    ## - Electricity_Costs:Gas_Costs      1  1223473 62242
    ## 
    ## Step:  AIC=61700.28
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size
    ## 
    ##                                   Df Deviance   AIC
    ## + size:Precipitation               1  1133546 61646
    ## + renovated                        1  1135028 61656
    ## + size:leasing_rate                1  1136600 61667
    ## + net                              1  1137066 61670
    ## + class_b                          1  1137898 61676
    ## + age                              1  1139298 61686
    ## + leasing_rate:Precipitation       1  1139463 61687
    ## + Energystar:Precipitation         1  1140014 61691
    ## + leasing_rate:class_a             1  1140073 61691
    ## + size:class_a                     1  1140311 61693
    ## + total_dd_07                      1  1140565 61694
    ## + leasing_rate:Gas_Costs           1  1141112 61698
    ## + class_a:Gas_Costs                1  1141191 61699
    ## - LEED:Energystar                  1  1141845 61699
    ## + Energystar:Gas_Costs             1  1141270 61699
    ## + stories                          1  1141290 61699
    ## <none>                                1141696 61700
    ## + LEED:Electricity_Costs           1  1141488 61701
    ## + leasing_rate:Electricity_Costs   1  1141548 61701
    ## + LEED:leasing_rate                1  1141555 61701
    ## - Electricity_Costs:class_a        1  1142152 61701
    ## + Energystar:Electricity_Costs     1  1141588 61702
    ## + Energystar:size                  1  1141588 61702
    ## + Energystar:leasing_rate          1  1141653 61702
    ## + LEED:Precipitation               1  1141666 61702
    ## + size:Gas_Costs                   1  1141678 61702
    ## + LEED:class_a                     1  1141678 61702
    ## + LEED:Gas_Costs                   1  1141688 61702
    ## + amenities                        1  1141692 61702
    ## + class_a:Precipitation            1  1141696 61702
    ## + LEED:size                        1  1141696 61702
    ## + Energystar:class_a               1  1141696 61702
    ## - Precipitation:Gas_Costs          1  1148609 61746
    ## - Electricity_Costs:Precipitation  1  1160181 61825
    ## - leasing_rate                     1  1162190 61839
    ## - Electricity_Costs:Gas_Costs      1  1166992 61871
    ## - Electricity_Costs:size           1  1188585 62016
    ## 
    ## Step:  AIC=61645.72
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + Precipitation:size
    ## 
    ##                                   Df Deviance   AIC
    ## + size:Gas_Costs                   1  1127056 61602
    ## + renovated                        1  1127894 61608
    ## + size:leasing_rate                1  1129243 61618
    ## + net                              1  1129460 61619
    ## + class_b                          1  1129983 61623
    ## + Energystar:Precipitation         1  1130941 61630
    ## + class_a:Precipitation            1  1131005 61630
    ## + class_a:Gas_Costs                1  1131138 61631
    ## + age                              1  1131561 61634
    ## + leasing_rate:class_a             1  1131931 61636
    ## + size:class_a                     1  1132043 61637
    ## + total_dd_07                      1  1132303 61639
    ## + leasing_rate:Gas_Costs           1  1132363 61639
    ## + leasing_rate:Precipitation       1  1132705 61642
    ## + Energystar:Gas_Costs             1  1132788 61642
    ## - LEED:Energystar                  1  1133742 61645
    ## <none>                                1133546 61646
    ## - Electricity_Costs:class_a        1  1133927 61646
    ## + LEED:Electricity_Costs           1  1133372 61647
    ## + leasing_rate:Electricity_Costs   1  1133381 61647
    ## + LEED:leasing_rate                1  1133441 61647
    ## + Energystar:Electricity_Costs     1  1133451 61647
    ## + LEED:Precipitation               1  1133479 61647
    ## + Energystar:leasing_rate          1  1133499 61647
    ## + LEED:size                        1  1133503 61647
    ## + Energystar:class_a               1  1133534 61648
    ## + amenities                        1  1133535 61648
    ## + LEED:Gas_Costs                   1  1133537 61648
    ## + Energystar:size                  1  1133540 61648
    ## + stories                          1  1133544 61648
    ## + LEED:class_a                     1  1133545 61648
    ## - Precipitation:Gas_Costs          1  1139703 61686
    ## - Precipitation:size               1  1141696 61700
    ## - Electricity_Costs:Precipitation  1  1150506 61761
    ## - leasing_rate                     1  1153776 61783
    ## - Electricity_Costs:Gas_Costs      1  1158887 61818
    ## - Electricity_Costs:size           1  1182964 61981
    ## 
    ## Step:  AIC=61602.4
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + Precipitation:size + 
    ##     Gas_Costs:size
    ## 
    ##                                   Df Deviance   AIC
    ## + renovated                        1  1121522 61566
    ## + class_b                          1  1123213 61577
    ## + size:leasing_rate                1  1123312 61578
    ## + net                              1  1123454 61579
    ## + Energystar:Precipitation         1  1124763 61588
    ## + age                              1  1125043 61590
    ## + leasing_rate:class_a             1  1125333 61592
    ## + class_a:Precipitation            1  1125547 61594
    ## + leasing_rate:Precipitation       1  1125561 61594
    ## + size:class_a                     1  1125868 61596
    ## + total_dd_07                      1  1125877 61596
    ## - Electricity_Costs:class_a        1  1127264 61602
    ## + LEED:Electricity_Costs           1  1126711 61602
    ## - LEED:Energystar                  1  1127284 61602
    ## + Energystar:Gas_Costs             1  1126718 61602
    ## <none>                                1127056 61602
    ## + class_a:Gas_Costs                1  1126897 61603
    ## + Energystar:Electricity_Costs     1  1126928 61603
    ## + LEED:leasing_rate                1  1126935 61604
    ## + stories                          1  1126961 61604
    ## + LEED:Precipitation               1  1126997 61604
    ## + Energystar:size                  1  1127005 61604
    ## + LEED:Gas_Costs                   1  1127006 61604
    ## + Energystar:leasing_rate          1  1127027 61604
    ## + amenities                        1  1127029 61604
    ## + LEED:size                        1  1127031 61604
    ## + leasing_rate:Gas_Costs           1  1127047 61604
    ## + Energystar:class_a               1  1127048 61604
    ## + LEED:class_a                     1  1127049 61604
    ## + leasing_rate:Electricity_Costs   1  1127053 61604
    ## - Precipitation:Gas_Costs          1  1131228 61630
    ## - Gas_Costs:size                   1  1133546 61646
    ## - Precipitation:size               1  1141678 61702
    ## - Electricity_Costs:Precipitation  1  1141844 61703
    ## - leasing_rate                     1  1148867 61752
    ## - Electricity_Costs:Gas_Costs      1  1156113 61801
    ## - Electricity_Costs:size           1  1180962 61969
    ## 
    ## Step:  AIC=61565.54
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + LEED:Energystar + 
    ##     Electricity_Costs:Gas_Costs + Electricity_Costs:Precipitation + 
    ##     Precipitation:Gas_Costs + Electricity_Costs:class_a + Electricity_Costs:size + 
    ##     Precipitation:size + Gas_Costs:size
    ## 
    ##                                   Df Deviance   AIC
    ## + class_b                          1  1117627 61540
    ## + net                              1  1117849 61542
    ## + size:leasing_rate                1  1118090 61543
    ## + Energystar:Precipitation         1  1119123 61551
    ## + leasing_rate:class_a             1  1119771 61555
    ## + class_a:Precipitation            1  1120002 61557
    ## + leasing_rate:Precipitation       1  1120049 61557
    ## + renovated:Gas_Costs              1  1120108 61558
    ## + size:class_a                     1  1120149 61558
    ## + renovated:class_a                1  1120439 61560
    ## + total_dd_07                      1  1120465 61560
    ## - Electricity_Costs:class_a        1  1121676 61565
    ## + Energystar:Gas_Costs             1  1121163 61565
    ## + LEED:Electricity_Costs           1  1121166 61565
    ## - LEED:Energystar                  1  1121764 61565
    ## <none>                                1121522 61566
    ## + renovated:Electricity_Costs      1  1121253 61566
    ## + stories                          1  1121282 61566
    ## + Energystar:renovated             1  1121321 61566
    ## + age                              1  1121332 61566
    ## + class_a:Gas_Costs                1  1121342 61566
    ## + LEED:leasing_rate                1  1121382 61567
    ## + leasing_rate:renovated           1  1121397 61567
    ## + size:renovated                   1  1121398 61567
    ## + Energystar:Electricity_Costs     1  1121438 61567
    ## + renovated:Precipitation          1  1121438 61567
    ## + amenities                        1  1121463 61567
    ## + LEED:Precipitation               1  1121464 61567
    ## + LEED:Gas_Costs                   1  1121476 61567
    ## + LEED:size                        1  1121483 61567
    ## + Energystar:leasing_rate          1  1121489 61567
    ## + Energystar:size                  1  1121490 61567
    ## + Energystar:class_a               1  1121499 61567
    ## + leasing_rate:Gas_Costs           1  1121512 61567
    ## + LEED:class_a                     1  1121521 61568
    ## + LEED:renovated                   1  1121521 61568
    ## + leasing_rate:Electricity_Costs   1  1121522 61568
    ## - Precipitation:Gas_Costs          1  1125819 61594
    ## - renovated                        1  1127056 61602
    ## - Gas_Costs:size                   1  1127894 61608
    ## - Electricity_Costs:Precipitation  1  1134181 61652
    ## - Precipitation:size               1  1134971 61658
    ## - leasing_rate                     1  1144054 61721
    ## - Electricity_Costs:Gas_Costs      1  1148824 61753
    ## - Electricity_Costs:size           1  1175656 61936
    ## 
    ## Step:  AIC=61540.08
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     LEED:Energystar + Electricity_Costs:Gas_Costs + Electricity_Costs:Precipitation + 
    ##     Precipitation:Gas_Costs + Electricity_Costs:class_a + Electricity_Costs:size + 
    ##     Precipitation:size + Gas_Costs:size
    ## 
    ##                                   Df Deviance   AIC
    ## + size:leasing_rate                1  1113074 61510
    ## + net                              1  1113930 61516
    ## + class_b:Electricity_Costs        1  1114869 61523
    ## + Energystar:Precipitation         1  1115242 61525
    ## + leasing_rate:class_a             1  1115347 61526
    ## + class_a:Precipitation            1  1115984 61530
    ## + renovated:Gas_Costs              1  1116240 61532
    ## + leasing_rate:Precipitation       1  1116427 61534
    ## + renovated:class_a                1  1116596 61535
    ## + total_dd_07                      1  1116732 61536
    ## + size:class_a                     1  1116902 61537
    ## + size:class_b                     1  1116974 61537
    ## - Electricity_Costs:class_a        1  1117821 61539
    ## + LEED:Electricity_Costs           1  1117276 61540
    ## + Energystar:Gas_Costs             1  1117279 61540
    ## + renovated:Electricity_Costs      1  1117289 61540
    ## + renovated:class_b                1  1117301 61540
    ## - LEED:Energystar                  1  1117886 61540
    ## <none>                                1117627 61540
    ## + Energystar:renovated             1  1117416 61541
    ## + stories                          1  1117446 61541
    ## + class_a:Gas_Costs                1  1117447 61541
    ## + leasing_rate:renovated           1  1117466 61541
    ## + class_b:Precipitation            1  1117472 61541
    ## + LEED:leasing_rate                1  1117484 61541
    ## + size:renovated                   1  1117490 61541
    ## + Energystar:class_b               1  1117503 61541
    ## + Energystar:Electricity_Costs     1  1117526 61541
    ## + Energystar:size                  1  1117551 61542
    ## + Energystar:class_a               1  1117554 61542
    ## + Energystar:leasing_rate          1  1117561 61542
    ## + renovated:Precipitation          1  1117564 61542
    ## + leasing_rate:Gas_Costs           1  1117568 61542
    ## + LEED:Precipitation               1  1117570 61542
    ## + LEED:Gas_Costs                   1  1117575 61542
    ## + LEED:class_b                     1  1117594 61542
    ## + LEED:size                        1  1117601 61542
    ## + LEED:class_a                     1  1117622 61542
    ## + leasing_rate:class_b             1  1117623 61542
    ## + class_b:Gas_Costs                1  1117624 61542
    ## + leasing_rate:Electricity_Costs   1  1117624 61542
    ## + amenities                        1  1117625 61542
    ## + age                              1  1117627 61542
    ## + LEED:renovated                   1  1117627 61542
    ## - class_b                          1  1121522 61566
    ## - Precipitation:Gas_Costs          1  1122548 61573
    ## - renovated                        1  1123213 61577
    ## - Gas_Costs:size                   1  1124278 61585
    ## - Electricity_Costs:Precipitation  1  1129577 61622
    ## - Precipitation:size               1  1131091 61633
    ## - leasing_rate                     1  1137221 61675
    ## - Electricity_Costs:Gas_Costs      1  1143328 61718
    ## - Electricity_Costs:size           1  1172324 61915
    ## 
    ## Step:  AIC=61509.86
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     LEED:Energystar + Electricity_Costs:Gas_Costs + Electricity_Costs:Precipitation + 
    ##     Precipitation:Gas_Costs + Electricity_Costs:class_a + Electricity_Costs:size + 
    ##     Precipitation:size + Gas_Costs:size + leasing_rate:size
    ## 
    ##                                   Df Deviance   AIC
    ## + net                              1  1109569 61487
    ## + class_b:Electricity_Costs        1  1110304 61492
    ## + Energystar:Precipitation         1  1110752 61495
    ## + renovated:Gas_Costs              1  1111544 61501
    ## + class_a:Precipitation            1  1111553 61501
    ## + size:class_a                     1  1111761 61503
    ## + size:class_b                     1  1111948 61504
    ## + renovated:class_a                1  1111967 61504
    ## + total_dd_07                      1  1112172 61505
    ## + leasing_rate:Precipitation       1  1112273 61506
    ## - Electricity_Costs:class_a        1  1113194 61509
    ## + renovated:class_b                1  1112716 61509
    ## + Energystar:Gas_Costs             1  1112746 61510
    ## + leasing_rate:class_a             1  1112757 61510
    ## + stories                          1  1112762 61510
    ## + renovated:Electricity_Costs      1  1112776 61510
    ## - LEED:Energystar                  1  1113347 61510
    ## <none>                                1113074 61510
    ## + LEED:Electricity_Costs           1  1112832 61510
    ## + leasing_rate:renovated           1  1112851 61510
    ## + Energystar:renovated             1  1112903 61511
    ## + class_b:Precipitation            1  1112909 61511
    ## + class_a:Gas_Costs                1  1112916 61511
    ## + renovated:Precipitation          1  1112936 61511
    ## + Energystar:class_b               1  1112946 61511
    ## + Energystar:Electricity_Costs     1  1112954 61511
    ## + Energystar:size                  1  1112966 61511
    ## + leasing_rate:Electricity_Costs   1  1112981 61511
    ## + Energystar:class_a               1  1112999 61511
    ## + LEED:Precipitation               1  1113005 61511
    ## + amenities                        1  1113015 61511
    ## + LEED:Gas_Costs                   1  1113016 61511
    ## + size:renovated                   1  1113021 61511
    ## + LEED:leasing_rate                1  1113028 61512
    ## + leasing_rate:class_b             1  1113028 61512
    ## + leasing_rate:Gas_Costs           1  1113042 61512
    ## + LEED:class_b                     1  1113047 61512
    ## + class_b:Gas_Costs                1  1113064 61512
    ## + LEED:size                        1  1113066 61512
    ## + LEED:class_a                     1  1113066 61512
    ## + Energystar:leasing_rate          1  1113069 61512
    ## + LEED:renovated                   1  1113073 61512
    ## + age                              1  1113073 61512
    ## - leasing_rate:size                1  1117627 61540
    ## - class_b                          1  1118090 61543
    ## - renovated                        1  1118306 61545
    ## - Precipitation:Gas_Costs          1  1118387 61545
    ## - Gas_Costs:size                   1  1119143 61551
    ## - Electricity_Costs:Precipitation  1  1124967 61592
    ## - Precipitation:size               1  1125157 61593
    ## - Electricity_Costs:Gas_Costs      1  1137730 61681
    ## - Electricity_Costs:size           1  1163130 61855
    ## 
    ## Step:  AIC=61486.96
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + LEED:Energystar + Electricity_Costs:Gas_Costs + Electricity_Costs:Precipitation + 
    ##     Precipitation:Gas_Costs + Electricity_Costs:class_a + Electricity_Costs:size + 
    ##     Precipitation:size + Gas_Costs:size + leasing_rate:size
    ## 
    ##                                   Df Deviance   AIC
    ## + class_b:Electricity_Costs        1  1106883 61470
    ## + Energystar:Precipitation         1  1107238 61472
    ## + renovated:Gas_Costs              1  1107729 61476
    ## + class_a:Precipitation            1  1108272 61480
    ## + size:class_a                     1  1108514 61481
    ## + renovated:class_a                1  1108528 61482
    ## + size:class_b                     1  1108662 61483
    ## + leasing_rate:Precipitation       1  1108700 61483
    ## + total_dd_07                      1  1108701 61483
    ## + net:Gas_Costs                    1  1108840 61484
    ## + net:Electricity_Costs            1  1109126 61486
    ## + renovated:Electricity_Costs      1  1109141 61486
    ## - Electricity_Costs:class_a        1  1109720 61486
    ## + renovated:class_b                1  1109225 61487
    ## + Energystar:Gas_Costs             1  1109229 61487
    ## + leasing_rate:class_a             1  1109257 61487
    ## <none>                                1109569 61487
    ## - LEED:Energystar                  1  1109859 61487
    ## + stories                          1  1109303 61487
    ## + LEED:Electricity_Costs           1  1109356 61487
    ## + renovated:net                    1  1109359 61487
    ## + leasing_rate:renovated           1  1109361 61487
    ## + renovated:Precipitation          1  1109398 61488
    ## + Energystar:Electricity_Costs     1  1109413 61488
    ## + Energystar:renovated             1  1109415 61488
    ## + Energystar:class_b               1  1109429 61488
    ## + Energystar:size                  1  1109444 61488
    ## + class_b:Precipitation            1  1109446 61488
    ## + class_a:Gas_Costs                1  1109463 61488
    ## + LEED:net                         1  1109466 61488
    ## + leasing_rate:Electricity_Costs   1  1109469 61488
    ## + net:Precipitation                1  1109481 61488
    ## + Energystar:class_a               1  1109485 61488
    ## + LEED:Precipitation               1  1109493 61488
    ## + LEED:Gas_Costs                   1  1109502 61488
    ## + size:net                         1  1109520 61489
    ## + amenities                        1  1109520 61489
    ## + Energystar:net                   1  1109520 61489
    ## + LEED:leasing_rate                1  1109527 61489
    ## + leasing_rate:class_b             1  1109533 61489
    ## + class_a:net                      1  1109544 61489
    ## + class_b:net                      1  1109545 61489
    ## + size:renovated                   1  1109547 61489
    ## + leasing_rate:Gas_Costs           1  1109548 61489
    ## + LEED:class_b                     1  1109550 61489
    ## + LEED:class_a                     1  1109555 61489
    ## + LEED:size                        1  1109557 61489
    ## + class_b:Gas_Costs                1  1109558 61489
    ## + Energystar:leasing_rate          1  1109560 61489
    ## + LEED:renovated                   1  1109568 61489
    ## + age                              1  1109569 61489
    ## + leasing_rate:net                 1  1109569 61489
    ## - net                              1  1113074 61510
    ## - leasing_rate:size                1  1113930 61516
    ## - class_b                          1  1114584 61521
    ## - renovated                        1  1114876 61523
    ## - Precipitation:Gas_Costs          1  1114951 61523
    ## - Gas_Costs:size                   1  1115189 61525
    ## - Precipitation:size               1  1120766 61564
    ## - Electricity_Costs:Precipitation  1  1121812 61572
    ## - Electricity_Costs:Gas_Costs      1  1132804 61649
    ## - Electricity_Costs:size           1  1158425 61825
    ## 
    ## Step:  AIC=61469.83
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + LEED:Energystar + Electricity_Costs:Gas_Costs + Electricity_Costs:Precipitation + 
    ##     Precipitation:Gas_Costs + Electricity_Costs:class_a + Electricity_Costs:size + 
    ##     Precipitation:size + Gas_Costs:size + leasing_rate:size + 
    ##     Electricity_Costs:class_b
    ## 
    ##                                   Df Deviance   AIC
    ## + Energystar:Precipitation         1  1104472 61455
    ## + renovated:Gas_Costs              1  1104762 61457
    ## + class_a:Precipitation            1  1105398 61461
    ## + renovated:class_a                1  1105752 61464
    ## + size:class_a                     1  1105866 61465
    ## + net:Gas_Costs                    1  1105894 61465
    ## + leasing_rate:Precipitation       1  1105971 61465
    ## + total_dd_07                      1  1106133 61466
    ## + size:class_b                     1  1106183 61467
    ## + net:Electricity_Costs            1  1106284 61468
    ## + leasing_rate:Electricity_Costs   1  1106324 61468
    ## + renovated:Electricity_Costs      1  1106502 61469
    ## + Energystar:Gas_Costs             1  1106538 61469
    ## <none>                                1106883 61470
    ## - LEED:Energystar                  1  1107169 61470
    ## + leasing_rate:class_a             1  1106611 61470
    ## + stories                          1  1106614 61470
    ## + leasing_rate:renovated           1  1106623 61470
    ## + renovated:net                    1  1106650 61470
    ## + renovated:Precipitation          1  1106657 61470
    ## + LEED:Electricity_Costs           1  1106704 61471
    ## + renovated:class_b                1  1106729 61471
    ## + Energystar:renovated             1  1106752 61471
    ## + Energystar:Electricity_Costs     1  1106757 61471
    ## + net:Precipitation                1  1106770 61471
    ## + Energystar:size                  1  1106771 61471
    ## + Energystar:class_b               1  1106776 61471
    ## + leasing_rate:Gas_Costs           1  1106779 61471
    ## + LEED:net                         1  1106780 61471
    ## + class_a:Gas_Costs                1  1106786 61471
    ## + LEED:Precipitation               1  1106800 61471
    ## + amenities                        1  1106806 61471
    ## + class_b:Gas_Costs                1  1106807 61471
    ## + Energystar:class_a               1  1106820 61471
    ## + size:net                         1  1106829 61471
    ## + LEED:leasing_rate                1  1106829 61471
    ## + leasing_rate:class_b             1  1106830 61471
    ## + class_b:net                      1  1106836 61471
    ## + Energystar:net                   1  1106838 61472
    ## + LEED:Gas_Costs                   1  1106847 61472
    ## + size:renovated                   1  1106848 61472
    ## + class_a:net                      1  1106855 61472
    ## + LEED:class_a                     1  1106861 61472
    ## + LEED:class_b                     1  1106865 61472
    ## + Energystar:leasing_rate          1  1106868 61472
    ## + LEED:size                        1  1106875 61472
    ## + LEED:renovated                   1  1106879 61472
    ## + leasing_rate:net                 1  1106882 61472
    ## + age                              1  1106883 61472
    ## + class_b:Precipitation            1  1106883 61472
    ## - Electricity_Costs:class_a        1  1109059 61483
    ## - Electricity_Costs:class_b        1  1109569 61487
    ## - net                              1  1110304 61492
    ## - leasing_rate:size                1  1111259 61499
    ## - Gas_Costs:size                   1  1111489 61501
    ## - renovated                        1  1111875 61503
    ## - Precipitation:Gas_Costs          1  1112907 61511
    ## - Precipitation:size               1  1116759 61538
    ## - Electricity_Costs:Precipitation  1  1119052 61554
    ## - Electricity_Costs:Gas_Costs      1  1129566 61628
    ## - Electricity_Costs:size           1  1157123 61818
    ## 
    ## Step:  AIC=61454.61
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + LEED:Energystar + Electricity_Costs:Gas_Costs + Electricity_Costs:Precipitation + 
    ##     Precipitation:Gas_Costs + Electricity_Costs:class_a + Electricity_Costs:size + 
    ##     Precipitation:size + Gas_Costs:size + leasing_rate:size + 
    ##     Electricity_Costs:class_b + Energystar:Precipitation
    ## 
    ##                                   Df Deviance   AIC
    ## + renovated:Gas_Costs              1  1102039 61439
    ## + renovated:class_a                1  1103277 61448
    ## + leasing_rate:Precipitation       1  1103338 61449
    ## + net:Gas_Costs                    1  1103462 61449
    ## + size:class_a                     1  1103488 61450
    ## + class_a:Precipitation            1  1103618 61451
    ## + total_dd_07                      1  1103646 61451
    ## + size:class_b                     1  1103813 61452
    ## + net:Electricity_Costs            1  1103858 61452
    ## + leasing_rate:Electricity_Costs   1  1103930 61453
    ## + Energystar:Electricity_Costs     1  1103956 61453
    ## + renovated:Precipitation          1  1104053 61454
    ## + Energystar:size                  1  1104093 61454
    ## + renovated:Electricity_Costs      1  1104152 61454
    ## + Energystar:class_b               1  1104172 61454
    ## + leasing_rate:renovated           1  1104192 61455
    ## <none>                                1104472 61455
    ## + Energystar:Gas_Costs             1  1104201 61455
    ## + stories                          1  1104207 61455
    ## + leasing_rate:class_a             1  1104212 61455
    ## - LEED:Energystar                  1  1104825 61455
    ## + Energystar:class_a               1  1104267 61455
    ## + renovated:net                    1  1104279 61455
    ## + renovated:class_b                1  1104289 61455
    ## + LEED:Electricity_Costs           1  1104294 61455
    ## + LEED:net                         1  1104337 61456
    ## + leasing_rate:Gas_Costs           1  1104340 61456
    ## + net:Precipitation                1  1104351 61456
    ## + Energystar:renovated             1  1104372 61456
    ## + amenities                        1  1104385 61456
    ## + LEED:Precipitation               1  1104404 61456
    ## + LEED:leasing_rate                1  1104410 61456
    ## + Energystar:net                   1  1104411 61456
    ## + size:net                         1  1104414 61456
    ## + leasing_rate:class_b             1  1104422 61456
    ## + size:renovated                   1  1104422 61456
    ## + class_b:Precipitation            1  1104424 61456
    ## + class_b:net                      1  1104427 61456
    ## + class_b:Gas_Costs                1  1104443 61456
    ## + class_a:Gas_Costs                1  1104444 61456
    ## + class_a:net                      1  1104446 61456
    ## + LEED:Gas_Costs                   1  1104446 61456
    ## + LEED:class_b                     1  1104447 61456
    ## + LEED:class_a                     1  1104456 61456
    ## + LEED:size                        1  1104457 61457
    ## + Energystar:leasing_rate          1  1104460 61457
    ## + leasing_rate:net                 1  1104470 61457
    ## + LEED:renovated                   1  1104470 61457
    ## + age                              1  1104471 61457
    ## - Electricity_Costs:class_a        1  1106846 61470
    ## - Energystar:Precipitation         1  1106883 61470
    ## - Electricity_Costs:class_b        1  1107238 61472
    ## - net                              1  1107899 61477
    ## - leasing_rate:size                1  1108785 61483
    ## - Gas_Costs:size                   1  1108801 61483
    ## - renovated                        1  1109563 61489
    ## - Precipitation:Gas_Costs          1  1110256 61494
    ## - Precipitation:size               1  1114863 61527
    ## - Electricity_Costs:Precipitation  1  1116703 61540
    ## - Electricity_Costs:Gas_Costs      1  1127263 61614
    ## - Electricity_Costs:size           1  1154384 61802
    ## 
    ## Step:  AIC=61439.2
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + LEED:Energystar + Electricity_Costs:Gas_Costs + Electricity_Costs:Precipitation + 
    ##     Precipitation:Gas_Costs + Electricity_Costs:class_a + Electricity_Costs:size + 
    ##     Precipitation:size + Gas_Costs:size + leasing_rate:size + 
    ##     Electricity_Costs:class_b + Energystar:Precipitation + Gas_Costs:renovated
    ## 
    ##                                   Df Deviance   AIC
    ## + class_a:Precipitation            1  1100795 61432
    ## + leasing_rate:Precipitation       1  1100948 61433
    ## + renovated:class_a                1  1101042 61434
    ## + size:class_a                     1  1101043 61434
    ## + total_dd_07                      1  1101175 61435
    ## + size:class_b                     1  1101349 61436
    ## + leasing_rate:Electricity_Costs   1  1101485 61437
    ## + Energystar:Electricity_Costs     1  1101511 61437
    ## + net:Gas_Costs                    1  1101581 61438
    ## + Energystar:size                  1  1101666 61439
    ## + class_a:Gas_Costs                1  1101674 61439
    ## + Energystar:class_b               1  1101728 61439
    ## <none>                                1102039 61439
    ## + net:Electricity_Costs            1  1101770 61439
    ## + leasing_rate:renovated           1  1101788 61439
    ## + leasing_rate:class_a             1  1101794 61439
    ## + Energystar:class_a               1  1101821 61440
    ## + LEED:Electricity_Costs           1  1101847 61440
    ## + stories                          1  1101852 61440
    ## + class_b:Gas_Costs                1  1101856 61440
    ## - LEED:Energystar                  1  1102422 61440
    ## + renovated:Precipitation          1  1101880 61440
    ## + LEED:net                         1  1101880 61440
    ## + renovated:net                    1  1101883 61440
    ## + Energystar:Gas_Costs             1  1101885 61440
    ## + leasing_rate:Gas_Costs           1  1101898 61440
    ## + renovated:Electricity_Costs      1  1101925 61440
    ## + renovated:class_b                1  1101940 61440
    ## + Energystar:net                   1  1101949 61441
    ## + LEED:Precipitation               1  1101954 61441
    ## + amenities                        1  1101964 61441
    ## + LEED:leasing_rate                1  1101971 61441
    ## + Energystar:renovated             1  1101984 61441
    ## + size:net                         1  1101988 61441
    ## + leasing_rate:class_b             1  1101989 61441
    ## + class_b:net                      1  1101990 61441
    ## + net:Precipitation                1  1101991 61441
    ## + LEED:Gas_Costs                   1  1101992 61441
    ## + size:renovated                   1  1101999 61441
    ## + class_a:net                      1  1102005 61441
    ## + LEED:class_b                     1  1102010 61441
    ## + Energystar:leasing_rate          1  1102020 61441
    ## + LEED:class_a                     1  1102024 61441
    ## + LEED:size                        1  1102024 61441
    ## + class_b:Precipitation            1  1102027 61441
    ## + leasing_rate:net                 1  1102029 61441
    ## + LEED:renovated                   1  1102038 61441
    ## + age                              1  1102038 61441
    ## - Gas_Costs:renovated              1  1104472 61455
    ## - Energystar:Precipitation         1  1104762 61457
    ## - Electricity_Costs:class_a        1  1104903 61458
    ## - Electricity_Costs:class_b        1  1105117 61459
    ## - net                              1  1105818 61464
    ## - Gas_Costs:size                   1  1105923 61465
    ## - leasing_rate:size                1  1106517 61469
    ## - Precipitation:Gas_Costs          1  1107410 61476
    ## - Precipitation:size               1  1112045 61509
    ## - Electricity_Costs:Precipitation  1  1114357 61525
    ## - Electricity_Costs:Gas_Costs      1  1126071 61607
    ## - Electricity_Costs:size           1  1153144 61795
    ## 
    ## Step:  AIC=61432.28
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + LEED:Energystar + Electricity_Costs:Gas_Costs + Electricity_Costs:Precipitation + 
    ##     Precipitation:Gas_Costs + Electricity_Costs:class_a + Electricity_Costs:size + 
    ##     Precipitation:size + Gas_Costs:size + leasing_rate:size + 
    ##     Electricity_Costs:class_b + Energystar:Precipitation + Gas_Costs:renovated + 
    ##     class_a:Precipitation
    ## 
    ##                                   Df Deviance   AIC
    ## + class_b:Precipitation            1  1098604 61419
    ## + leasing_rate:Precipitation       1  1099423 61424
    ## + renovated:class_a                1  1099868 61428
    ## + total_dd_07                      1  1100009 61429
    ## + size:class_a                     1  1100047 61429
    ## + leasing_rate:Electricity_Costs   1  1100199 61430
    ## + size:class_b                     1  1100306 61431
    ## + Energystar:Electricity_Costs     1  1100323 61431
    ## + net:Gas_Costs                    1  1100332 61431
    ## + Energystar:size                  1  1100413 61432
    ## + leasing_rate:class_a             1  1100502 61432
    ## <none>                                1100795 61432
    ## + net:Electricity_Costs            1  1100524 61432
    ## + leasing_rate:renovated           1  1100566 61433
    ## + leasing_rate:Gas_Costs           1  1100582 61433
    ## + LEED:Electricity_Costs           1  1100584 61433
    ## + Energystar:class_b               1  1100588 61433
    ## + stories                          1  1100592 61433
    ## - LEED:Energystar                  1  1101174 61433
    ## + renovated:net                    1  1100629 61433
    ## + LEED:net                         1  1100641 61433
    ## + Energystar:class_a               1  1100658 61433
    ## + Energystar:Gas_Costs             1  1100682 61433
    ## + renovated:Electricity_Costs      1  1100705 61434
    ## + renovated:Precipitation          1  1100710 61434
    ## + Energystar:net                   1  1100711 61434
    ## + renovated:class_b                1  1100716 61434
    ## + net:Precipitation                1  1100722 61434
    ## + LEED:Precipitation               1  1100724 61434
    ## + LEED:leasing_rate                1  1100730 61434
    ## + Energystar:renovated             1  1100734 61434
    ## + LEED:Gas_Costs                   1  1100750 61434
    ## + size:net                         1  1100751 61434
    ## + class_b:net                      1  1100763 61434
    ## + leasing_rate:class_b             1  1100766 61434
    ## + size:renovated                   1  1100767 61434
    ## + amenities                        1  1100768 61434
    ## + Energystar:leasing_rate          1  1100768 61434
    ## + LEED:class_a                     1  1100771 61434
    ## + LEED:class_b                     1  1100775 61434
    ## + class_a:net                      1  1100776 61434
    ## + LEED:size                        1  1100778 61434
    ## + class_b:Gas_Costs                1  1100779 61434
    ## + class_a:Gas_Costs                1  1100780 61434
    ## + leasing_rate:net                 1  1100781 61434
    ## + age                              1  1100794 61434
    ## + LEED:renovated                   1  1100795 61434
    ## - class_a:Precipitation            1  1102039 61439
    ## - Energystar:Precipitation         1  1102755 61444
    ## - Gas_Costs:renovated              1  1103618 61451
    ## - Gas_Costs:size                   1  1103955 61453
    ## - Electricity_Costs:class_b        1  1104076 61454
    ## - Electricity_Costs:class_a        1  1104258 61455
    ## - net                              1  1104368 61456
    ## - leasing_rate:size                1  1105191 61462
    ## - Precipitation:Gas_Costs          1  1105483 61464
    ## - Precipitation:size               1  1112035 61510
    ## - Electricity_Costs:Precipitation  1  1113640 61522
    ## - Electricity_Costs:Gas_Costs      1  1125326 61604
    ## - Electricity_Costs:size           1  1152765 61794
    ## 
    ## Step:  AIC=61418.56
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + LEED:Energystar + Electricity_Costs:Gas_Costs + Electricity_Costs:Precipitation + 
    ##     Precipitation:Gas_Costs + Electricity_Costs:class_a + Electricity_Costs:size + 
    ##     Precipitation:size + Gas_Costs:size + leasing_rate:size + 
    ##     Electricity_Costs:class_b + Energystar:Precipitation + Gas_Costs:renovated + 
    ##     class_a:Precipitation + Precipitation:class_b
    ## 
    ##                                   Df Deviance   AIC
    ## + leasing_rate:Precipitation       1  1096496 61405
    ## + leasing_rate:Electricity_Costs   1  1097682 61414
    ## + renovated:class_a                1  1097684 61414
    ## + leasing_rate:Gas_Costs           1  1097684 61414
    ## + size:class_a                     1  1097728 61414
    ## + total_dd_07                      1  1097781 61415
    ## + size:class_b                     1  1097951 61416
    ## + net:Gas_Costs                    1  1098006 61416
    ## + class_b:Gas_Costs                1  1098100 61417
    ## + Energystar:Electricity_Costs     1  1098139 61417
    ## + Energystar:size                  1  1098202 61418
    ## + net:Electricity_Costs            1  1098250 61418
    ## <none>                                1098604 61419
    ## + leasing_rate:class_a             1  1098336 61419
    ## + leasing_rate:renovated           1  1098365 61419
    ## + Energystar:class_b               1  1098365 61419
    ## + stories                          1  1098371 61419
    ## + LEED:Electricity_Costs           1  1098392 61419
    ## + renovated:net                    1  1098427 61419
    ## - LEED:Energystar                  1  1098985 61419
    ## + Energystar:class_a               1  1098438 61419
    ## + LEED:net                         1  1098455 61419
    ## + renovated:Electricity_Costs      1  1098465 61420
    ## + renovated:class_b                1  1098484 61420
    ## + Energystar:Gas_Costs             1  1098492 61420
    ## + renovated:Precipitation          1  1098514 61420
    ## + net:Precipitation                1  1098518 61420
    ## + Energystar:net                   1  1098520 61420
    ## + LEED:leasing_rate                1  1098529 61420
    ## + LEED:Precipitation               1  1098537 61420
    ## + Energystar:renovated             1  1098538 61420
    ## + size:net                         1  1098554 61420
    ## + LEED:class_b                     1  1098571 61420
    ## + class_b:net                      1  1098571 61420
    ## + class_a:Gas_Costs                1  1098576 61420
    ## + LEED:Gas_Costs                   1  1098576 61420
    ## + Energystar:leasing_rate          1  1098578 61420
    ## + LEED:size                        1  1098582 61420
    ## + class_a:net                      1  1098583 61420
    ## + LEED:class_a                     1  1098584 61420
    ## + leasing_rate:class_b             1  1098585 61420
    ## + size:renovated                   1  1098586 61420
    ## + amenities                        1  1098587 61420
    ## + leasing_rate:net                 1  1098592 61420
    ## + LEED:renovated                   1  1098603 61421
    ## + age                              1  1098604 61421
    ## - Energystar:Precipitation         1  1100516 61430
    ## - Precipitation:class_b            1  1100795 61432
    ## - Gas_Costs:size                   1  1101174 61435
    ## - Gas_Costs:renovated              1  1101560 61438
    ## - class_a:Precipitation            1  1102027 61441
    ## - net                              1  1102076 61441
    ## - leasing_rate:size                1  1102780 61447
    ## - Precipitation:Gas_Costs          1  1103517 61452
    ## - Electricity_Costs:class_a        1  1103532 61452
    ## - Electricity_Costs:class_b        1  1103556 61452
    ## - Precipitation:size               1  1110024 61498
    ## - Electricity_Costs:Precipitation  1  1111375 61508
    ## - Electricity_Costs:Gas_Costs      1  1123669 61595
    ## - Electricity_Costs:size           1  1150523 61781
    ## 
    ## Step:  AIC=61405.4
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + LEED:Energystar + Electricity_Costs:Gas_Costs + Electricity_Costs:Precipitation + 
    ##     Precipitation:Gas_Costs + Electricity_Costs:class_a + Electricity_Costs:size + 
    ##     Precipitation:size + Gas_Costs:size + leasing_rate:size + 
    ##     Electricity_Costs:class_b + Energystar:Precipitation + Gas_Costs:renovated + 
    ##     class_a:Precipitation + Precipitation:class_b + leasing_rate:Precipitation
    ## 
    ##                                   Df Deviance   AIC
    ## + leasing_rate:Electricity_Costs   1  1094803 61395
    ## + renovated:class_a                1  1095511 61400
    ## + total_dd_07                      1  1095727 61402
    ## + size:class_a                     1  1095746 61402
    ## + net:Gas_Costs                    1  1095879 61403
    ## + size:class_b                     1  1095906 61403
    ## + Energystar:Electricity_Costs     1  1096036 61404
    ## + leasing_rate:renovated           1  1096091 61404
    ## + Energystar:size                  1  1096116 61405
    ## + net:Electricity_Costs            1  1096137 61405
    ## + class_b:Gas_Costs                1  1096156 61405
    ## <none>                                1096496 61405
    ## + Energystar:class_b               1  1096256 61406
    ## + leasing_rate:class_a             1  1096258 61406
    ## + LEED:Electricity_Costs           1  1096268 61406
    ## - LEED:Energystar                  1  1096864 61406
    ## + stories                          1  1096311 61406
    ## + renovated:net                    1  1096322 61406
    ## + Energystar:class_a               1  1096326 61406
    ## + renovated:class_b                1  1096344 61406
    ## + leasing_rate:class_b             1  1096353 61406
    ## + LEED:net                         1  1096354 61406
    ## + Energystar:Gas_Costs             1  1096368 61406
    ## + renovated:Electricity_Costs      1  1096401 61407
    ## + net:Precipitation                1  1096408 61407
    ## + Energystar:net                   1  1096413 61407
    ## + renovated:Precipitation          1  1096416 61407
    ## + Energystar:renovated             1  1096421 61407
    ## + LEED:Precipitation               1  1096427 61407
    ## + LEED:leasing_rate                1  1096433 61407
    ## + size:net                         1  1096443 61407
    ## + size:renovated                   1  1096457 61407
    ## + LEED:Gas_Costs                   1  1096460 61407
    ## + class_a:Gas_Costs                1  1096467 61407
    ## + leasing_rate:Gas_Costs           1  1096467 61407
    ## + LEED:class_b                     1  1096469 61407
    ## + class_b:net                      1  1096472 61407
    ## + LEED:class_a                     1  1096472 61407
    ## + LEED:size                        1  1096476 61407
    ## + class_a:net                      1  1096481 61407
    ## + Energystar:leasing_rate          1  1096485 61407
    ## + amenities                        1  1096490 61407
    ## + leasing_rate:net                 1  1096490 61407
    ## + age                              1  1096495 61407
    ## + LEED:renovated                   1  1096496 61407
    ## - Energystar:Precipitation         1  1098587 61418
    ## - leasing_rate:Precipitation       1  1098604 61419
    ## - Precipitation:class_b            1  1099423 61424
    ## - Gas_Costs:size                   1  1099431 61424
    ## - Gas_Costs:renovated              1  1099474 61425
    ## - leasing_rate:size                1  1100015 61429
    ## - net                              1  1100027 61429
    ## - class_a:Precipitation            1  1100929 61435
    ## - Precipitation:Gas_Costs          1  1101145 61437
    ## - Electricity_Costs:class_a        1  1101924 61442
    ## - Electricity_Costs:class_b        1  1101965 61443
    ## - Precipitation:size               1  1107493 61482
    ## - Electricity_Costs:Precipitation  1  1108709 61491
    ## - Electricity_Costs:Gas_Costs      1  1121692 61583
    ## - Electricity_Costs:size           1  1148900 61772
    ## 
    ## Step:  AIC=61395.2
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + LEED:Energystar + Electricity_Costs:Gas_Costs + Electricity_Costs:Precipitation + 
    ##     Precipitation:Gas_Costs + Electricity_Costs:class_a + Electricity_Costs:size + 
    ##     Precipitation:size + Gas_Costs:size + leasing_rate:size + 
    ##     Electricity_Costs:class_b + Energystar:Precipitation + Gas_Costs:renovated + 
    ##     class_a:Precipitation + Precipitation:class_b + leasing_rate:Precipitation + 
    ##     Electricity_Costs:leasing_rate
    ## 
    ##                                   Df Deviance   AIC
    ## + renovated:class_a                1  1093741 61390
    ## + size:class_a                     1  1094012 61391
    ## + total_dd_07                      1  1094019 61392
    ## + net:Gas_Costs                    1  1094193 61393
    ## + size:class_b                     1  1094210 61393
    ## + Energystar:Electricity_Costs     1  1094273 61393
    ## + Energystar:size                  1  1094429 61395
    ## + leasing_rate:renovated           1  1094438 61395
    ## + net:Electricity_Costs            1  1094463 61395
    ## + leasing_rate:class_b             1  1094484 61395
    ## <none>                                1094803 61395
    ## + leasing_rate:Gas_Costs           1  1094529 61395
    ## + LEED:Electricity_Costs           1  1094552 61395
    ## + Energystar:class_b               1  1094591 61396
    ## - LEED:Energystar                  1  1095164 61396
    ## + stories                          1  1094623 61396
    ## + renovated:net                    1  1094634 61396
    ## + renovated:class_b                1  1094637 61396
    ## + Energystar:class_a               1  1094653 61396
    ## + LEED:net                         1  1094656 61396
    ## + renovated:Electricity_Costs      1  1094660 61396
    ## + Energystar:Gas_Costs             1  1094682 61396
    ## + renovated:Precipitation          1  1094688 61396
    ## + class_b:Gas_Costs                1  1094692 61396
    ## + net:Precipitation                1  1094711 61397
    ## + leasing_rate:class_a             1  1094716 61397
    ## + Energystar:net                   1  1094719 61397
    ## + LEED:Precipitation               1  1094728 61397
    ## + LEED:Gas_Costs                   1  1094739 61397
    ## + LEED:leasing_rate                1  1094741 61397
    ## + Energystar:renovated             1  1094745 61397
    ## + size:net                         1  1094752 61397
    ## + size:renovated                   1  1094758 61397
    ## + LEED:class_a                     1  1094773 61397
    ## + amenities                        1  1094775 61397
    ## + Energystar:leasing_rate          1  1094777 61397
    ## + LEED:class_b                     1  1094781 61397
    ## + class_b:net                      1  1094783 61397
    ## + leasing_rate:net                 1  1094785 61397
    ## + class_a:Gas_Costs                1  1094788 61397
    ## + LEED:size                        1  1094792 61397
    ## + class_a:net                      1  1094793 61397
    ## + LEED:renovated                   1  1094802 61397
    ## + age                              1  1094802 61397
    ## - Electricity_Costs:leasing_rate   1  1096496 61405
    ## - Energystar:Precipitation         1  1096879 61408
    ## - leasing_rate:Precipitation       1  1097682 61414
    ## - Gas_Costs:renovated              1  1097836 61415
    ## - net                              1  1098340 61419
    ## - Gas_Costs:size                   1  1098361 61419
    ## - Precipitation:class_b            1  1098402 61419
    ## - leasing_rate:size                1  1098996 61423
    ## - Precipitation:Gas_Costs          1  1099034 61424
    ## - class_a:Precipitation            1  1099999 61431
    ## - Electricity_Costs:class_b        1  1101664 61443
    ## - Electricity_Costs:class_a        1  1101712 61443
    ## - Precipitation:size               1  1106371 61476
    ## - Electricity_Costs:Precipitation  1  1106430 61477
    ## - Electricity_Costs:Gas_Costs      1  1120437 61576
    ## - Electricity_Costs:size           1  1144915 61747
    ## 
    ## Step:  AIC=61389.54
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + LEED:Energystar + Electricity_Costs:Gas_Costs + Electricity_Costs:Precipitation + 
    ##     Precipitation:Gas_Costs + Electricity_Costs:class_a + Electricity_Costs:size + 
    ##     Precipitation:size + Gas_Costs:size + leasing_rate:size + 
    ##     Electricity_Costs:class_b + Energystar:Precipitation + Gas_Costs:renovated + 
    ##     class_a:Precipitation + Precipitation:class_b + leasing_rate:Precipitation + 
    ##     Electricity_Costs:leasing_rate + class_a:renovated
    ## 
    ##                                   Df Deviance   AIC
    ## + total_dd_07                      1  1092847 61385
    ## + size:class_a                     1  1093066 61387
    ## + net:Gas_Costs                    1  1093127 61387
    ## + size:class_b                     1  1093231 61388
    ## + Energystar:Electricity_Costs     1  1093263 61388
    ## + net:Electricity_Costs            1  1093391 61389
    ## + Energystar:size                  1  1093407 61389
    ## + leasing_rate:class_b             1  1093429 61389
    ## + leasing_rate:Gas_Costs           1  1093460 61390
    ## <none>                                1093741 61390
    ## + renovated:class_b                1  1093467 61390
    ## + Energystar:renovated             1  1093495 61390
    ## + LEED:Electricity_Costs           1  1093502 61390
    ## - LEED:Energystar                  1  1094126 61390
    ## + leasing_rate:renovated           1  1093579 61390
    ## + Energystar:class_b               1  1093585 61390
    ## + LEED:net                         1  1093594 61390
    ## + stories                          1  1093599 61391
    ## + renovated:Precipitation          1  1093600 61391
    ## + renovated:Electricity_Costs      1  1093616 61391
    ## + class_b:Gas_Costs                1  1093622 61391
    ## + Energystar:Gas_Costs             1  1093622 61391
    ## + Energystar:class_a               1  1093636 61391
    ## + leasing_rate:class_a             1  1093640 61391
    ## + renovated:net                    1  1093656 61391
    ## + size:renovated                   1  1093658 61391
    ## + LEED:Precipitation               1  1093665 61391
    ## + LEED:leasing_rate                1  1093671 61391
    ## + Energystar:net                   1  1093672 61391
    ## + net:Precipitation                1  1093674 61391
    ## + LEED:Gas_Costs                   1  1093686 61391
    ## + size:net                         1  1093708 61391
    ## + amenities                        1  1093709 61391
    ## + Energystar:leasing_rate          1  1093714 61391
    ## + LEED:class_a                     1  1093715 61391
    ## + LEED:class_b                     1  1093717 61391
    ## + class_b:net                      1  1093721 61391
    ## + leasing_rate:net                 1  1093724 61391
    ## + age                              1  1093725 61391
    ## + LEED:size                        1  1093728 61391
    ## + class_a:Gas_Costs                1  1093730 61391
    ## + class_a:net                      1  1093731 61391
    ## + LEED:renovated                   1  1093735 61391
    ## - class_a:renovated                1  1094803 61395
    ## - Electricity_Costs:leasing_rate   1  1095511 61400
    ## - Energystar:Precipitation         1  1095881 61403
    ## - Gas_Costs:renovated              1  1096533 61408
    ## - leasing_rate:Precipitation       1  1096718 61409
    ## - net                              1  1097202 61412
    ## - Precipitation:class_b            1  1097365 61414
    ## - Gas_Costs:size                   1  1097399 61414
    ## - Precipitation:Gas_Costs          1  1097617 61415
    ## - leasing_rate:size                1  1098009 61418
    ## - class_a:Precipitation            1  1098876 61425
    ## - Electricity_Costs:class_b        1  1100750 61438
    ## - Electricity_Costs:class_a        1  1100950 61439
    ## - Precipitation:size               1  1105405 61471
    ## - Electricity_Costs:Precipitation  1  1105523 61472
    ## - Electricity_Costs:Gas_Costs      1  1119930 61574
    ## - Electricity_Costs:size           1  1143639 61740
    ## 
    ## Step:  AIC=61385.08
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + Precipitation:size + 
    ##     Gas_Costs:size + leasing_rate:size + Electricity_Costs:class_b + 
    ##     Energystar:Precipitation + Gas_Costs:renovated + class_a:Precipitation + 
    ##     Precipitation:class_b + leasing_rate:Precipitation + Electricity_Costs:leasing_rate + 
    ##     class_a:renovated
    ## 
    ##                                   Df Deviance   AIC
    ## + total_dd_07:Precipitation        1  1087669 61350
    ## + size:total_dd_07                 1  1090257 61368
    ## + total_dd_07:Gas_Costs            1  1091171 61375
    ## + class_b:total_dd_07              1  1091650 61378
    ## + net:total_dd_07                  1  1091973 61381
    ## + size:class_a                     1  1092119 61382
    ## + net:Gas_Costs                    1  1092238 61383
    ## + size:class_b                     1  1092283 61383
    ## + renovated:total_dd_07            1  1092302 61383
    ## + Energystar:Electricity_Costs     1  1092390 61384
    ## + leasing_rate:class_b             1  1092526 61385
    ## + net:Electricity_Costs            1  1092533 61385
    ## + Energystar:size                  1  1092552 61385
    ## + renovated:class_b                1  1092556 61385
    ## <none>                                1092847 61385
    ## + Energystar:renovated             1  1092583 61385
    ## + LEED:Electricity_Costs           1  1092606 61385
    ## + leasing_rate:Gas_Costs           1  1092613 61385
    ## + stories                          1  1092682 61386
    ## + Energystar:class_b               1  1092685 61386
    ## + LEED:net                         1  1092695 61386
    ## + renovated:Precipitation          1  1092704 61386
    ## - LEED:Energystar                  1  1093258 61386
    ## + leasing_rate:renovated           1  1092705 61386
    ## + class_b:Gas_Costs                1  1092707 61386
    ## + leasing_rate:total_dd_07         1  1092707 61386
    ## + renovated:Electricity_Costs      1  1092721 61386
    ## + Energystar:Gas_Costs             1  1092731 61386
    ## + Energystar:class_a               1  1092735 61386
    ## + leasing_rate:class_a             1  1092748 61386
    ## + size:renovated                   1  1092751 61386
    ## + net:Precipitation                1  1092765 61386
    ## + renovated:net                    1  1092768 61387
    ## + LEED:leasing_rate                1  1092770 61387
    ## + LEED:Precipitation               1  1092772 61387
    ## + Energystar:net                   1  1092779 61387
    ## + LEED:Gas_Costs                   1  1092795 61387
    ## + LEED:class_a                     1  1092816 61387
    ## + class_a:total_dd_07              1  1092816 61387
    ## + Energystar:leasing_rate          1  1092819 61387
    ## + LEED:class_b                     1  1092820 61387
    ## + size:net                         1  1092821 61387
    ## + amenities                        1  1092823 61387
    ## + total_dd_07:Electricity_Costs    1  1092828 61387
    ## + LEED:total_dd_07                 1  1092829 61387
    ## + leasing_rate:net                 1  1092833 61387
    ## + class_b:net                      1  1092833 61387
    ## + LEED:size                        1  1092834 61387
    ## + LEED:renovated                   1  1092835 61387
    ## + class_a:Gas_Costs                1  1092837 61387
    ## + Energystar:total_dd_07           1  1092839 61387
    ## + class_a:net                      1  1092840 61387
    ## + age                              1  1092845 61387
    ## - total_dd_07                      1  1093741 61390
    ## - class_a:renovated                1  1094019 61392
    ## - Electricity_Costs:leasing_rate   1  1094637 61396
    ## - Energystar:Precipitation         1  1095083 61399
    ## - Gas_Costs:renovated              1  1095653 61403
    ## - leasing_rate:Precipitation       1  1095766 61404
    ## - net                              1  1096278 61408
    ## - Gas_Costs:size                   1  1096492 61409
    ## - Precipitation:class_b            1  1096512 61410
    ## - leasing_rate:size                1  1097138 61414
    ## - Precipitation:Gas_Costs          1  1097322 61415
    ## - class_a:Precipitation            1  1097902 61420
    ## - Electricity_Costs:class_b        1  1099701 61432
    ## - Electricity_Costs:class_a        1  1099824 61433
    ## - Electricity_Costs:Precipitation  1  1100313 61437
    ## - Precipitation:size               1  1104534 61467
    ## - Electricity_Costs:Gas_Costs      1  1116099 61549
    ## - Electricity_Costs:size           1  1140499 61720
    ## 
    ## Step:  AIC=61349.6
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + Precipitation:size + 
    ##     Gas_Costs:size + leasing_rate:size + Electricity_Costs:class_b + 
    ##     Energystar:Precipitation + Gas_Costs:renovated + class_a:Precipitation + 
    ##     Precipitation:class_b + leasing_rate:Precipitation + Electricity_Costs:leasing_rate + 
    ##     class_a:renovated + Precipitation:total_dd_07
    ## 
    ##                                   Df Deviance   AIC
    ## + size:total_dd_07                 1  1085190 61334
    ## + class_b:total_dd_07              1  1086315 61342
    ## + total_dd_07:Electricity_Costs    1  1086499 61343
    ## + renovated:total_dd_07            1  1086675 61344
    ## + size:class_a                     1  1086841 61346
    ## + net:total_dd_07                  1  1086940 61346
    ## + size:class_b                     1  1087094 61347
    ## + net:Gas_Costs                    1  1087101 61347
    ## + renovated:class_b                1  1087245 61349
    ## + LEED:Electricity_Costs           1  1087282 61349
    ## + Energystar:Electricity_Costs     1  1087307 61349
    ## + renovated:Precipitation          1  1087347 61349
    ## + Energystar:renovated             1  1087381 61350
    ## + total_dd_07:Gas_Costs            1  1087384 61350
    ## + net:Electricity_Costs            1  1087385 61350
    ## <none>                                1087669 61350
    ## + leasing_rate:class_b             1  1087458 61350
    ## + amenities                        1  1087458 61350
    ## + Energystar:size                  1  1087461 61350
    ## + leasing_rate:Gas_Costs           1  1087488 61350
    ## + Energystar:Gas_Costs             1  1087490 61350
    ## + class_b:Gas_Costs                1  1087505 61350
    ## + size:renovated                   1  1087521 61351
    ## + leasing_rate:class_a             1  1087529 61351
    ## + Energystar:class_b               1  1087531 61351
    ## + LEED:net                         1  1087534 61351
    ## + renovated:net                    1  1087561 61351
    ## + leasing_rate:renovated           1  1087568 61351
    ## + Energystar:class_a               1  1087572 61351
    ## - LEED:Energystar                  1  1088125 61351
    ## + LEED:Precipitation               1  1087575 61351
    ## + LEED:leasing_rate                1  1087579 61351
    ## + leasing_rate:total_dd_07         1  1087595 61351
    ## + net:Precipitation                1  1087598 61351
    ## + renovated:Electricity_Costs      1  1087602 61351
    ## + stories                          1  1087611 61351
    ## + Energystar:net                   1  1087616 61351
    ## + class_a:total_dd_07              1  1087617 61351
    ## + LEED:Gas_Costs                   1  1087621 61351
    ## + LEED:class_b                     1  1087627 61351
    ## + LEED:class_a                     1  1087631 61351
    ## + size:net                         1  1087632 61351
    ## + Energystar:leasing_rate          1  1087639 61351
    ## + Energystar:total_dd_07           1  1087641 61351
    ## + class_b:net                      1  1087641 61351
    ## + class_a:net                      1  1087648 61351
    ## + leasing_rate:net                 1  1087650 61351
    ## + LEED:renovated                   1  1087653 61351
    ## + LEED:size                        1  1087654 61351
    ## + class_a:Gas_Costs                1  1087659 61352
    ## + age                              1  1087662 61352
    ## + LEED:total_dd_07                 1  1087669 61352
    ## - class_a:renovated                1  1088669 61355
    ## - Electricity_Costs:Precipitation  1  1088835 61356
    ## - Electricity_Costs:leasing_rate   1  1089243 61359
    ## - leasing_rate:Precipitation       1  1089881 61364
    ## - Energystar:Precipitation         1  1090203 61366
    ## - Gas_Costs:renovated              1  1090251 61366
    ## - Precipitation:class_b            1  1090836 61371
    ## - net                              1  1091199 61373
    ## - Gas_Costs:size                   1  1091511 61375
    ## - leasing_rate:size                1  1091683 61377
    ## - class_a:Precipitation            1  1092307 61381
    ## - Precipitation:total_dd_07        1  1092847 61385
    ## - Electricity_Costs:class_a        1  1092966 61386
    ## - Electricity_Costs:class_b        1  1093063 61387
    ## - Precipitation:Gas_Costs          1  1093748 61392
    ## - Precipitation:size               1  1100181 61438
    ## - Electricity_Costs:Gas_Costs      1  1109394 61504
    ## - Electricity_Costs:size           1  1131757 61661
    ## 
    ## Step:  AIC=61333.58
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + Precipitation:size + 
    ##     Gas_Costs:size + leasing_rate:size + Electricity_Costs:class_b + 
    ##     Energystar:Precipitation + Gas_Costs:renovated + class_a:Precipitation + 
    ##     Precipitation:class_b + leasing_rate:Precipitation + Electricity_Costs:leasing_rate + 
    ##     class_a:renovated + Precipitation:total_dd_07 + size:total_dd_07
    ## 
    ##                                   Df Deviance   AIC
    ## + class_a:total_dd_07              1  1083791 61325
    ## + renovated:total_dd_07            1  1084331 61329
    ## + size:class_a                     1  1084465 61330
    ## + total_dd_07:Electricity_Costs    1  1084473 61330
    ## + net:Gas_Costs                    1  1084596 61331
    ## + renovated:class_b                1  1084644 61332
    ## + class_b:total_dd_07              1  1084733 61332
    ## + size:class_b                     1  1084736 61332
    ## + net:total_dd_07                  1  1084770 61333
    ## + net:Electricity_Costs            1  1084792 61333
    ## + LEED:Electricity_Costs           1  1084800 61333
    ## + leasing_rate:Gas_Costs           1  1084802 61333
    ## + total_dd_07:Gas_Costs            1  1084822 61333
    ## + Energystar:Electricity_Costs     1  1084871 61333
    ## + Energystar:size                  1  1084875 61333
    ## + Energystar:renovated             1  1084903 61333
    ## + stories                          1  1084907 61334
    ## + renovated:Precipitation          1  1084914 61334
    ## <none>                                1085190 61334
    ## + amenities                        1  1084927 61334
    ## + class_b:Gas_Costs                1  1084929 61334
    ## + leasing_rate:class_b             1  1084962 61334
    ## + renovated:net                    1  1085006 61334
    ## + size:net                         1  1085020 61334
    ## + leasing_rate:class_a             1  1085030 61334
    ## - LEED:Energystar                  1  1085586 61334
    ## + Energystar:class_b               1  1085040 61334
    ## + LEED:net                         1  1085048 61335
    ## + Energystar:total_dd_07           1  1085060 61335
    ## + net:Precipitation                1  1085063 61335
    ## + LEED:leasing_rate                1  1085063 61335
    ## + leasing_rate:renovated           1  1085073 61335
    ## + Energystar:Gas_Costs             1  1085076 61335
    ## + Energystar:class_a               1  1085086 61335
    ## + LEED:Precipitation               1  1085098 61335
    ## + size:renovated                   1  1085104 61335
    ## + class_b:net                      1  1085128 61335
    ## + class_a:net                      1  1085134 61335
    ## + LEED:class_b                     1  1085143 61335
    ## + Energystar:net                   1  1085147 61335
    ## + LEED:class_a                     1  1085154 61335
    ## + LEED:Gas_Costs                   1  1085161 61335
    ## + renovated:Electricity_Costs      1  1085163 61335
    ## + leasing_rate:net                 1  1085163 61335
    ## + LEED:size                        1  1085170 61335
    ## + Energystar:leasing_rate          1  1085173 61335
    ## + class_a:Gas_Costs                1  1085176 61335
    ## + LEED:renovated                   1  1085176 61335
    ## + leasing_rate:total_dd_07         1  1085178 61335
    ## + LEED:total_dd_07                 1  1085190 61336
    ## + age                              1  1085190 61336
    ## - class_a:renovated                1  1086064 61338
    ## - Electricity_Costs:Precipitation  1  1086271 61339
    ## - Electricity_Costs:leasing_rate   1  1086617 61342
    ## - Energystar:Precipitation         1  1087223 61346
    ## - leasing_rate:Precipitation       1  1087433 61348
    ## - size:total_dd_07                 1  1087669 61350
    ## - Gas_Costs:renovated              1  1087899 61351
    ## - Gas_Costs:size                   1  1088151 61353
    ## - Precipitation:class_b            1  1088413 61355
    ## - leasing_rate:size                1  1088927 61359
    ## - net                              1  1089170 61360
    ## - class_a:Precipitation            1  1089670 61364
    ## - Precipitation:size               1  1090027 61367
    ## - Precipitation:total_dd_07        1  1090257 61368
    ## - Electricity_Costs:class_a        1  1090480 61370
    ## - Electricity_Costs:class_b        1  1090662 61371
    ## - Precipitation:Gas_Costs          1  1090765 61372
    ## - Electricity_Costs:Gas_Costs      1  1107677 61493
    ## - Electricity_Costs:size           1  1125919 61622
    ## 
    ## Step:  AIC=61325.4
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + Precipitation:size + 
    ##     Gas_Costs:size + leasing_rate:size + Electricity_Costs:class_b + 
    ##     Energystar:Precipitation + Gas_Costs:renovated + class_a:Precipitation + 
    ##     Precipitation:class_b + leasing_rate:Precipitation + Electricity_Costs:leasing_rate + 
    ##     class_a:renovated + Precipitation:total_dd_07 + size:total_dd_07 + 
    ##     class_a:total_dd_07
    ## 
    ##                                   Df Deviance   AIC
    ## + class_b:total_dd_07              1  1077411 61281
    ## + renovated:total_dd_07            1  1083111 61322
    ## + net:Gas_Costs                    1  1083189 61323
    ## + total_dd_07:Gas_Costs            1  1083200 61323
    ## + renovated:class_b                1  1083295 61324
    ## + total_dd_07:Electricity_Costs    1  1083313 61324
    ## + net:Electricity_Costs            1  1083354 61324
    ## + LEED:Electricity_Costs           1  1083367 61324
    ## + size:class_a                     1  1083372 61324
    ## + net:total_dd_07                  1  1083375 61324
    ## + Energystar:Electricity_Costs     1  1083388 61324
    ## + renovated:Precipitation          1  1083391 61324
    ## + stories                          1  1083394 61325
    ## + Energystar:size                  1  1083427 61325
    ## + leasing_rate:Gas_Costs           1  1083482 61325
    ## <none>                                1083791 61325
    ## + Energystar:renovated             1  1083520 61325
    ## + renovated:net                    1  1083579 61326
    ## + size:class_b                     1  1083580 61326
    ## + amenities                        1  1083590 61326
    ## + leasing_rate:class_a             1  1083601 61326
    ## + size:net                         1  1083603 61326
    ## + leasing_rate:class_b             1  1083616 61326
    ## + LEED:net                         1  1083648 61326
    ## - LEED:Energystar                  1  1084204 61326
    ## + LEED:leasing_rate                1  1083655 61326
    ## + net:Precipitation                1  1083656 61326
    ## + Energystar:class_b               1  1083670 61327
    ## + leasing_rate:renovated           1  1083671 61327
    ## + LEED:Precipitation               1  1083683 61327
    ## + Energystar:Gas_Costs             1  1083696 61327
    ## + Energystar:class_a               1  1083716 61327
    ## + class_b:Gas_Costs                1  1083722 61327
    ## + size:renovated                   1  1083738 61327
    ## + Energystar:net                   1  1083742 61327
    ## + LEED:class_b                     1  1083746 61327
    ## + class_a:Gas_Costs                1  1083751 61327
    ## + LEED:class_a                     1  1083753 61327
    ## + LEED:Gas_Costs                   1  1083753 61327
    ## + class_b:net                      1  1083756 61327
    ## + class_a:net                      1  1083761 61327
    ## + leasing_rate:net                 1  1083764 61327
    ## + renovated:Electricity_Costs      1  1083767 61327
    ## + Energystar:total_dd_07           1  1083767 61327
    ## + LEED:renovated                   1  1083771 61327
    ## + LEED:size                        1  1083771 61327
    ## + Energystar:leasing_rate          1  1083772 61327
    ## + age                              1  1083787 61327
    ## + LEED:total_dd_07                 1  1083791 61327
    ## + leasing_rate:total_dd_07         1  1083791 61327
    ## - class_a:renovated                1  1084769 61331
    ## - Electricity_Costs:Precipitation  1  1084981 61332
    ## - class_a:total_dd_07              1  1085190 61334
    ## - Electricity_Costs:leasing_rate   1  1085201 61334
    ## - Energystar:Precipitation         1  1085836 61338
    ## - leasing_rate:Precipitation       1  1085950 61339
    ## - Gas_Costs:size                   1  1086138 61340
    ## - class_a:Precipitation            1  1086334 61342
    ## - Precipitation:size               1  1086719 61345
    ## - Gas_Costs:renovated              1  1086735 61345
    ## - Precipitation:class_b            1  1086784 61345
    ## - size:total_dd_07                 1  1087617 61351
    ## - net                              1  1087632 61351
    ## - leasing_rate:size                1  1087662 61352
    ## - Precipitation:total_dd_07        1  1088952 61361
    ## - Electricity_Costs:class_b        1  1089074 61362
    ## - Precipitation:Gas_Costs          1  1089611 61366
    ## - Electricity_Costs:class_a        1  1090478 61372
    ## - Electricity_Costs:Gas_Costs      1  1105835 61482
    ## - Electricity_Costs:size           1  1125158 61619
    ## 
    ## Step:  AIC=61280.79
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + Precipitation:size + 
    ##     Gas_Costs:size + leasing_rate:size + Electricity_Costs:class_b + 
    ##     Energystar:Precipitation + Gas_Costs:renovated + class_a:Precipitation + 
    ##     Precipitation:class_b + leasing_rate:Precipitation + Electricity_Costs:leasing_rate + 
    ##     class_a:renovated + Precipitation:total_dd_07 + size:total_dd_07 + 
    ##     class_a:total_dd_07 + class_b:total_dd_07
    ## 
    ##                                   Df Deviance   AIC
    ## + net:Gas_Costs                    1  1076506 61276
    ## + net:Electricity_Costs            1  1076745 61278
    ## + renovated:total_dd_07            1  1076772 61278
    ## + size:class_a                     1  1076793 61278
    ## + total_dd_07:Electricity_Costs    1  1076820 61278
    ## + total_dd_07:Gas_Costs            1  1076952 61279
    ## + class_b:Gas_Costs                1  1076952 61279
    ## + Energystar:Electricity_Costs     1  1076979 61280
    ## + stories                          1  1077005 61280
    ## + size:class_b                     1  1077024 61280
    ## + renovated:Precipitation          1  1077030 61280
    ## + LEED:Electricity_Costs           1  1077045 61280
    ## + net:total_dd_07                  1  1077046 61280
    ## + Energystar:size                  1  1077067 61280
    ## + renovated:class_b                1  1077099 61281
    ## + Energystar:renovated             1  1077106 61281
    ## + size:net                         1  1077109 61281
    ## + renovated:net                    1  1077119 61281
    ## <none>                                1077411 61281
    ## + leasing_rate:total_dd_07         1  1077139 61281
    ## - Precipitation:class_b            1  1077768 61281
    ## + net:Precipitation                1  1077224 61281
    ## - LEED:Energystar                  1  1077790 61282
    ## + leasing_rate:class_b             1  1077255 61282
    ## + Energystar:class_b               1  1077256 61282
    ## + LEED:leasing_rate                1  1077271 61282
    ## + LEED:net                         1  1077277 61282
    ## + amenities                        1  1077278 61282
    ## + leasing_rate:renovated           1  1077282 61282
    ## + leasing_rate:class_a             1  1077301 61282
    ## + LEED:Precipitation               1  1077305 61282
    ## + Energystar:Gas_Costs             1  1077316 61282
    ## + size:renovated                   1  1077326 61282
    ## + Energystar:class_a               1  1077335 61282
    ## + class_b:net                      1  1077351 61282
    ## + class_a:Gas_Costs                1  1077353 61282
    ## + LEED:class_b                     1  1077354 61282
    ## + class_a:net                      1  1077359 61282
    ## + leasing_rate:net                 1  1077368 61282
    ## + Energystar:net                   1  1077370 61282
    ## + LEED:size                        1  1077381 61283
    ## + LEED:class_a                     1  1077385 61283
    ## + LEED:renovated                   1  1077386 61283
    ## + age                              1  1077388 61283
    ## + LEED:Gas_Costs                   1  1077392 61283
    ## + Energystar:leasing_rate          1  1077395 61283
    ## + Energystar:total_dd_07           1  1077399 61283
    ## + renovated:Electricity_Costs      1  1077407 61283
    ## + LEED:total_dd_07                 1  1077410 61283
    ## + leasing_rate:Gas_Costs           1  1077411 61283
    ## - class_a:Precipitation            1  1077974 61283
    ## - class_a:renovated                1  1078207 61285
    ## - Electricity_Costs:Precipitation  1  1078695 61288
    ## - Gas_Costs:size                   1  1079243 61292
    ## - Energystar:Precipitation         1  1079352 61293
    ## - Precipitation:size               1  1079477 61294
    ## - Electricity_Costs:leasing_rate   1  1079814 61296
    ## - leasing_rate:Precipitation       1  1080240 61299
    ## - Gas_Costs:renovated              1  1080516 61302
    ## - leasing_rate:size                1  1081048 61305
    ## - net                              1  1081232 61307
    ## - Precipitation:Gas_Costs          1  1082285 61314
    ## - size:total_dd_07                 1  1082577 61317
    ## - Precipitation:total_dd_07        1  1083515 61323
    ## - class_b:total_dd_07              1  1083791 61325
    ## - class_a:total_dd_07              1  1084733 61332
    ## - Electricity_Costs:class_b        1  1088959 61363
    ## - Electricity_Costs:class_a        1  1089838 61369
    ## - Electricity_Costs:Gas_Costs      1  1101422 61453
    ## - Electricity_Costs:size           1  1120261 61587
    ## 
    ## Step:  AIC=61276.15
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + Precipitation:size + 
    ##     Gas_Costs:size + leasing_rate:size + Electricity_Costs:class_b + 
    ##     Energystar:Precipitation + Gas_Costs:renovated + class_a:Precipitation + 
    ##     Precipitation:class_b + leasing_rate:Precipitation + Electricity_Costs:leasing_rate + 
    ##     class_a:renovated + Precipitation:total_dd_07 + size:total_dd_07 + 
    ##     class_a:total_dd_07 + class_b:total_dd_07 + Gas_Costs:net
    ## 
    ##                                   Df Deviance   AIC
    ## + size:class_a                     1  1075910 61274
    ## + renovated:total_dd_07            1  1075916 61274
    ## + total_dd_07:Electricity_Costs    1  1075956 61274
    ## + total_dd_07:Gas_Costs            1  1076050 61275
    ## + Energystar:Electricity_Costs     1  1076070 61275
    ## + stories                          1  1076071 61275
    ## + class_b:Gas_Costs                1  1076113 61275
    ## + size:class_b                     1  1076136 61275
    ## + Energystar:size                  1  1076153 61276
    ## + net:total_dd_07                  1  1076162 61276
    ## + LEED:Electricity_Costs           1  1076178 61276
    ## + Energystar:renovated             1  1076207 61276
    ## + renovated:class_b                1  1076209 61276
    ## <none>                                1076506 61276
    ## + leasing_rate:total_dd_07         1  1076235 61276
    ## + renovated:Precipitation          1  1076257 61276
    ## - LEED:Energystar                  1  1076861 61277
    ## + size:net                         1  1076328 61277
    ## + leasing_rate:class_b             1  1076340 61277
    ## + Energystar:class_b               1  1076351 61277
    ## - Precipitation:class_b            1  1076897 61277
    ## + renovated:net                    1  1076362 61277
    ## + LEED:leasing_rate                1  1076365 61277
    ## + LEED:net                         1  1076373 61277
    ## + leasing_rate:class_a             1  1076378 61277
    ## + leasing_rate:renovated           1  1076384 61277
    ## + amenities                        1  1076397 61277
    ## + LEED:Precipitation               1  1076399 61277
    ## + class_a:net                      1  1076407 61277
    ## + Energystar:Gas_Costs             1  1076409 61277
    ## + size:renovated                   1  1076410 61277
    ## + class_b:net                      1  1076419 61278
    ## + Energystar:net                   1  1076426 61278
    ## + leasing_rate:net                 1  1076430 61278
    ## + Energystar:class_a               1  1076430 61278
    ## + LEED:class_b                     1  1076443 61278
    ## + class_a:Gas_Costs                1  1076466 61278
    ## + LEED:size                        1  1076473 61278
    ## + age                              1  1076474 61278
    ## + net:Electricity_Costs            1  1076475 61278
    ## + LEED:class_a                     1  1076483 61278
    ## + net:Precipitation                1  1076485 61278
    ## + LEED:renovated                   1  1076487 61278
    ## + Energystar:leasing_rate          1  1076492 61278
    ## + Energystar:total_dd_07           1  1076496 61278
    ## + LEED:Gas_Costs                   1  1076497 61278
    ## + LEED:total_dd_07                 1  1076506 61278
    ## + renovated:Electricity_Costs      1  1076506 61278
    ## + leasing_rate:Gas_Costs           1  1076506 61278
    ## - class_a:Precipitation            1  1077101 61279
    ## - class_a:renovated                1  1077301 61280
    ## - Gas_Costs:net                    1  1077411 61281
    ## - Electricity_Costs:Precipitation  1  1077875 61284
    ## - Energystar:Precipitation         1  1078417 61288
    ## - Gas_Costs:size                   1  1078481 61289
    ## - Precipitation:size               1  1078530 61289
    ## - Gas_Costs:renovated              1  1078801 61291
    ## - Electricity_Costs:leasing_rate   1  1078928 61292
    ## - leasing_rate:Precipitation       1  1079382 61295
    ## - leasing_rate:size                1  1080081 61300
    ## - Precipitation:Gas_Costs          1  1081315 61309
    ## - size:total_dd_07                 1  1081760 61313
    ## - Precipitation:total_dd_07        1  1082582 61319
    ## - class_b:total_dd_07              1  1083189 61323
    ## - class_a:total_dd_07              1  1084095 61330
    ## - Electricity_Costs:class_b        1  1088629 61363
    ## - Electricity_Costs:class_a        1  1089520 61369
    ## - Electricity_Costs:Gas_Costs      1  1101413 61455
    ## - Electricity_Costs:size           1  1119449 61583
    ## 
    ## Step:  AIC=61273.78
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + Precipitation:size + 
    ##     Gas_Costs:size + leasing_rate:size + Electricity_Costs:class_b + 
    ##     Energystar:Precipitation + Gas_Costs:renovated + class_a:Precipitation + 
    ##     Precipitation:class_b + leasing_rate:Precipitation + Electricity_Costs:leasing_rate + 
    ##     class_a:renovated + Precipitation:total_dd_07 + size:total_dd_07 + 
    ##     class_a:total_dd_07 + class_b:total_dd_07 + Gas_Costs:net + 
    ##     class_a:size
    ## 
    ##                                   Df Deviance   AIC
    ## + stories                          1  1075271 61271
    ## + total_dd_07:Electricity_Costs    1  1075284 61271
    ## + renovated:total_dd_07            1  1075310 61271
    ## + class_b:Gas_Costs                1  1075386 61272
    ## + Energystar:size                  1  1075398 61272
    ## + total_dd_07:Gas_Costs            1  1075448 61272
    ## + Energystar:Electricity_Costs     1  1075465 61273
    ## + net:total_dd_07                  1  1075515 61273
    ## + LEED:Electricity_Costs           1  1075593 61273
    ## + renovated:class_b                1  1075613 61274
    ## + Energystar:renovated             1  1075629 61274
    ## <none>                                1075910 61274
    ## + renovated:Precipitation          1  1075638 61274
    ## + leasing_rate:total_dd_07         1  1075646 61274
    ## + size:class_b                     1  1075709 61274
    ## - LEED:Energystar                  1  1076258 61274
    ## + Energystar:class_b               1  1075739 61275
    ## + leasing_rate:class_a             1  1075767 61275
    ## + LEED:leasing_rate                1  1075768 61275
    ## + leasing_rate:class_b             1  1075772 61275
    ## + leasing_rate:renovated           1  1075778 61275
    ## + renovated:net                    1  1075778 61275
    ## - Precipitation:class_b            1  1076324 61275
    ## + LEED:net                         1  1075783 61275
    ## + LEED:Precipitation               1  1075808 61275
    ## + size:net                         1  1075811 61275
    ## + Energystar:Gas_Costs             1  1075818 61275
    ## + Energystar:class_a               1  1075820 61275
    ## + Energystar:net                   1  1075828 61275
    ## + class_a:net                      1  1075830 61275
    ## + size:renovated                   1  1075833 61275
    ## + class_b:net                      1  1075840 61275
    ## + leasing_rate:net                 1  1075842 61275
    ## + LEED:class_b                     1  1075845 61275
    ## + amenities                        1  1075852 61275
    ## + age                              1  1075881 61276
    ## + net:Precipitation                1  1075884 61276
    ## + net:Electricity_Costs            1  1075888 61276
    ## + LEED:class_a                     1  1075888 61276
    ## + Energystar:leasing_rate          1  1075890 61276
    ## + LEED:renovated                   1  1075892 61276
    ## + LEED:size                        1  1075894 61276
    ## + Energystar:total_dd_07           1  1075899 61276
    ## + class_a:Gas_Costs                1  1075902 61276
    ## + LEED:Gas_Costs                   1  1075902 61276
    ## + renovated:Electricity_Costs      1  1075909 61276
    ## + LEED:total_dd_07                 1  1075910 61276
    ## + leasing_rate:Gas_Costs           1  1075910 61276
    ## - class_a:Precipitation            1  1076500 61276
    ## - class_a:size                     1  1076506 61276
    ## - class_a:renovated                1  1076599 61277
    ## - Gas_Costs:net                    1  1076793 61278
    ## - Electricity_Costs:Precipitation  1  1077245 61282
    ## - Gas_Costs:size                   1  1077836 61286
    ## - Energystar:Precipitation         1  1077853 61286
    ## - Precipitation:size               1  1077978 61287
    ## - Gas_Costs:renovated              1  1078171 61288
    ## - Electricity_Costs:leasing_rate   1  1078393 61290
    ## - leasing_rate:Precipitation       1  1078676 61292
    ## - leasing_rate:size                1  1079828 61300
    ## - Precipitation:Gas_Costs          1  1080564 61306
    ## - size:total_dd_07                 1  1080686 61307
    ## - Precipitation:total_dd_07        1  1082086 61317
    ## - class_b:total_dd_07              1  1082788 61322
    ## - class_a:total_dd_07              1  1083140 61325
    ## - Electricity_Costs:class_b        1  1088211 61362
    ## - Electricity_Costs:class_a        1  1089259 61369
    ## - Electricity_Costs:Gas_Costs      1  1101007 61454
    ## - Electricity_Costs:size           1  1119424 61585
    ## 
    ## Step:  AIC=61271.09
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + stories + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + Precipitation:size + 
    ##     Gas_Costs:size + leasing_rate:size + Electricity_Costs:class_b + 
    ##     Energystar:Precipitation + Gas_Costs:renovated + class_a:Precipitation + 
    ##     Precipitation:class_b + leasing_rate:Precipitation + Electricity_Costs:leasing_rate + 
    ##     class_a:renovated + Precipitation:total_dd_07 + size:total_dd_07 + 
    ##     class_a:total_dd_07 + class_b:total_dd_07 + Gas_Costs:net + 
    ##     class_a:size
    ## 
    ##                                   Df Deviance   AIC
    ## + stories:total_dd_07              1  1072273 61251
    ## + stories:Precipitation            1  1073973 61264
    ## + stories:Gas_Costs                1  1074321 61266
    ## + stories:class_a                  1  1074378 61267
    ## + stories:class_b                  1  1074420 61267
    ## + stories:renovated                1  1074499 61267
    ## + renovated:total_dd_07            1  1074668 61269
    ## + total_dd_07:Electricity_Costs    1  1074705 61269
    ## + total_dd_07:Gas_Costs            1  1074759 61269
    ## + class_b:Gas_Costs                1  1074776 61269
    ## + Energystar:size                  1  1074821 61270
    ## + Energystar:Electricity_Costs     1  1074825 61270
    ## + leasing_rate:stories             1  1074853 61270
    ## + size:stories                     1  1074896 61270
    ## + net:total_dd_07                  1  1074902 61270
    ## + renovated:class_b                1  1074958 61271
    ## + renovated:Precipitation          1  1074975 61271
    ## + LEED:Electricity_Costs           1  1074976 61271
    ## + Energystar:renovated             1  1074992 61271
    ## <none>                                1075271 61271
    ## + Energystar:stories               1  1075032 61271
    ## + leasing_rate:total_dd_07         1  1075051 61271
    ## - LEED:Energystar                  1  1075609 61272
    ## + Energystar:class_b               1  1075090 61272
    ## + size:class_b                     1  1075105 61272
    ## + leasing_rate:class_b             1  1075131 61272
    ## + LEED:leasing_rate                1  1075134 61272
    ## + stories:Electricity_Costs        1  1075138 61272
    ## + LEED:net                         1  1075139 61272
    ## + leasing_rate:renovated           1  1075142 61272
    ## + leasing_rate:class_a             1  1075155 61272
    ## + renovated:net                    1  1075156 61272
    ## - Precipitation:class_b            1  1075704 61272
    ## + size:net                         1  1075160 61272
    ## + LEED:Precipitation               1  1075161 61272
    ## + Energystar:class_a               1  1075169 61272
    ## + size:renovated                   1  1075175 61272
    ## + Energystar:Gas_Costs             1  1075179 61272
    ## + class_a:net                      1  1075187 61272
    ## + Energystar:net                   1  1075195 61273
    ## + class_b:net                      1  1075196 61273
    ## + leasing_rate:net                 1  1075198 61273
    ## + LEED:class_b                     1  1075210 61273
    ## + age                              1  1075217 61273
    ## + stories:net                      1  1075223 61273
    ## + net:Precipitation                1  1075237 61273
    ## + amenities                        1  1075242 61273
    ## + Energystar:leasing_rate          1  1075246 61273
    ## + LEED:class_a                     1  1075247 61273
    ## + net:Electricity_Costs            1  1075247 61273
    ## + LEED:size                        1  1075249 61273
    ## + LEED:renovated                   1  1075253 61273
    ## + class_a:Gas_Costs                1  1075255 61273
    ## + LEED:Gas_Costs                   1  1075262 61273
    ## + Energystar:total_dd_07           1  1075262 61273
    ## + LEED:stories                     1  1075264 61273
    ## + renovated:Electricity_Costs      1  1075270 61273
    ## + LEED:total_dd_07                 1  1075271 61273
    ## + leasing_rate:Gas_Costs           1  1075271 61273
    ## - class_a:Precipitation            1  1075847 61273
    ## - class_a:renovated                1  1075880 61274
    ## - stories                          1  1075910 61274
    ## - class_a:size                     1  1076071 61275
    ## - Gas_Costs:net                    1  1076186 61276
    ## - Electricity_Costs:Precipitation  1  1076655 61279
    ## - Energystar:Precipitation         1  1077144 61283
    ## - Gas_Costs:size                   1  1077371 61284
    ## - Gas_Costs:renovated              1  1077424 61285
    ## - Precipitation:size               1  1077515 61286
    ## - Electricity_Costs:leasing_rate   1  1077741 61287
    ## - leasing_rate:Precipitation       1  1077929 61289
    ## - leasing_rate:size                1  1079427 61300
    ## - Precipitation:Gas_Costs          1  1079619 61301
    ## - size:total_dd_07                 1  1080524 61308
    ## - Precipitation:total_dd_07        1  1081138 61312
    ## - class_b:total_dd_07              1  1082204 61320
    ## - class_a:total_dd_07              1  1082671 61323
    ## - Electricity_Costs:class_b        1  1087665 61360
    ## - Electricity_Costs:class_a        1  1088724 61367
    ## - Electricity_Costs:Gas_Costs      1  1101003 61456
    ## - Electricity_Costs:size           1  1119421 61587
    ## 
    ## Step:  AIC=61251.05
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + stories + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + Precipitation:size + 
    ##     Gas_Costs:size + leasing_rate:size + Electricity_Costs:class_b + 
    ##     Energystar:Precipitation + Gas_Costs:renovated + class_a:Precipitation + 
    ##     Precipitation:class_b + leasing_rate:Precipitation + Electricity_Costs:leasing_rate + 
    ##     class_a:renovated + Precipitation:total_dd_07 + size:total_dd_07 + 
    ##     class_a:total_dd_07 + class_b:total_dd_07 + Gas_Costs:net + 
    ##     class_a:size + total_dd_07:stories
    ## 
    ##                                   Df Deviance   AIC
    ## + stories:Gas_Costs                1  1071092 61244
    ## + stories:renovated                1  1071457 61247
    ## + total_dd_07:Gas_Costs            1  1071545 61248
    ## + stories:Electricity_Costs        1  1071677 61249
    ## + Energystar:Electricity_Costs     1  1071678 61249
    ## + Energystar:size                  1  1071780 61249
    ## + leasing_rate:stories             1  1071810 61250
    ## + class_b:Gas_Costs                1  1071817 61250
    ## + size:stories                     1  1071877 61250
    ## + net:total_dd_07                  1  1071878 61250
    ## + LEED:Electricity_Costs           1  1071897 61250
    ## + stories:class_b                  1  1071934 61251
    ## + Energystar:stories               1  1071944 61251
    ## + renovated:total_dd_07            1  1071948 61251
    ## - size:total_dd_07                 1  1072520 61251
    ## + total_dd_07:Electricity_Costs    1  1071986 61251
    ## + stories:class_a                  1  1071988 61251
    ## + Energystar:renovated             1  1072001 61251
    ## <none>                                1072273 61251
    ## + renovated:Precipitation          1  1072009 61251
    ## + renovated:class_b                1  1072029 61251
    ## - LEED:Energystar                  1  1072592 61251
    ## + Energystar:class_b               1  1072061 61251
    ## + leasing_rate:total_dd_07         1  1072078 61252
    ## - Precipitation:class_b            1  1072662 61252
    ## + leasing_rate:class_a             1  1072119 61252
    ## + stories:Precipitation            1  1072123 61252
    ## + leasing_rate:renovated           1  1072129 61252
    ## + leasing_rate:class_b             1  1072132 61252
    ## + LEED:leasing_rate                1  1072140 61252
    ## + LEED:net                         1  1072147 61252
    ## + size:net                         1  1072148 61252
    ## + Energystar:class_a               1  1072154 61252
    ## + renovated:net                    1  1072165 61252
    ## + Energystar:net                   1  1072172 61252
    ## + Energystar:Gas_Costs             1  1072172 61252
    ## + size:renovated                   1  1072174 61252
    ## + LEED:Precipitation               1  1072186 61252
    ## - class_a:Precipitation            1  1072735 61252
    ## + class_a:net                      1  1072195 61252
    ## + LEED:class_b                     1  1072203 61253
    ## + stories:net                      1  1072210 61253
    ## + class_b:net                      1  1072212 61253
    ## + leasing_rate:net                 1  1072215 61253
    ## + size:class_b                     1  1072223 61253
    ## + net:Precipitation                1  1072228 61253
    ## + class_a:Gas_Costs                1  1072238 61253
    ## + LEED:size                        1  1072247 61253
    ## + LEED:renovated                   1  1072251 61253
    ## + Energystar:leasing_rate          1  1072251 61253
    ## + LEED:class_a                     1  1072253 61253
    ## + renovated:Electricity_Costs      1  1072255 61253
    ## + net:Electricity_Costs            1  1072256 61253
    ## + amenities                        1  1072262 61253
    ## + LEED:total_dd_07                 1  1072265 61253
    ## + LEED:stories                     1  1072265 61253
    ## + Energystar:total_dd_07           1  1072266 61253
    ## + LEED:Gas_Costs                   1  1072266 61253
    ## + leasing_rate:Gas_Costs           1  1072268 61253
    ## + age                              1  1072269 61253
    ## - class_a:renovated                1  1072974 61254
    ## - Gas_Costs:net                    1  1073179 61256
    ## - Electricity_Costs:Precipitation  1  1073296 61257
    ## - class_a:size                     1  1073569 61259
    ## - Energystar:Precipitation         1  1073990 61262
    ## - Gas_Costs:renovated              1  1074455 61265
    ## - leasing_rate:Precipitation       1  1074700 61267
    ## - Gas_Costs:size                   1  1074879 61268
    ## - Precipitation:size               1  1074992 61269
    ## - Electricity_Costs:leasing_rate   1  1075075 61270
    ## - total_dd_07:stories              1  1075271 61271
    ## - Precipitation:Gas_Costs          1  1076347 61279
    ## - leasing_rate:size                1  1076692 61282
    ## - Precipitation:total_dd_07        1  1079084 61299
    ## - class_b:total_dd_07              1  1079493 61302
    ## - class_a:total_dd_07              1  1080416 61309
    ## - Electricity_Costs:class_b        1  1084671 61340
    ## - Electricity_Costs:class_a        1  1085599 61347
    ## - Electricity_Costs:Gas_Costs      1  1099339 61446
    ## - Electricity_Costs:size           1  1116208 61566
    ## 
    ## Step:  AIC=61244.35
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + stories + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + Precipitation:size + 
    ##     Gas_Costs:size + leasing_rate:size + Electricity_Costs:class_b + 
    ##     Energystar:Precipitation + Gas_Costs:renovated + class_a:Precipitation + 
    ##     Precipitation:class_b + leasing_rate:Precipitation + Electricity_Costs:leasing_rate + 
    ##     class_a:renovated + Precipitation:total_dd_07 + size:total_dd_07 + 
    ##     class_a:total_dd_07 + class_b:total_dd_07 + Gas_Costs:net + 
    ##     class_a:size + total_dd_07:stories + Gas_Costs:stories
    ## 
    ##                                   Df Deviance   AIC
    ## + stories:Electricity_Costs        1  1069572 61235
    ## + stories:Precipitation            1  1069896 61238
    ## + stories:renovated                1  1070168 61240
    ## + total_dd_07:Gas_Costs            1  1070357 61241
    ## + Energystar:Electricity_Costs     1  1070438 61242
    ## + leasing_rate:stories             1  1070572 61243
    ## + Energystar:size                  1  1070610 61243
    ## + LEED:Electricity_Costs           1  1070649 61243
    ## + renovated:Precipitation          1  1070670 61243
    ## + net:total_dd_07                  1  1070697 61243
    ## + renovated:total_dd_07            1  1070731 61244
    ## - size:total_dd_07                 1  1071288 61244
    ## + leasing_rate:total_dd_07         1  1070762 61244
    ## + class_b:Gas_Costs                1  1070772 61244
    ## + Energystar:stories               1  1070784 61244
    ## + size:stories                     1  1070787 61244
    ## + total_dd_07:Electricity_Costs    1  1070800 61244
    ## + Energystar:renovated             1  1070820 61244
    ## <none>                                1071092 61244
    ## + class_a:Gas_Costs                1  1070823 61244
    ## + renovated:class_b                1  1070859 61245
    ## - Precipitation:class_b            1  1071412 61245
    ## - LEED:Energystar                  1  1071415 61245
    ## - Gas_Costs:size                   1  1071420 61245
    ## - class_a:Precipitation            1  1071431 61245
    ## + Energystar:class_b               1  1070889 61245
    ## + stories:class_b                  1  1070890 61245
    ## + leasing_rate:Gas_Costs           1  1070931 61245
    ## + leasing_rate:renovated           1  1070943 61245
    ## + stories:class_a                  1  1070947 61245
    ## + size:net                         1  1070953 61245
    ## + leasing_rate:class_b             1  1070961 61245
    ## + LEED:leasing_rate                1  1070968 61245
    ## + leasing_rate:class_a             1  1070970 61245
    ## + LEED:net                         1  1070973 61245
    ## + Energystar:class_a               1  1070979 61246
    ## + LEED:Precipitation               1  1070982 61246
    ## + renovated:net                    1  1070986 61246
    ## + Energystar:Gas_Costs             1  1070992 61246
    ## + Energystar:net                   1  1070992 61246
    ## + size:renovated                   1  1070993 61246
    ## + class_a:net                      1  1071028 61246
    ## + LEED:class_b                     1  1071038 61246
    ## + class_b:net                      1  1071046 61246
    ## + size:class_b                     1  1071053 61246
    ## + net:Precipitation                1  1071054 61246
    ## + LEED:Gas_Costs                   1  1071057 61246
    ## + stories:net                      1  1071060 61246
    ## + LEED:class_a                     1  1071065 61246
    ## + Energystar:leasing_rate          1  1071065 61246
    ## + leasing_rate:net                 1  1071067 61246
    ## + LEED:renovated                   1  1071067 61246
    ## + LEED:size                        1  1071075 61246
    ## + net:Electricity_Costs            1  1071078 61246
    ## + Energystar:total_dd_07           1  1071079 61246
    ## + LEED:stories                     1  1071080 61246
    ## + LEED:total_dd_07                 1  1071086 61246
    ## + age                              1  1071086 61246
    ## + renovated:Electricity_Costs      1  1071089 61246
    ## + amenities                        1  1071092 61246
    ## - Gas_Costs:net                    1  1071638 61246
    ## - class_a:renovated                1  1071800 61248
    ## - Electricity_Costs:Precipitation  1  1072051 61249
    ## - Gas_Costs:stories                1  1072273 61251
    ## - class_a:size                     1  1072473 61253
    ## - Energystar:Precipitation         1  1072996 61256
    ## - Gas_Costs:renovated              1  1073711 61262
    ## - Precipitation:size               1  1073742 61262
    ## - leasing_rate:Precipitation       1  1073824 61262
    ## - total_dd_07:stories              1  1074321 61266
    ## - Electricity_Costs:leasing_rate   1  1074437 61267
    ## - Precipitation:Gas_Costs          1  1075421 61274
    ## - leasing_rate:size                1  1075738 61277
    ## - class_b:total_dd_07              1  1077854 61292
    ## - Precipitation:total_dd_07        1  1077989 61293
    ## - class_a:total_dd_07              1  1078642 61298
    ## - Electricity_Costs:class_a        1  1082364 61325
    ## - Electricity_Costs:class_b        1  1082371 61325
    ## - Electricity_Costs:Gas_Costs      1  1092108 61396
    ## - Electricity_Costs:size           1  1112695 61543
    ## 
    ## Step:  AIC=61235.15
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + stories + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + Precipitation:size + 
    ##     Gas_Costs:size + leasing_rate:size + Electricity_Costs:class_b + 
    ##     Energystar:Precipitation + Gas_Costs:renovated + class_a:Precipitation + 
    ##     Precipitation:class_b + leasing_rate:Precipitation + Electricity_Costs:leasing_rate + 
    ##     class_a:renovated + Precipitation:total_dd_07 + size:total_dd_07 + 
    ##     class_a:total_dd_07 + class_b:total_dd_07 + Gas_Costs:net + 
    ##     class_a:size + total_dd_07:stories + Gas_Costs:stories + 
    ##     Electricity_Costs:stories
    ## 
    ##                                   Df Deviance   AIC
    ## + stories:Precipitation            1  1068398 61228
    ## + total_dd_07:Gas_Costs            1  1068634 61230
    ## + stories:renovated                1  1068828 61232
    ## - size:total_dd_07                 1  1069580 61233
    ## + Energystar:Electricity_Costs     1  1069054 61233
    ## + renovated:Precipitation          1  1069079 61234
    ## + Energystar:size                  1  1069111 61234
    ## - Gas_Costs:size                   1  1069689 61234
    ## + renovated:total_dd_07            1  1069159 61234
    ## + net:total_dd_07                  1  1069194 61234
    ## + LEED:Electricity_Costs           1  1069209 61234
    ## + class_a:Gas_Costs                1  1069221 61235
    ## + size:stories                     1  1069241 61235
    ## + Energystar:stories               1  1069266 61235
    ## + class_b:Gas_Costs                1  1069276 61235
    ## + stories:class_b                  1  1069294 61235
    ## <none>                                1069572 61235
    ## + leasing_rate:total_dd_07         1  1069303 61235
    ## + Energystar:renovated             1  1069306 61235
    ## - Precipitation:class_b            1  1069859 61235
    ## + stories:class_a                  1  1069323 61235
    ## + renovated:class_b                1  1069328 61235
    ## - class_a:Precipitation            1  1069873 61235
    ## + leasing_rate:stories             1  1069337 61235
    ## + total_dd_07:Electricity_Costs    1  1069350 61236
    ## - LEED:Energystar                  1  1069905 61236
    ## + leasing_rate:Gas_Costs           1  1069369 61236
    ## + Energystar:class_b               1  1069389 61236
    ## + leasing_rate:renovated           1  1069429 61236
    ## + size:net                         1  1069433 61236
    ## + leasing_rate:class_b             1  1069444 61236
    ## + LEED:net                         1  1069445 61236
    ## + LEED:leasing_rate                1  1069446 61236
    ## + Energystar:net                   1  1069453 61236
    ## + LEED:Precipitation               1  1069459 61236
    ## + renovated:net                    1  1069463 61236
    ## + size:renovated                   1  1069473 61236
    ## + Energystar:class_a               1  1069474 61236
    ## + leasing_rate:class_a             1  1069478 61236
    ## + Energystar:Gas_Costs             1  1069486 61237
    ## + net:Precipitation                1  1069490 61237
    ## + class_a:net                      1  1069506 61237
    ## + LEED:class_b                     1  1069512 61237
    ## + size:class_b                     1  1069514 61237
    ## + class_b:net                      1  1069524 61237
    ## + stories:net                      1  1069527 61237
    ## + leasing_rate:net                 1  1069530 61237
    ## + LEED:Gas_Costs                   1  1069543 61237
    ## + age                              1  1069543 61237
    ## + Energystar:leasing_rate          1  1069547 61237
    ## + LEED:renovated                   1  1069549 61237
    ## + LEED:class_a                     1  1069552 61237
    ## + LEED:stories                     1  1069552 61237
    ## + Energystar:total_dd_07           1  1069556 61237
    ## + net:Electricity_Costs            1  1069559 61237
    ## + LEED:size                        1  1069560 61237
    ## + amenities                        1  1069566 61237
    ## + renovated:Electricity_Costs      1  1069568 61237
    ## + LEED:total_dd_07                 1  1069570 61237
    ## - Gas_Costs:net                    1  1070190 61238
    ## - class_a:renovated                1  1070231 61238
    ## - Electricity_Costs:Precipitation  1  1070265 61238
    ## - class_a:size                     1  1070827 61242
    ## - Electricity_Costs:stories        1  1071092 61244
    ## - Energystar:Precipitation         1  1071491 61247
    ## - Gas_Costs:stories                1  1071677 61249
    ## - leasing_rate:Precipitation       1  1072134 61252
    ## - Gas_Costs:renovated              1  1072137 61252
    ## - Electricity_Costs:leasing_rate   1  1072633 61256
    ## - Precipitation:size               1  1072671 61256
    ## - Precipitation:Gas_Costs          1  1073149 61260
    ## - leasing_rate:size                1  1074287 61268
    ## - total_dd_07:stories              1  1074306 61268
    ## - Precipitation:total_dd_07        1  1076523 61284
    ## - class_b:total_dd_07              1  1076662 61285
    ## - class_a:total_dd_07              1  1077842 61294
    ## - Electricity_Costs:class_b        1  1081281 61319
    ## - Electricity_Costs:class_a        1  1081922 61324
    ## - Electricity_Costs:size           1  1082626 61329
    ## - Electricity_Costs:Gas_Costs      1  1091301 61392
    ## 
    ## Step:  AIC=61228.47
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + stories + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + Precipitation:size + 
    ##     Gas_Costs:size + leasing_rate:size + Electricity_Costs:class_b + 
    ##     Energystar:Precipitation + Gas_Costs:renovated + class_a:Precipitation + 
    ##     Precipitation:class_b + leasing_rate:Precipitation + Electricity_Costs:leasing_rate + 
    ##     class_a:renovated + Precipitation:total_dd_07 + size:total_dd_07 + 
    ##     class_a:total_dd_07 + class_b:total_dd_07 + Gas_Costs:net + 
    ##     class_a:size + total_dd_07:stories + Gas_Costs:stories + 
    ##     Electricity_Costs:stories + Precipitation:stories
    ## 
    ##                                   Df Deviance   AIC
    ## + total_dd_07:Gas_Costs            1  1067485 61224
    ## + stories:renovated                1  1067626 61225
    ## + class_a:Gas_Costs                1  1067852 61226
    ## - Gas_Costs:size                   1  1068401 61226
    ## - Precipitation:size               1  1068454 61227
    ## + Energystar:Electricity_Costs     1  1067917 61227
    ## + renovated:Precipitation          1  1067920 61227
    ## + renovated:total_dd_07            1  1067921 61227
    ## + Energystar:size                  1  1067948 61227
    ## - size:total_dd_07                 1  1068533 61227
    ## + size:stories                     1  1067993 61227
    ## + LEED:Electricity_Costs           1  1068015 61228
    ## + leasing_rate:Gas_Costs           1  1068055 61228
    ## + net:total_dd_07                  1  1068070 61228
    ## + Energystar:stories               1  1068080 61228
    ## + total_dd_07:Electricity_Costs    1  1068083 61228
    ## + leasing_rate:total_dd_07         1  1068089 61228
    ## + stories:class_b                  1  1068104 61228
    ## <none>                                1068398 61228
    ## + Energystar:renovated             1  1068138 61229
    ## + stories:class_a                  1  1068140 61229
    ## + renovated:class_b                1  1068156 61229
    ## + class_b:Gas_Costs                1  1068158 61229
    ## - Precipitation:class_b            1  1068711 61229
    ## - LEED:Energystar                  1  1068717 61229
    ## + size:net                         1  1068178 61229
    ## + leasing_rate:stories             1  1068181 61229
    ## + Energystar:class_b               1  1068220 61229
    ## + leasing_rate:renovated           1  1068223 61229
    ## + Energystar:net                   1  1068266 61230
    ## + LEED:net                         1  1068267 61230
    ## + renovated:net                    1  1068271 61230
    ## + leasing_rate:class_b             1  1068280 61230
    ## + leasing_rate:class_a             1  1068282 61230
    ## + LEED:leasing_rate                1  1068283 61230
    ## + Energystar:class_a               1  1068302 61230
    ## + LEED:Precipitation               1  1068308 61230
    ## - class_a:Precipitation            1  1068858 61230
    ## + class_a:net                      1  1068323 61230
    ## + Energystar:Gas_Costs             1  1068327 61230
    ## + stories:net                      1  1068330 61230
    ## + size:renovated                   1  1068335 61230
    ## + LEED:class_b                     1  1068340 61230
    ## + net:Precipitation                1  1068341 61230
    ## + class_b:net                      1  1068347 61230
    ## - Electricity_Costs:Precipitation  1  1068890 61230
    ## + size:class_b                     1  1068349 61230
    ## + leasing_rate:net                 1  1068358 61230
    ## + LEED:Gas_Costs                   1  1068360 61230
    ## + Energystar:leasing_rate          1  1068364 61230
    ## + LEED:renovated                   1  1068373 61230
    ## + age                              1  1068374 61230
    ## - Gas_Costs:net                    1  1068919 61230
    ## + LEED:class_a                     1  1068379 61230
    ## + LEED:stories                     1  1068382 61230
    ## + LEED:size                        1  1068382 61230
    ## + net:Electricity_Costs            1  1068388 61230
    ## + amenities                        1  1068392 61230
    ## + Energystar:total_dd_07           1  1068394 61230
    ## + LEED:total_dd_07                 1  1068396 61230
    ## + renovated:Electricity_Costs      1  1068396 61230
    ## - class_a:renovated                1  1069017 61231
    ## - Precipitation:stories            1  1069572 61235
    ## - class_a:size                     1  1069733 61236
    ## - Electricity_Costs:stories        1  1069896 61238
    ## - Energystar:Precipitation         1  1070001 61238
    ## - total_dd_07:stories              1  1070767 61244
    ## - leasing_rate:Precipitation       1  1070921 61245
    ## - Precipitation:Gas_Costs          1  1071025 61246
    ## - Gas_Costs:renovated              1  1071341 61248
    ## - Electricity_Costs:leasing_rate   1  1071607 61250
    ## - Gas_Costs:stories                1  1071616 61250
    ## - leasing_rate:size                1  1073133 61261
    ## - Precipitation:total_dd_07        1  1075174 61276
    ## - class_b:total_dd_07              1  1075236 61277
    ## - class_a:total_dd_07              1  1075903 61282
    ## - Electricity_Costs:class_b        1  1079712 61310
    ## - Electricity_Costs:class_a        1  1080192 61313
    ## - Electricity_Costs:size           1  1082271 61328
    ## - Electricity_Costs:Gas_Costs      1  1090083 61385
    ## 
    ## Step:  AIC=61223.73
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + stories + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + Precipitation:size + 
    ##     Gas_Costs:size + leasing_rate:size + Electricity_Costs:class_b + 
    ##     Energystar:Precipitation + Gas_Costs:renovated + class_a:Precipitation + 
    ##     Precipitation:class_b + leasing_rate:Precipitation + Electricity_Costs:leasing_rate + 
    ##     class_a:renovated + Precipitation:total_dd_07 + size:total_dd_07 + 
    ##     class_a:total_dd_07 + class_b:total_dd_07 + Gas_Costs:net + 
    ##     class_a:size + total_dd_07:stories + Gas_Costs:stories + 
    ##     Electricity_Costs:stories + Precipitation:stories + Gas_Costs:total_dd_07
    ## 
    ##                                   Df Deviance   AIC
    ## + stories:renovated                1  1066838 61221
    ## + class_a:Gas_Costs                1  1066840 61221
    ## + renovated:Precipitation          1  1066880 61221
    ## - Gas_Costs:size                   1  1067498 61222
    ## + Energystar:Electricity_Costs     1  1066974 61222
    ## + renovated:total_dd_07            1  1066995 61222
    ## - Precipitation:size               1  1067546 61222
    ## + size:stories                     1  1067015 61222
    ## - size:total_dd_07                 1  1067610 61223
    ## + Energystar:size                  1  1067072 61223
    ## + LEED:Electricity_Costs           1  1067098 61223
    ## + total_dd_07:Electricity_Costs    1  1067107 61223
    ## + leasing_rate:total_dd_07         1  1067119 61223
    ## - Electricity_Costs:Precipitation  1  1067663 61223
    ## + net:total_dd_07                  1  1067152 61223
    ## + leasing_rate:Gas_Costs           1  1067165 61223
    ## + stories:class_b                  1  1067176 61223
    ## + renovated:class_b                1  1067186 61224
    ## + stories:class_a                  1  1067192 61224
    ## + Energystar:stories               1  1067207 61224
    ## <none>                                1067485 61224
    ## + size:net                         1  1067228 61224
    ## + Energystar:renovated             1  1067250 61224
    ## - LEED:Energystar                  1  1067794 61224
    ## + class_b:Gas_Costs                1  1067276 61224
    ## + leasing_rate:stories             1  1067277 61224
    ## + leasing_rate:renovated           1  1067312 61224
    ## + Energystar:class_b               1  1067314 61224
    ## - Precipitation:class_b            1  1067886 61225
    ## + LEED:net                         1  1067352 61225
    ## + LEED:leasing_rate                1  1067357 61225
    ## + renovated:net                    1  1067361 61225
    ## + Energystar:net                   1  1067362 61225
    ## + leasing_rate:class_b             1  1067364 61225
    ## + Energystar:Gas_Costs             1  1067384 61225
    ## + leasing_rate:class_a             1  1067389 61225
    ## + Energystar:class_a               1  1067390 61225
    ## + LEED:Precipitation               1  1067391 61225
    ## + size:renovated                   1  1067395 61225
    ## + stories:net                      1  1067400 61225
    ## + class_a:net                      1  1067406 61225
    ## + LEED:class_b                     1  1067423 61225
    ## - class_a:Precipitation            1  1067971 61225
    ## + leasing_rate:net                 1  1067431 61225
    ## + class_b:net                      1  1067434 61225
    ## + net:Precipitation                1  1067435 61225
    ## + age                              1  1067435 61225
    ## + size:class_b                     1  1067436 61225
    ## + Energystar:leasing_rate          1  1067443 61225
    ## + LEED:Gas_Costs                   1  1067449 61225
    ## + LEED:stories                     1  1067460 61226
    ## + LEED:class_a                     1  1067465 61226
    ## + amenities                        1  1067466 61226
    ## + LEED:renovated                   1  1067467 61226
    ## - Gas_Costs:net                    1  1068009 61226
    ## + net:Electricity_Costs            1  1067475 61226
    ## + LEED:size                        1  1067475 61226
    ## + Energystar:total_dd_07           1  1067480 61226
    ## + renovated:Electricity_Costs      1  1067483 61226
    ## + LEED:total_dd_07                 1  1067484 61226
    ## - class_a:renovated                1  1068144 61227
    ## - Gas_Costs:total_dd_07            1  1068398 61228
    ## - Precipitation:stories            1  1068634 61230
    ## - class_a:size                     1  1068864 61232
    ## - Energystar:Precipitation         1  1069134 61234
    ## - Electricity_Costs:stories        1  1069183 61234
    ## - leasing_rate:Precipitation       1  1069741 61238
    ## - total_dd_07:stories              1  1070160 61241
    ## - Precipitation:Gas_Costs          1  1070209 61242
    ## - Gas_Costs:renovated              1  1070546 61244
    ## - Gas_Costs:stories                1  1070786 61246
    ## - Electricity_Costs:leasing_rate   1  1070794 61246
    ## - leasing_rate:size                1  1072108 61256
    ## - Precipitation:total_dd_07        1  1073592 61267
    ## - class_b:total_dd_07              1  1074179 61271
    ## - class_a:total_dd_07              1  1075338 61280
    ## - Electricity_Costs:Gas_Costs      1  1075520 61281
    ## - Electricity_Costs:class_b        1  1078673 61304
    ## - Electricity_Costs:class_a        1  1079559 61311
    ## - Electricity_Costs:size           1  1080939 61321
    ## 
    ## Step:  AIC=61220.94
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + stories + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + Precipitation:size + 
    ##     Gas_Costs:size + leasing_rate:size + Electricity_Costs:class_b + 
    ##     Energystar:Precipitation + Gas_Costs:renovated + class_a:Precipitation + 
    ##     Precipitation:class_b + leasing_rate:Precipitation + Electricity_Costs:leasing_rate + 
    ##     class_a:renovated + Precipitation:total_dd_07 + size:total_dd_07 + 
    ##     class_a:total_dd_07 + class_b:total_dd_07 + Gas_Costs:net + 
    ##     class_a:size + total_dd_07:stories + Gas_Costs:stories + 
    ##     Electricity_Costs:stories + Precipitation:stories + Gas_Costs:total_dd_07 + 
    ##     renovated:stories
    ## 
    ##                                   Df Deviance   AIC
    ## + size:renovated                   1  1064601 61206
    ## + renovated:Precipitation          1  1065946 61216
    ## + renovated:total_dd_07            1  1065963 61216
    ## + class_a:Gas_Costs                1  1066135 61218
    ## + stories:class_a                  1  1066297 61219
    ## - Gas_Costs:size                   1  1066849 61219
    ## + stories:class_b                  1  1066343 61219
    ## - Precipitation:size               1  1066887 61219
    ## + Energystar:Electricity_Costs     1  1066352 61219
    ## - class_a:renovated                1  1066958 61220
    ## - size:total_dd_07                 1  1066991 61220
    ## + LEED:Electricity_Costs           1  1066452 61220
    ## + leasing_rate:total_dd_07         1  1066458 61220
    ## + total_dd_07:Electricity_Costs    1  1066477 61220
    ## - Electricity_Costs:Precipitation  1  1067038 61220
    ## + leasing_rate:Gas_Costs           1  1066507 61220
    ## + size:stories                     1  1066517 61221
    ## + Energystar:size                  1  1066518 61221
    ## + net:total_dd_07                  1  1066520 61221
    ## <none>                                1066838 61221
    ## - LEED:Energystar                  1  1067141 61221
    ## + leasing_rate:stories             1  1066620 61221
    ## + size:net                         1  1066627 61221
    ## + renovated:class_b                1  1066629 61221
    ## + class_b:Gas_Costs                1  1066635 61221
    ## + Energystar:stories               1  1066645 61222
    ## + Energystar:renovated             1  1066660 61222
    ## + Energystar:class_b               1  1066666 61222
    ## - Precipitation:class_b            1  1067232 61222
    ## + LEED:net                         1  1066704 61222
    ## + LEED:leasing_rate                1  1066719 61222
    ## + leasing_rate:class_b             1  1066726 61222
    ## + leasing_rate:renovated           1  1066727 61222
    ## + Energystar:Gas_Costs             1  1066731 61222
    ## + leasing_rate:class_a             1  1066732 61222
    ## + Energystar:net                   1  1066737 61222
    ## + age                              1  1066738 61222
    ## + Energystar:class_a               1  1066742 61222
    ## + LEED:Precipitation               1  1066744 61222
    ## + renovated:net                    1  1066758 61222
    ## + class_a:net                      1  1066765 61222
    ## + net:Precipitation                1  1066766 61222
    ## + stories:net                      1  1066767 61222
    ## + size:class_b                     1  1066774 61222
    ## - class_a:Precipitation            1  1067317 61222
    ## + LEED:class_b                     1  1066781 61223
    ## - Gas_Costs:net                    1  1067327 61223
    ## + class_b:net                      1  1066790 61223
    ## + leasing_rate:net                 1  1066791 61223
    ## + LEED:Gas_Costs                   1  1066800 61223
    ## + Energystar:leasing_rate          1  1066804 61223
    ## + LEED:class_a                     1  1066816 61223
    ## + amenities                        1  1066818 61223
    ## + LEED:size                        1  1066818 61223
    ## + LEED:renovated                   1  1066823 61223
    ## + net:Electricity_Costs            1  1066825 61223
    ## + LEED:stories                     1  1066826 61223
    ## + renovated:Electricity_Costs      1  1066830 61223
    ## + Energystar:total_dd_07           1  1066835 61223
    ## + LEED:total_dd_07                 1  1066837 61223
    ## - renovated:stories                1  1067485 61224
    ## - Gas_Costs:total_dd_07            1  1067626 61225
    ## - Precipitation:stories            1  1068013 61228
    ## - Electricity_Costs:stories        1  1068341 61230
    ## - class_a:size                     1  1068418 61231
    ## - Energystar:Precipitation         1  1068504 61231
    ## - leasing_rate:Precipitation       1  1069233 61237
    ## - total_dd_07:stories              1  1069382 61238
    ## - Precipitation:Gas_Costs          1  1069609 61239
    ## - Gas_Costs:renovated              1  1069935 61242
    ## - Electricity_Costs:leasing_rate   1  1070138 61243
    ## - Gas_Costs:stories                1  1070204 61244
    ## - leasing_rate:size                1  1071261 61252
    ## - Precipitation:total_dd_07        1  1072531 61261
    ## - class_b:total_dd_07              1  1073479 61268
    ## - class_a:total_dd_07              1  1074616 61276
    ## - Electricity_Costs:Gas_Costs      1  1074971 61279
    ## - Electricity_Costs:class_b        1  1077925 61301
    ## - Electricity_Costs:class_a        1  1078741 61307
    ## - Electricity_Costs:size           1  1080555 61320
    ## 
    ## Step:  AIC=61206.37
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + stories + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + Precipitation:size + 
    ##     Gas_Costs:size + leasing_rate:size + Electricity_Costs:class_b + 
    ##     Energystar:Precipitation + Gas_Costs:renovated + class_a:Precipitation + 
    ##     Precipitation:class_b + leasing_rate:Precipitation + Electricity_Costs:leasing_rate + 
    ##     class_a:renovated + Precipitation:total_dd_07 + size:total_dd_07 + 
    ##     class_a:total_dd_07 + class_b:total_dd_07 + Gas_Costs:net + 
    ##     class_a:size + total_dd_07:stories + Gas_Costs:stories + 
    ##     Electricity_Costs:stories + Precipitation:stories + Gas_Costs:total_dd_07 + 
    ##     renovated:stories + size:renovated
    ## 
    ##                                   Df Deviance   AIC
    ## + renovated:Precipitation          1  1063869 61203
    ## + class_a:Gas_Costs                1  1063908 61203
    ## + renovated:total_dd_07            1  1063913 61203
    ## + stories:class_a                  1  1063949 61204
    ## + Energystar:Electricity_Costs     1  1064058 61204
    ## - Gas_Costs:size                   1  1064610 61204
    ## + stories:class_b                  1  1064075 61204
    ## + size:stories                     1  1064139 61205
    ## - size:total_dd_07                 1  1064707 61205
    ## + Energystar:size                  1  1064189 61205
    ## - Precipitation:size               1  1064736 61205
    ## + LEED:Electricity_Costs           1  1064226 61206
    ## + total_dd_07:Electricity_Costs    1  1064240 61206
    ## + leasing_rate:total_dd_07         1  1064241 61206
    ## + leasing_rate:Gas_Costs           1  1064251 61206
    ## + net:total_dd_07                  1  1064270 61206
    ## - Electricity_Costs:Precipitation  1  1064821 61206
    ## <none>                                1064601 61206
    ## + renovated:class_b                1  1064341 61206
    ## + size:net                         1  1064353 61207
    ## + class_b:Gas_Costs                1  1064362 61207
    ## - LEED:Energystar                  1  1064906 61207
    ## - class_a:renovated                1  1064918 61207
    ## + Energystar:stories               1  1064381 61207
    ## + leasing_rate:stories             1  1064387 61207
    ## + Energystar:class_b               1  1064408 61207
    ## + renovated:net                    1  1064454 61207
    ## + leasing_rate:renovated           1  1064456 61207
    ## - Precipitation:class_b            1  1065002 61207
    ## + LEED:net                         1  1064470 61207
    ## + Energystar:renovated             1  1064474 61207
    ## + Energystar:net                   1  1064482 61207
    ## + Energystar:class_a               1  1064488 61208
    ## + leasing_rate:class_a             1  1064492 61208
    ## + LEED:leasing_rate                1  1064495 61208
    ## + Energystar:Gas_Costs             1  1064495 61208
    ## + LEED:Precipitation               1  1064501 61208
    ## + leasing_rate:class_b             1  1064511 61208
    ## + stories:net                      1  1064518 61208
    ## + class_a:net                      1  1064527 61208
    ## + net:Precipitation                1  1064533 61208
    ## + age                              1  1064535 61208
    ## + leasing_rate:net                 1  1064544 61208
    ## + class_b:net                      1  1064552 61208
    ## + LEED:Gas_Costs                   1  1064552 61208
    ## - Gas_Costs:net                    1  1065093 61208
    ## + LEED:class_b                     1  1064558 61208
    ## + Energystar:leasing_rate          1  1064560 61208
    ## + LEED:class_a                     1  1064568 61208
    ## + size:class_b                     1  1064571 61208
    ## + LEED:stories                     1  1064576 61208
    ## + net:Electricity_Costs            1  1064587 61208
    ## + amenities                        1  1064592 61208
    ## - class_a:Precipitation            1  1065132 61208
    ## + Energystar:total_dd_07           1  1064593 61208
    ## + LEED:size                        1  1064595 61208
    ## + renovated:Electricity_Costs      1  1064596 61208
    ## + LEED:renovated                   1  1064597 61208
    ## + LEED:total_dd_07                 1  1064600 61208
    ## - Gas_Costs:total_dd_07            1  1065345 61210
    ## - Precipitation:stories            1  1065524 61211
    ## - Electricity_Costs:stories        1  1065739 61213
    ## - Energystar:Precipitation         1  1066261 61217
    ## - class_a:size                     1  1066503 61218
    ## - size:renovated                   1  1066838 61221
    ## - leasing_rate:Precipitation       1  1067059 61223
    ## - total_dd_07:stories              1  1067112 61223
    ## - Precipitation:Gas_Costs          1  1067386 61225
    ## - renovated:stories                1  1067395 61225
    ## - Gas_Costs:renovated              1  1067555 61226
    ## - Gas_Costs:stories                1  1067880 61229
    ## - Electricity_Costs:leasing_rate   1  1067914 61229
    ## - leasing_rate:size                1  1069369 61240
    ## - Precipitation:total_dd_07        1  1070155 61245
    ## - class_b:total_dd_07              1  1071418 61255
    ## - class_a:total_dd_07              1  1072233 61261
    ## - Electricity_Costs:Gas_Costs      1  1072775 61265
    ## - Electricity_Costs:class_b        1  1075636 61286
    ## - Electricity_Costs:class_a        1  1076636 61293
    ## - Electricity_Costs:size           1  1079472 61314
    ## 
    ## Step:  AIC=61202.94
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + stories + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + Precipitation:size + 
    ##     Gas_Costs:size + leasing_rate:size + Electricity_Costs:class_b + 
    ##     Energystar:Precipitation + Gas_Costs:renovated + class_a:Precipitation + 
    ##     Precipitation:class_b + leasing_rate:Precipitation + Electricity_Costs:leasing_rate + 
    ##     class_a:renovated + Precipitation:total_dd_07 + size:total_dd_07 + 
    ##     class_a:total_dd_07 + class_b:total_dd_07 + Gas_Costs:net + 
    ##     class_a:size + total_dd_07:stories + Gas_Costs:stories + 
    ##     Electricity_Costs:stories + Precipitation:stories + Gas_Costs:total_dd_07 + 
    ##     renovated:stories + size:renovated + Precipitation:renovated
    ## 
    ##                                   Df Deviance   AIC
    ## + stories:class_a                  1  1063172 61200
    ## + stories:class_b                  1  1063289 61201
    ## + Energystar:Electricity_Costs     1  1063314 61201
    ## + class_a:Gas_Costs                1  1063348 61201
    ## - Gas_Costs:size                   1  1063904 61201
    ## + total_dd_07:Electricity_Costs    1  1063423 61202
    ## - size:total_dd_07                 1  1063980 61202
    ## + size:stories                     1  1063441 61202
    ## - Precipitation:size               1  1063989 61202
    ## + LEED:Electricity_Costs           1  1063457 61202
    ## + leasing_rate:total_dd_07         1  1063468 61202
    ## + Energystar:size                  1  1063481 61202
    ## + leasing_rate:Gas_Costs           1  1063488 61202
    ## + net:total_dd_07                  1  1063500 61202
    ## - Electricity_Costs:Precipitation  1  1064078 61202
    ## + renovated:total_dd_07            1  1063553 61203
    ## + class_b:Gas_Costs                1  1063576 61203
    ## - class_a:renovated                1  1064127 61203
    ## <none>                                1063869 61203
    ## - LEED:Energystar                  1  1064175 61203
    ## - Gas_Costs:net                    1  1064183 61203
    ## + size:net                         1  1063663 61203
    ## + renovated:class_b                1  1063667 61203
    ## + Energystar:stories               1  1063668 61203
    ## + Energystar:class_b               1  1063670 61203
    ## + leasing_rate:stories             1  1063673 61203
    ## + Energystar:renovated             1  1063712 61204
    ## + renovated:net                    1  1063731 61204
    ## - Precipitation:class_b            1  1064270 61204
    ## + LEED:net                         1  1063743 61204
    ## - class_a:Precipitation            1  1064284 61204
    ## + leasing_rate:renovated           1  1063750 61204
    ## + Energystar:class_a               1  1063755 61204
    ## + renovated:Electricity_Costs      1  1063757 61204
    ## + LEED:leasing_rate                1  1063762 61204
    ## + leasing_rate:class_a             1  1063763 61204
    ## + Energystar:net                   1  1063766 61204
    ## + LEED:Precipitation               1  1063770 61204
    ## + age                              1  1063774 61204
    ## + leasing_rate:class_b             1  1063775 61204
    ## + Energystar:Gas_Costs             1  1063800 61204
    ## + LEED:Gas_Costs                   1  1063805 61204
    ## + stories:net                      1  1063814 61205
    ## + net:Precipitation                1  1063816 61205
    ## + class_a:net                      1  1063823 61205
    ## + LEED:class_a                     1  1063825 61205
    ## + leasing_rate:net                 1  1063825 61205
    ## + size:class_b                     1  1063836 61205
    ## + Energystar:leasing_rate          1  1063837 61205
    ## + LEED:class_b                     1  1063838 61205
    ## + class_b:net                      1  1063839 61205
    ## + LEED:stories                     1  1063841 61205
    ## + net:Electricity_Costs            1  1063858 61205
    ## + amenities                        1  1063863 61205
    ## + Energystar:total_dd_07           1  1063864 61205
    ## + LEED:size                        1  1063866 61205
    ## + LEED:renovated                   1  1063867 61205
    ## + LEED:total_dd_07                 1  1063868 61205
    ## - Precipitation:renovated          1  1064601 61206
    ## - Gas_Costs:total_dd_07            1  1064725 61207
    ## - Precipitation:stories            1  1064786 61208
    ## - Electricity_Costs:stories        1  1065080 61210
    ## - Energystar:Precipitation         1  1065423 61212
    ## - class_a:size                     1  1065889 61216
    ## - size:renovated                   1  1065946 61216
    ## - leasing_rate:Precipitation       1  1066318 61219
    ## - total_dd_07:stories              1  1066440 61220
    ## - Precipitation:Gas_Costs          1  1066582 61221
    ## - renovated:stories                1  1066855 61223
    ## - Electricity_Costs:leasing_rate   1  1067358 61227
    ## - Gas_Costs:renovated              1  1067370 61227
    ## - Gas_Costs:stories                1  1067473 61228
    ## - leasing_rate:size                1  1068480 61235
    ## - Precipitation:total_dd_07        1  1069798 61245
    ## - class_b:total_dd_07              1  1070564 61250
    ## - class_a:total_dd_07              1  1071688 61259
    ## - Electricity_Costs:Gas_Costs      1  1071990 61261
    ## - Electricity_Costs:class_b        1  1074672 61281
    ## - Electricity_Costs:class_a        1  1076222 61292
    ## - Electricity_Costs:size           1  1078614 61310
    ## 
    ## Step:  AIC=61199.77
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + stories + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + Precipitation:size + 
    ##     Gas_Costs:size + leasing_rate:size + Electricity_Costs:class_b + 
    ##     Energystar:Precipitation + Gas_Costs:renovated + class_a:Precipitation + 
    ##     Precipitation:class_b + leasing_rate:Precipitation + Electricity_Costs:leasing_rate + 
    ##     class_a:renovated + Precipitation:total_dd_07 + size:total_dd_07 + 
    ##     class_a:total_dd_07 + class_b:total_dd_07 + Gas_Costs:net + 
    ##     class_a:size + total_dd_07:stories + Gas_Costs:stories + 
    ##     Electricity_Costs:stories + Precipitation:stories + Gas_Costs:total_dd_07 + 
    ##     renovated:stories + size:renovated + Precipitation:renovated + 
    ##     class_a:stories
    ## 
    ##                                   Df Deviance   AIC
    ## + total_dd_07:Electricity_Costs    1  1062595 61197
    ## - Gas_Costs:size                   1  1063187 61198
    ## + Energystar:Electricity_Costs     1  1062676 61198
    ## + renovated:total_dd_07            1  1062721 61198
    ## - class_a:renovated                1  1063289 61199
    ## - Precipitation:size               1  1063289 61199
    ## + class_a:Gas_Costs                1  1062754 61199
    ## + Energystar:size                  1  1062762 61199
    ## + LEED:Electricity_Costs           1  1062774 61199
    ## + size:stories                     1  1062782 61199
    ## - size:total_dd_07                 1  1063332 61199
    ## + leasing_rate:total_dd_07         1  1062796 61199
    ## - class_a:size                     1  1063339 61199
    ## + net:total_dd_07                  1  1062810 61199
    ## + leasing_rate:Gas_Costs           1  1062812 61199
    ## + class_b:Gas_Costs                1  1062838 61199
    ## - Electricity_Costs:Precipitation  1  1063381 61199
    ## + leasing_rate:stories             1  1062902 61200
    ## <none>                                1063172 61200
    ## + Energystar:stories               1  1062924 61200
    ## - LEED:Energystar                  1  1063464 61200
    ## - Gas_Costs:net                    1  1063489 61200
    ## + renovated:class_b                1  1062965 61200
    ## + size:net                         1  1062981 61200
    ## + age                              1  1063013 61201
    ## + Energystar:class_b               1  1063020 61201
    ## - class_a:Precipitation            1  1063563 61201
    ## + Energystar:renovated             1  1063025 61201
    ## + renovated:net                    1  1063027 61201
    ## - Precipitation:class_b            1  1063588 61201
    ## + LEED:net                         1  1063052 61201
    ## + leasing_rate:class_a             1  1063058 61201
    ## + LEED:leasing_rate                1  1063064 61201
    ## + Energystar:net                   1  1063073 61201
    ## + leasing_rate:renovated           1  1063076 61201
    ## + LEED:Precipitation               1  1063079 61201
    ## + Energystar:class_a               1  1063089 61201
    ## + leasing_rate:class_b             1  1063096 61201
    ## + Energystar:Gas_Costs             1  1063106 61201
    ## + LEED:Gas_Costs                   1  1063113 61201
    ## + renovated:Electricity_Costs      1  1063116 61201
    ## + class_a:net                      1  1063120 61201
    ## + net:Precipitation                1  1063121 61201
    ## + leasing_rate:net                 1  1063126 61201
    ## + LEED:class_b                     1  1063128 61201
    ## + stories:net                      1  1063130 61201
    ## + Energystar:leasing_rate          1  1063134 61201
    ## + class_b:net                      1  1063135 61201
    ## + LEED:class_a                     1  1063141 61202
    ## + LEED:stories                     1  1063143 61202
    ## + stories:class_b                  1  1063153 61202
    ## + size:class_b                     1  1063153 61202
    ## + net:Electricity_Costs            1  1063156 61202
    ## + LEED:size                        1  1063167 61202
    ## + LEED:renovated                   1  1063167 61202
    ## + LEED:total_dd_07                 1  1063170 61202
    ## + Energystar:total_dd_07           1  1063170 61202
    ## + amenities                        1  1063171 61202
    ## - class_a:stories                  1  1063869 61203
    ## - Precipitation:renovated          1  1063949 61204
    ## - Gas_Costs:total_dd_07            1  1064057 61204
    ## - Precipitation:stories            1  1064099 61205
    ## - Electricity_Costs:stories        1  1064517 61208
    ## - Energystar:Precipitation         1  1064793 61210
    ## - total_dd_07:stories              1  1065301 61214
    ## - size:renovated                   1  1065355 61214
    ## - leasing_rate:Precipitation       1  1065485 61215
    ## - Precipitation:Gas_Costs          1  1065832 61217
    ## - Gas_Costs:stories                1  1066506 61222
    ## - renovated:stories                1  1066586 61223
    ## - Gas_Costs:renovated              1  1066650 61224
    ## - Electricity_Costs:leasing_rate   1  1066679 61224
    ## - leasing_rate:size                1  1068058 61234
    ## - Precipitation:total_dd_07        1  1069218 61243
    ## - class_b:total_dd_07              1  1069885 61247
    ## - class_a:total_dd_07              1  1070512 61252
    ## - Electricity_Costs:Gas_Costs      1  1071250 61258
    ## - Electricity_Costs:class_b        1  1074052 61278
    ## - Electricity_Costs:class_a        1  1075347 61288
    ## - Electricity_Costs:size           1  1076739 61298
    ## 
    ## Step:  AIC=61197.48
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + stories + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + Precipitation:size + 
    ##     Gas_Costs:size + leasing_rate:size + Electricity_Costs:class_b + 
    ##     Energystar:Precipitation + Gas_Costs:renovated + class_a:Precipitation + 
    ##     Precipitation:class_b + leasing_rate:Precipitation + Electricity_Costs:leasing_rate + 
    ##     class_a:renovated + Precipitation:total_dd_07 + size:total_dd_07 + 
    ##     class_a:total_dd_07 + class_b:total_dd_07 + Gas_Costs:net + 
    ##     class_a:size + total_dd_07:stories + Gas_Costs:stories + 
    ##     Electricity_Costs:stories + Precipitation:stories + Gas_Costs:total_dd_07 + 
    ##     renovated:stories + size:renovated + Precipitation:renovated + 
    ##     class_a:stories + Electricity_Costs:total_dd_07
    ## 
    ##                                   Df Deviance   AIC
    ## - Gas_Costs:size                   1  1062612 61196
    ## + Energystar:Electricity_Costs     1  1062083 61196
    ## + LEED:Electricity_Costs           1  1062157 61196
    ## + class_a:Gas_Costs                1  1062159 61196
    ## - Precipitation:size               1  1062701 61196
    ## + Energystar:size                  1  1062176 61196
    ## - class_a:renovated                1  1062722 61196
    ## + renovated:total_dd_07            1  1062194 61197
    ## - class_a:size                     1  1062735 61197
    ## - size:total_dd_07                 1  1062757 61197
    ## + size:stories                     1  1062221 61197
    ## + leasing_rate:Gas_Costs           1  1062223 61197
    ## + net:total_dd_07                  1  1062229 61197
    ## + leasing_rate:total_dd_07         1  1062230 61197
    ## + class_b:Gas_Costs                1  1062269 61197
    ## <none>                                1062595 61197
    ## - Gas_Costs:net                    1  1062871 61198
    ## + leasing_rate:stories             1  1062342 61198
    ## + Energystar:stories               1  1062346 61198
    ## - LEED:Energystar                  1  1062888 61198
    ## + age                              1  1062382 61198
    ## + renovated:class_b                1  1062383 61198
    ## - Precipitation:class_b            1  1062944 61198
    ## + size:net                         1  1062408 61198
    ## - class_a:Precipitation            1  1062964 61198
    ## + Energystar:renovated             1  1062438 61198
    ## + Energystar:class_b               1  1062445 61198
    ## + renovated:net                    1  1062449 61198
    ## + leasing_rate:class_a             1  1062460 61198
    ## + LEED:leasing_rate                1  1062466 61199
    ## + LEED:net                         1  1062466 61199
    ## + Energystar:net                   1  1062484 61199
    ## + leasing_rate:renovated           1  1062490 61199
    ## - Electricity_Costs:Precipitation  1  1063032 61199
    ## + LEED:Precipitation               1  1062494 61199
    ## + Energystar:class_a               1  1062509 61199
    ## + renovated:Electricity_Costs      1  1062519 61199
    ## + Energystar:Gas_Costs             1  1062521 61199
    ## + leasing_rate:class_b             1  1062522 61199
    ## + LEED:Gas_Costs                   1  1062534 61199
    ## + LEED:class_b                     1  1062543 61199
    ## + net:Precipitation                1  1062544 61199
    ## + class_a:net                      1  1062550 61199
    ## + leasing_rate:net                 1  1062552 61199
    ## + stories:net                      1  1062556 61199
    ## + LEED:class_a                     1  1062556 61199
    ## + LEED:stories                     1  1062559 61199
    ## + class_b:net                      1  1062560 61199
    ## + Energystar:leasing_rate          1  1062564 61199
    ## + size:class_b                     1  1062569 61199
    ## + net:Electricity_Costs            1  1062582 61199
    ## + stories:class_b                  1  1062586 61199
    ## + Energystar:total_dd_07           1  1062589 61199
    ## + LEED:renovated                   1  1062590 61199
    ## + amenities                        1  1062590 61199
    ## + LEED:total_dd_07                 1  1062592 61199
    ## + LEED:size                        1  1062593 61199
    ## - Electricity_Costs:total_dd_07    1  1063172 61200
    ## - class_a:stories                  1  1063423 61202
    ## - Precipitation:renovated          1  1063478 61202
    ## - Gas_Costs:total_dd_07            1  1063572 61203
    ## - Precipitation:stories            1  1063638 61203
    ## - Electricity_Costs:stories        1  1063868 61205
    ## - Energystar:Precipitation         1  1064307 61208
    ## - total_dd_07:stories              1  1064329 61208
    ## - size:renovated                   1  1064778 61212
    ## - leasing_rate:Precipitation       1  1064968 61213
    ## - Precipitation:Gas_Costs          1  1065363 61216
    ## - Gas_Costs:stories                1  1066003 61221
    ## - renovated:stories                1  1066044 61221
    ## - Electricity_Costs:leasing_rate   1  1066164 61222
    ## - Gas_Costs:renovated              1  1066241 61223
    ## - leasing_rate:size                1  1067354 61231
    ## - Precipitation:total_dd_07        1  1069214 61245
    ## - class_b:total_dd_07              1  1069375 61246
    ## - class_a:total_dd_07              1  1069459 61246
    ## - Electricity_Costs:Gas_Costs      1  1070915 61257
    ## - Electricity_Costs:class_b        1  1073393 61275
    ## - Electricity_Costs:class_a        1  1074641 61284
    ## - Electricity_Costs:size           1  1076046 61295
    ## 
    ## Step:  AIC=61195.6
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + stories + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + Precipitation:size + 
    ##     leasing_rate:size + Electricity_Costs:class_b + Energystar:Precipitation + 
    ##     Gas_Costs:renovated + class_a:Precipitation + Precipitation:class_b + 
    ##     leasing_rate:Precipitation + Electricity_Costs:leasing_rate + 
    ##     class_a:renovated + Precipitation:total_dd_07 + size:total_dd_07 + 
    ##     class_a:total_dd_07 + class_b:total_dd_07 + Gas_Costs:net + 
    ##     class_a:size + total_dd_07:stories + Gas_Costs:stories + 
    ##     Electricity_Costs:stories + Precipitation:stories + Gas_Costs:total_dd_07 + 
    ##     renovated:stories + size:renovated + Precipitation:renovated + 
    ##     class_a:stories + Electricity_Costs:total_dd_07
    ## 
    ##                                   Df Deviance   AIC
    ## + Energystar:Electricity_Costs     1  1062098 61194
    ## + class_a:Gas_Costs                1  1062159 61194
    ## + LEED:Electricity_Costs           1  1062171 61194
    ## + Energystar:size                  1  1062188 61194
    ## - class_a:renovated                1  1062740 61195
    ## - class_a:size                     1  1062746 61195
    ## + renovated:total_dd_07            1  1062210 61195
    ## + size:stories                     1  1062221 61195
    ## - size:total_dd_07                 1  1062761 61195
    ## + leasing_rate:Gas_Costs           1  1062235 61195
    ## + net:total_dd_07                  1  1062240 61195
    ## + leasing_rate:total_dd_07         1  1062246 61195
    ## - Precipitation:size               1  1062819 61195
    ## + class_b:Gas_Costs                1  1062308 61195
    ## <none>                                1062612 61196
    ## + leasing_rate:stories             1  1062352 61196
    ## + Energystar:stories               1  1062360 61196
    ## - LEED:Energystar                  1  1062906 61196
    ## - Gas_Costs:net                    1  1062920 61196
    ## + renovated:class_b                1  1062399 61196
    ## + age                              1  1062401 61196
    ## - Precipitation:class_b            1  1062961 61196
    ## + size:net                         1  1062439 61196
    ## - class_a:Precipitation            1  1062978 61196
    ## + Energystar:renovated             1  1062455 61196
    ## + Energystar:class_b               1  1062461 61196
    ## + renovated:net                    1  1062470 61197
    ## + leasing_rate:class_a             1  1062474 61197
    ## + LEED:leasing_rate                1  1062481 61197
    ## + LEED:net                         1  1062482 61197
    ## + Energystar:net                   1  1062501 61197
    ## + leasing_rate:renovated           1  1062507 61197
    ## + LEED:Precipitation               1  1062512 61197
    ## - Electricity_Costs:Precipitation  1  1063055 61197
    ## + Energystar:class_a               1  1062526 61197
    ## + Energystar:Gas_Costs             1  1062531 61197
    ## + renovated:Electricity_Costs      1  1062534 61197
    ## + leasing_rate:class_b             1  1062539 61197
    ## + LEED:Gas_Costs                   1  1062552 61197
    ## + net:Precipitation                1  1062559 61197
    ## + LEED:class_b                     1  1062559 61197
    ## + class_a:net                      1  1062567 61197
    ## + leasing_rate:net                 1  1062570 61197
    ## + LEED:class_a                     1  1062573 61197
    ## + stories:net                      1  1062574 61197
    ## + LEED:stories                     1  1062575 61197
    ## + class_b:net                      1  1062577 61197
    ## + Energystar:leasing_rate          1  1062581 61197
    ## + size:class_b                     1  1062585 61197
    ## + size:Gas_Costs                   1  1062595 61197
    ## + net:Electricity_Costs            1  1062599 61198
    ## + stories:class_b                  1  1062604 61198
    ## + Energystar:total_dd_07           1  1062605 61198
    ## + amenities                        1  1062606 61198
    ## + LEED:renovated                   1  1062607 61198
    ## + LEED:total_dd_07                 1  1062608 61198
    ## + LEED:size                        1  1062609 61198
    ## - Electricity_Costs:total_dd_07    1  1063187 61198
    ## - class_a:stories                  1  1063462 61200
    ## - Precipitation:renovated          1  1063479 61200
    ## - Gas_Costs:total_dd_07            1  1063576 61201
    ## - Precipitation:stories            1  1063693 61202
    ## - Electricity_Costs:stories        1  1063872 61203
    ## - Energystar:Precipitation         1  1064324 61206
    ## - total_dd_07:stories              1  1064368 61207
    ## - size:renovated                   1  1064802 61210
    ## - leasing_rate:Precipitation       1  1064996 61211
    ## - Precipitation:Gas_Costs          1  1065367 61214
    ## - renovated:stories                1  1066069 61219
    ## - Electricity_Costs:leasing_rate   1  1066178 61220
    ## - Gas_Costs:renovated              1  1066331 61221
    ## - leasing_rate:size                1  1067357 61229
    ## - Gas_Costs:stories                1  1068667 61238
    ## - Precipitation:total_dd_07        1  1069215 61243
    ## - class_b:total_dd_07              1  1069398 61244
    ## - class_a:total_dd_07              1  1069460 61244
    ## - Electricity_Costs:Gas_Costs      1  1071663 61261
    ## - Electricity_Costs:class_b        1  1073416 61273
    ## - Electricity_Costs:class_a        1  1074645 61282
    ## - Electricity_Costs:size           1  1076303 61295
    ## 
    ## Step:  AIC=61193.78
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + stories + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + Precipitation:size + 
    ##     leasing_rate:size + Electricity_Costs:class_b + Energystar:Precipitation + 
    ##     Gas_Costs:renovated + class_a:Precipitation + Precipitation:class_b + 
    ##     leasing_rate:Precipitation + Electricity_Costs:leasing_rate + 
    ##     class_a:renovated + Precipitation:total_dd_07 + size:total_dd_07 + 
    ##     class_a:total_dd_07 + class_b:total_dd_07 + Gas_Costs:net + 
    ##     class_a:size + total_dd_07:stories + Gas_Costs:stories + 
    ##     Electricity_Costs:stories + Precipitation:stories + Gas_Costs:total_dd_07 + 
    ##     renovated:stories + size:renovated + Precipitation:renovated + 
    ##     class_a:stories + Electricity_Costs:total_dd_07 + Energystar:Electricity_Costs
    ## 
    ##                                   Df Deviance   AIC
    ## + LEED:Electricity_Costs           1  1061660 61193
    ## + renovated:total_dd_07            1  1061668 61193
    ## - class_a:renovated                1  1062225 61193
    ## - size:total_dd_07                 1  1062237 61193
    ## - class_a:size                     1  1062260 61193
    ## + net:total_dd_07                  1  1061724 61193
    ## + class_a:Gas_Costs                1  1061725 61193
    ## + size:stories                     1  1061730 61193
    ## + leasing_rate:total_dd_07         1  1061733 61193
    ## + leasing_rate:Gas_Costs           1  1061742 61193
    ## + class_b:Gas_Costs                1  1061744 61193
    ## + Energystar:total_dd_07           1  1061749 61193
    ## - Precipitation:size               1  1062316 61193
    ## - LEED:Energystar                  1  1062352 61194
    ## <none>                                1062098 61194
    ## + leasing_rate:stories             1  1061835 61194
    ## - Gas_Costs:net                    1  1062400 61194
    ## + Energystar:size                  1  1061865 61194
    ## + Energystar:Gas_Costs             1  1061870 61194
    ## + age                              1  1061875 61194
    ## + renovated:class_b                1  1061881 61194
    ## - class_a:Precipitation            1  1062428 61194
    ## - Precipitation:class_b            1  1062444 61194
    ## + Energystar:renovated             1  1061919 61194
    ## + size:net                         1  1061925 61195
    ## + leasing_rate:class_a             1  1061956 61195
    ## + LEED:leasing_rate                1  1061957 61195
    ## + LEED:net                         1  1061959 61195
    ## + Energystar:class_b               1  1061964 61195
    ## + Energystar:stories               1  1061970 61195
    ## + renovated:net                    1  1061976 61195
    ## - Electricity_Costs:Precipitation  1  1062526 61195
    ## + leasing_rate:renovated           1  1061995 61195
    ## + LEED:Precipitation               1  1061997 61195
    ## + Energystar:net                   1  1062022 61195
    ## + Energystar:class_a               1  1062026 61195
    ## + leasing_rate:class_b             1  1062027 61195
    ## + LEED:Gas_Costs                   1  1062035 61195
    ## + renovated:Electricity_Costs      1  1062046 61195
    ## + LEED:class_a                     1  1062050 61195
    ## + class_a:net                      1  1062050 61195
    ## + net:Precipitation                1  1062053 61195
    ## + LEED:class_b                     1  1062055 61195
    ## + leasing_rate:net                 1  1062057 61195
    ## + stories:net                      1  1062059 61195
    ## + class_b:net                      1  1062062 61196
    ## + LEED:stories                     1  1062064 61196
    ## + Energystar:leasing_rate          1  1062066 61196
    ## + size:class_b                     1  1062071 61196
    ## - Energystar:Electricity_Costs     1  1062612 61196
    ## + size:Gas_Costs                   1  1062083 61196
    ## + net:Electricity_Costs            1  1062086 61196
    ## + amenities                        1  1062090 61196
    ## + stories:class_b                  1  1062091 61196
    ## + LEED:renovated                   1  1062093 61196
    ## + LEED:size                        1  1062095 61196
    ## + LEED:total_dd_07                 1  1062096 61196
    ## - Electricity_Costs:total_dd_07    1  1062690 61196
    ## - class_a:stories                  1  1062881 61198
    ## - Precipitation:renovated          1  1062980 61198
    ## - Gas_Costs:total_dd_07            1  1063098 61199
    ## - Precipitation:stories            1  1063141 61200
    ## - Electricity_Costs:stories        1  1063235 61200
    ## - total_dd_07:stories              1  1063894 61205
    ## - Energystar:Precipitation         1  1064159 61207
    ## - size:renovated                   1  1064335 61208
    ## - leasing_rate:Precipitation       1  1064505 61210
    ## - Precipitation:Gas_Costs          1  1064842 61212
    ## - renovated:stories                1  1065550 61217
    ## - Electricity_Costs:leasing_rate   1  1065798 61219
    ## - Gas_Costs:renovated              1  1065866 61220
    ## - leasing_rate:size                1  1066909 61227
    ## - Gas_Costs:stories                1  1068179 61237
    ## - Precipitation:total_dd_07        1  1068730 61241
    ## - class_b:total_dd_07              1  1068906 61242
    ## - class_a:total_dd_07              1  1069095 61244
    ## - Electricity_Costs:Gas_Costs      1  1071115 61259
    ## - Electricity_Costs:class_b        1  1072860 61271
    ## - Electricity_Costs:class_a        1  1073709 61278
    ## - Electricity_Costs:size           1  1076007 61294
    ## 
    ## Step:  AIC=61192.53
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + stories + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + Precipitation:size + 
    ##     leasing_rate:size + Electricity_Costs:class_b + Energystar:Precipitation + 
    ##     Gas_Costs:renovated + class_a:Precipitation + Precipitation:class_b + 
    ##     leasing_rate:Precipitation + Electricity_Costs:leasing_rate + 
    ##     class_a:renovated + Precipitation:total_dd_07 + size:total_dd_07 + 
    ##     class_a:total_dd_07 + class_b:total_dd_07 + Gas_Costs:net + 
    ##     class_a:size + total_dd_07:stories + Gas_Costs:stories + 
    ##     Electricity_Costs:stories + Precipitation:stories + Gas_Costs:total_dd_07 + 
    ##     renovated:stories + size:renovated + Precipitation:renovated + 
    ##     class_a:stories + Electricity_Costs:total_dd_07 + Energystar:Electricity_Costs + 
    ##     LEED:Electricity_Costs
    ## 
    ##                                     Df Deviance   AIC
    ## + renovated:total_dd_07              1  1061234 61191
    ## + leasing_rate:Gas_Costs             1  1061236 61191
    ## - class_a:renovated                  1  1061785 61191
    ## + class_b:Gas_Costs                  1  1061250 61191
    ## - size:total_dd_07                   1  1061802 61192
    ## + size:stories                       1  1061272 61192
    ## + leasing_rate:total_dd_07           1  1061274 61192
    ## + net:total_dd_07                    1  1061284 61192
    ## - class_a:size                       1  1061826 61192
    ## + Energystar:total_dd_07             1  1061301 61192
    ## + class_a:Gas_Costs                  1  1061309 61192
    ## - Precipitation:size                 1  1061874 61192
    ## - LEED:Energystar                    1  1061916 61192
    ## - Gas_Costs:net                      1  1061922 61192
    ## <none>                                  1061660 61193
    ## + leasing_rate:stories               1  1061394 61193
    ## + LEED:net                           1  1061420 61193
    ## + Energystar:size                    1  1061431 61193
    ## + Energystar:Gas_Costs               1  1061438 61193
    ## + age                                1  1061440 61193
    ## + renovated:class_b                  1  1061445 61193
    ## - class_a:Precipitation              1  1061995 61193
    ## + LEED:total_dd_07                   1  1061470 61193
    ## - Precipitation:class_b              1  1062008 61193
    ## + Energystar:renovated               1  1061484 61193
    ## + LEED:Energystar:Electricity_Costs  1  1061484 61193
    ## + size:net                           1  1061488 61193
    ## + leasing_rate:class_a               1  1061513 61193
    ## + Energystar:class_b                 1  1061517 61193
    ## - Electricity_Costs:Precipitation    1  1062072 61194
    ## + LEED:Precipitation                 1  1061534 61194
    ## + renovated:net                      1  1061537 61194
    ## + Energystar:stories                 1  1061539 61194
    ## + LEED:leasing_rate                  1  1061545 61194
    ## + leasing_rate:renovated             1  1061558 61194
    ## - LEED:Electricity_Costs             1  1062098 61194
    ## + Energystar:class_a                 1  1061581 61194
    ## + Energystar:net                     1  1061582 61194
    ## + leasing_rate:class_b               1  1061589 61194
    ## + LEED:class_b                       1  1061600 61194
    ## + renovated:Electricity_Costs        1  1061610 61194
    ## + stories:net                        1  1061621 61194
    ## + class_a:net                        1  1061621 61194
    ## + net:Precipitation                  1  1061622 61194
    ## + leasing_rate:net                   1  1061622 61194
    ## + class_b:net                        1  1061631 61194
    ## - Energystar:Electricity_Costs       1  1062171 61194
    ## + Energystar:leasing_rate            1  1061633 61194
    ## + size:class_b                       1  1061634 61194
    ## + LEED:class_a                       1  1061643 61194
    ## + size:Gas_Costs                     1  1061648 61194
    ## + net:Electricity_Costs              1  1061650 61194
    ## + LEED:size                          1  1061652 61194
    ## + amenities                          1  1061653 61194
    ## + stories:class_b                    1  1061653 61194
    ## + LEED:stories                       1  1061654 61194
    ## + LEED:Gas_Costs                     1  1061660 61195
    ## + LEED:renovated                     1  1061660 61195
    ## - Electricity_Costs:total_dd_07      1  1062293 61195
    ## - class_a:stories                    1  1062431 61196
    ## - Precipitation:renovated            1  1062591 61197
    ## - Gas_Costs:total_dd_07              1  1062675 61198
    ## - Electricity_Costs:stories          1  1062728 61198
    ## - Precipitation:stories              1  1062738 61199
    ## - total_dd_07:stories                1  1063444 61204
    ## - Energystar:Precipitation           1  1063720 61206
    ## - size:renovated                     1  1063880 61207
    ## - leasing_rate:Precipitation         1  1064096 61209
    ## - Precipitation:Gas_Costs            1  1064407 61211
    ## - renovated:stories                  1  1065098 61216
    ## - Electricity_Costs:leasing_rate     1  1065431 61219
    ## - Gas_Costs:renovated                1  1065556 61219
    ## - leasing_rate:size                  1  1066320 61225
    ## - Gas_Costs:stories                  1  1067938 61237
    ## - class_b:total_dd_07                1  1068366 61240
    ## - Precipitation:total_dd_07          1  1068474 61241
    ## - class_a:total_dd_07                1  1068619 61242
    ## - Electricity_Costs:Gas_Costs        1  1070550 61256
    ## - Electricity_Costs:class_b          1  1072215 61269
    ## - Electricity_Costs:class_a          1  1073123 61275
    ## - Electricity_Costs:size             1  1075785 61295
    ## 
    ## Step:  AIC=61191.36
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + stories + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + Precipitation:size + 
    ##     leasing_rate:size + Electricity_Costs:class_b + Energystar:Precipitation + 
    ##     Gas_Costs:renovated + class_a:Precipitation + Precipitation:class_b + 
    ##     leasing_rate:Precipitation + Electricity_Costs:leasing_rate + 
    ##     class_a:renovated + Precipitation:total_dd_07 + size:total_dd_07 + 
    ##     class_a:total_dd_07 + class_b:total_dd_07 + Gas_Costs:net + 
    ##     class_a:size + total_dd_07:stories + Gas_Costs:stories + 
    ##     Electricity_Costs:stories + Precipitation:stories + Gas_Costs:total_dd_07 + 
    ##     renovated:stories + size:renovated + Precipitation:renovated + 
    ##     class_a:stories + Electricity_Costs:total_dd_07 + Energystar:Electricity_Costs + 
    ##     LEED:Electricity_Costs + renovated:total_dd_07
    ## 
    ##                                     Df Deviance   AIC
    ## + renovated:Electricity_Costs        1  1060274 61186
    ## - class_a:renovated                  1  1061279 61190
    ## - class_a:size                       1  1061355 61190
    ## + leasing_rate:Gas_Costs             1  1060824 61190
    ## + class_b:Gas_Costs                  1  1060832 61190
    ## - size:total_dd_07                   1  1061401 61191
    ## + size:stories                       1  1060870 61191
    ## + net:total_dd_07                    1  1060872 61191
    ## + leasing_rate:total_dd_07           1  1060879 61191
    ## + class_a:Gas_Costs                  1  1060885 61191
    ## - Precipitation:size                 1  1061439 61191
    ## + Energystar:total_dd_07             1  1060914 61191
    ## - LEED:Energystar                    1  1061488 61191
    ## - Gas_Costs:net                      1  1061502 61191
    ## <none>                                  1061234 61191
    ## + leasing_rate:stories               1  1060971 61191
    ## + LEED:net                           1  1060991 61192
    ## + Energystar:Gas_Costs               1  1061007 61192
    ## + Energystar:size                    1  1061008 61192
    ## + renovated:class_b                  1  1061014 61192
    ## + Energystar:renovated               1  1061035 61192
    ## + LEED:total_dd_07                   1  1061036 61192
    ## + age                                1  1061049 61192
    ## + LEED:Energystar:Electricity_Costs  1  1061056 61192
    ## + size:net                           1  1061065 61192
    ## - Precipitation:class_b              1  1061608 61192
    ## + Energystar:class_b                 1  1061079 61192
    ## + leasing_rate:class_a               1  1061083 61192
    ## + renovated:net                      1  1061085 61192
    ## - class_a:Precipitation              1  1061623 61192
    ## - Electricity_Costs:Precipitation    1  1061643 61192
    ## + LEED:Precipitation                 1  1061108 61192
    ## + Energystar:stories                 1  1061116 61192
    ## - renovated:total_dd_07              1  1061660 61193
    ## + LEED:leasing_rate                  1  1061123 61193
    ## - LEED:Electricity_Costs             1  1061668 61193
    ## - Precipitation:renovated            1  1061679 61193
    ## + Energystar:class_a                 1  1061147 61193
    ## + Energystar:net                     1  1061150 61193
    ## + leasing_rate:renovated             1  1061152 61193
    ## + leasing_rate:class_b               1  1061163 61193
    ## + LEED:class_b                       1  1061174 61193
    ## + leasing_rate:net                   1  1061197 61193
    ## + stories:net                        1  1061197 61193
    ## + class_a:net                        1  1061198 61193
    ## + size:class_b                       1  1061200 61193
    ## + net:Precipitation                  1  1061203 61193
    ## + class_b:net                        1  1061205 61193
    ## + Energystar:leasing_rate            1  1061209 61193
    ## + LEED:class_a                       1  1061216 61193
    ## + size:Gas_Costs                     1  1061222 61193
    ## + net:Electricity_Costs              1  1061223 61193
    ## + LEED:size                          1  1061227 61193
    ## + stories:class_b                    1  1061227 61193
    ## + LEED:stories                       1  1061228 61193
    ## + amenities                          1  1061231 61193
    ## + LEED:renovated                     1  1061232 61193
    ## + LEED:Gas_Costs                     1  1061233 61193
    ## - Energystar:Electricity_Costs       1  1061773 61193
    ## - Electricity_Costs:total_dd_07      1  1061812 61194
    ## - class_a:stories                    1  1062134 61196
    ## - Gas_Costs:total_dd_07              1  1062197 61197
    ## - Electricity_Costs:stories          1  1062322 61197
    ## - Precipitation:stories              1  1062392 61198
    ## - total_dd_07:stories                1  1062785 61201
    ## - size:renovated                     1  1063354 61205
    ## - Energystar:Precipitation           1  1063355 61205
    ## - leasing_rate:Precipitation         1  1063658 61207
    ## - Precipitation:Gas_Costs            1  1063932 61209
    ## - renovated:stories                  1  1064896 61217
    ## - Gas_Costs:renovated                1  1064995 61217
    ## - Electricity_Costs:leasing_rate     1  1065078 61218
    ## - leasing_rate:size                  1  1065879 61224
    ## - Gas_Costs:stories                  1  1067577 61236
    ## - class_a:total_dd_07                1  1067737 61238
    ## - class_b:total_dd_07                1  1067885 61239
    ## - Precipitation:total_dd_07          1  1068051 61240
    ## - Electricity_Costs:Gas_Costs        1  1070238 61256
    ## - Electricity_Costs:class_b          1  1071864 61268
    ## - Electricity_Costs:class_a          1  1072705 61274
    ## - Electricity_Costs:size             1  1075238 61293
    ## 
    ## Step:  AIC=61186.22
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + stories + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + Precipitation:size + 
    ##     leasing_rate:size + Electricity_Costs:class_b + Energystar:Precipitation + 
    ##     Gas_Costs:renovated + class_a:Precipitation + Precipitation:class_b + 
    ##     leasing_rate:Precipitation + Electricity_Costs:leasing_rate + 
    ##     class_a:renovated + Precipitation:total_dd_07 + size:total_dd_07 + 
    ##     class_a:total_dd_07 + class_b:total_dd_07 + Gas_Costs:net + 
    ##     class_a:size + total_dd_07:stories + Gas_Costs:stories + 
    ##     Electricity_Costs:stories + Precipitation:stories + Gas_Costs:total_dd_07 + 
    ##     renovated:stories + size:renovated + Precipitation:renovated + 
    ##     class_a:stories + Electricity_Costs:total_dd_07 + Energystar:Electricity_Costs + 
    ##     LEED:Electricity_Costs + renovated:total_dd_07 + Electricity_Costs:renovated
    ## 
    ##                                     Df Deviance   AIC
    ## - class_a:renovated                  1  1060301 61184
    ## - class_a:size                       1  1060392 61185
    ## + class_b:Gas_Costs                  1  1059862 61185
    ## + leasing_rate:Gas_Costs             1  1059883 61185
    ## - Precipitation:size                 1  1060439 61185
    ## - size:total_dd_07                   1  1060443 61185
    ## + net:total_dd_07                    1  1059921 61186
    ## + size:stories                       1  1059926 61186
    ## + leasing_rate:total_dd_07           1  1059930 61186
    ## + class_a:Gas_Costs                  1  1059972 61186
    ## - LEED:Energystar                    1  1060540 61186
    ## <none>                                  1060274 61186
    ## + leasing_rate:stories               1  1060012 61186
    ## + Energystar:total_dd_07             1  1060025 61186
    ## - Precipitation:class_b              1  1060572 61186
    ## + LEED:net                           1  1060039 61186
    ## - Gas_Costs:net                      1  1060590 61187
    ## + renovated:class_b                  1  1060058 61187
    ## + Energystar:size                    1  1060062 61187
    ## + Energystar:Gas_Costs               1  1060064 61187
    ## + Energystar:renovated               1  1060082 61187
    ## - class_a:Precipitation              1  1060631 61187
    ## + age                                1  1060097 61187
    ## + size:net                           1  1060099 61187
    ## + LEED:total_dd_07                   1  1060103 61187
    ## - Electricity_Costs:Precipitation    1  1060643 61187
    ## + LEED:Energystar:Electricity_Costs  1  1060109 61187
    ## + leasing_rate:class_a               1  1060112 61187
    ## + Energystar:class_b                 1  1060117 61187
    ## + renovated:net                      1  1060121 61187
    ## + LEED:Precipitation                 1  1060146 61187
    ## - LEED:Electricity_Costs             1  1060695 61187
    ## + LEED:leasing_rate                  1  1060161 61187
    ## + Energystar:stories                 1  1060164 61187
    ## - Energystar:Electricity_Costs       1  1060710 61187
    ## + leasing_rate:renovated             1  1060184 61188
    ## + Energystar:class_a                 1  1060185 61188
    ## + Energystar:net                     1  1060197 61188
    ## + leasing_rate:class_b               1  1060198 61188
    ## + LEED:class_b                       1  1060211 61188
    ## + class_a:net                        1  1060234 61188
    ## + stories:net                        1  1060234 61188
    ## + size:class_b                       1  1060239 61188
    ## + class_b:net                        1  1060242 61188
    ## + leasing_rate:net                   1  1060243 61188
    ## + Energystar:leasing_rate            1  1060247 61188
    ## + net:Precipitation                  1  1060248 61188
    ## + net:Electricity_Costs              1  1060254 61188
    ## + LEED:class_a                       1  1060257 61188
    ## + size:Gas_Costs                     1  1060267 61188
    ## + stories:class_b                    1  1060268 61188
    ## + LEED:stories                       1  1060268 61188
    ## + LEED:size                          1  1060268 61188
    ## + amenities                          1  1060271 61188
    ## + LEED:renovated                     1  1060274 61188
    ## + LEED:Gas_Costs                     1  1060274 61188
    ## - Electricity_Costs:total_dd_07      1  1060884 61189
    ## - class_a:stories                    1  1061099 61190
    ## - Precipitation:renovated            1  1061101 61190
    ## - Electricity_Costs:renovated        1  1061234 61191
    ## - Gas_Costs:total_dd_07              1  1061328 61192
    ## - Precipitation:stories              1  1061506 61193
    ## - Electricity_Costs:stories          1  1061506 61193
    ## - renovated:total_dd_07              1  1061610 61194
    ## - total_dd_07:stories                1  1061899 61196
    ## - size:renovated                     1  1062167 61198
    ## - Energystar:Precipitation           1  1062445 61200
    ## - leasing_rate:Precipitation         1  1062764 61203
    ## - Precipitation:Gas_Costs            1  1063233 61206
    ## - renovated:stories                  1  1063843 61211
    ## - Electricity_Costs:leasing_rate     1  1064020 61212
    ## - leasing_rate:size                  1  1064753 61217
    ## - Gas_Costs:renovated                1  1064988 61219
    ## - Gas_Costs:stories                  1  1066713 61232
    ## - class_a:total_dd_07                1  1066862 61233
    ## - class_b:total_dd_07                1  1067160 61235
    ## - Precipitation:total_dd_07          1  1067792 61240
    ## - Electricity_Costs:Gas_Costs        1  1068966 61249
    ## - Electricity_Costs:class_b          1  1071244 61265
    ## - Electricity_Costs:class_a          1  1071701 61269
    ## - Electricity_Costs:size             1  1073984 61286
    ## 
    ## Step:  AIC=61184.42
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + stories + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + Precipitation:size + 
    ##     leasing_rate:size + Electricity_Costs:class_b + Energystar:Precipitation + 
    ##     Gas_Costs:renovated + class_a:Precipitation + Precipitation:class_b + 
    ##     leasing_rate:Precipitation + Electricity_Costs:leasing_rate + 
    ##     Precipitation:total_dd_07 + size:total_dd_07 + class_a:total_dd_07 + 
    ##     class_b:total_dd_07 + Gas_Costs:net + class_a:size + total_dd_07:stories + 
    ##     Gas_Costs:stories + Electricity_Costs:stories + Precipitation:stories + 
    ##     Gas_Costs:total_dd_07 + renovated:stories + size:renovated + 
    ##     Precipitation:renovated + class_a:stories + Electricity_Costs:total_dd_07 + 
    ##     Energystar:Electricity_Costs + LEED:Electricity_Costs + renovated:total_dd_07 + 
    ##     Electricity_Costs:renovated
    ## 
    ##                                     Df Deviance   AIC
    ## - class_a:size                       1  1060411 61183
    ## + class_b:Gas_Costs                  1  1059885 61183
    ## + leasing_rate:Gas_Costs             1  1059908 61183
    ## - Precipitation:size                 1  1060458 61184
    ## - size:total_dd_07                   1  1060481 61184
    ## + net:total_dd_07                    1  1059948 61184
    ## + leasing_rate:total_dd_07           1  1059956 61184
    ## + size:stories                       1  1059977 61184
    ## + class_a:Gas_Costs                  1  1060001 61184
    ## - LEED:Energystar                    1  1060562 61184
    ## <none>                                  1060301 61184
    ## + leasing_rate:stories               1  1060042 61184
    ## + Energystar:total_dd_07             1  1060052 61185
    ## - Precipitation:class_b              1  1060596 61185
    ## + LEED:net                           1  1060065 61185
    ## - Gas_Costs:net                      1  1060613 61185
    ## + Energystar:Gas_Costs               1  1060090 61185
    ## + Energystar:size                    1  1060091 61185
    ## - class_a:Precipitation              1  1060659 61185
    ## - Electricity_Costs:Precipitation    1  1060660 61185
    ## + age                                1  1060124 61185
    ## + size:net                           1  1060125 61185
    ## + LEED:total_dd_07                   1  1060126 61185
    ## + LEED:Energystar:Electricity_Costs  1  1060131 61185
    ## + Energystar:class_b                 1  1060138 61185
    ## + leasing_rate:class_a               1  1060139 61185
    ## + renovated:net                      1  1060140 61185
    ## + Energystar:renovated               1  1060147 61185
    ## + LEED:Precipitation                 1  1060173 61185
    ## - LEED:Electricity_Costs             1  1060722 61186
    ## + LEED:leasing_rate                  1  1060189 61186
    ## + Energystar:stories                 1  1060193 61186
    ## - Energystar:Electricity_Costs       1  1060737 61186
    ## + leasing_rate:renovated             1  1060201 61186
    ## + Energystar:class_a                 1  1060206 61186
    ## + Energystar:net                     1  1060223 61186
    ## + leasing_rate:class_b               1  1060225 61186
    ## + LEED:class_b                       1  1060237 61186
    ## + renovated:class_b                  1  1060253 61186
    ## + class_a:net                        1  1060260 61186
    ## + stories:net                        1  1060261 61186
    ## + size:class_b                       1  1060262 61186
    ## + class_b:net                        1  1060268 61186
    ## + leasing_rate:net                   1  1060269 61186
    ## + Energystar:leasing_rate            1  1060274 61186
    ## + renovated:class_a                  1  1060274 61186
    ## + net:Precipitation                  1  1060275 61186
    ## + net:Electricity_Costs              1  1060279 61186
    ## + LEED:class_a                       1  1060283 61186
    ## + size:Gas_Costs                     1  1060293 61186
    ## + LEED:size                          1  1060294 61186
    ## + stories:class_b                    1  1060294 61186
    ## + LEED:stories                       1  1060295 61186
    ## + amenities                          1  1060298 61186
    ## + LEED:renovated                     1  1060300 61186
    ## + LEED:Gas_Costs                     1  1060300 61186
    ## - Electricity_Costs:total_dd_07      1  1060904 61187
    ## - Precipitation:renovated            1  1061128 61189
    ## - class_a:stories                    1  1061222 61189
    ## - Electricity_Costs:renovated        1  1061279 61190
    ## - Gas_Costs:total_dd_07              1  1061340 61190
    ## - Electricity_Costs:stories          1  1061538 61192
    ## - Precipitation:stories              1  1061553 61192
    ## - renovated:total_dd_07              1  1061751 61193
    ## - total_dd_07:stories                1  1061902 61194
    ## - size:renovated                     1  1062169 61196
    ## - Energystar:Precipitation           1  1062467 61199
    ## - leasing_rate:Precipitation         1  1062786 61201
    ## - Precipitation:Gas_Costs            1  1063285 61205
    ## - Electricity_Costs:leasing_rate     1  1064038 61210
    ## - renovated:stories                  1  1064182 61211
    ## - leasing_rate:size                  1  1064763 61216
    ## - Gas_Costs:renovated                1  1065110 61218
    ## - Gas_Costs:stories                  1  1066753 61230
    ## - class_a:total_dd_07                1  1066864 61231
    ## - class_b:total_dd_07                1  1067212 61234
    ## - Precipitation:total_dd_07          1  1067804 61238
    ## - Electricity_Costs:Gas_Costs        1  1069002 61247
    ## - Electricity_Costs:class_b          1  1071278 61264
    ## - Electricity_Costs:class_a          1  1071701 61267
    ## - Electricity_Costs:size             1  1073996 61284
    ## 
    ## Step:  AIC=61183.24
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + stories + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + Precipitation:size + 
    ##     leasing_rate:size + Electricity_Costs:class_b + Energystar:Precipitation + 
    ##     Gas_Costs:renovated + class_a:Precipitation + Precipitation:class_b + 
    ##     leasing_rate:Precipitation + Electricity_Costs:leasing_rate + 
    ##     Precipitation:total_dd_07 + size:total_dd_07 + class_a:total_dd_07 + 
    ##     class_b:total_dd_07 + Gas_Costs:net + total_dd_07:stories + 
    ##     Gas_Costs:stories + Electricity_Costs:stories + Precipitation:stories + 
    ##     Gas_Costs:total_dd_07 + renovated:stories + size:renovated + 
    ##     Precipitation:renovated + class_a:stories + Electricity_Costs:total_dd_07 + 
    ##     Energystar:Electricity_Costs + LEED:Electricity_Costs + renovated:total_dd_07 + 
    ##     Electricity_Costs:renovated
    ## 
    ##                                     Df Deviance   AIC
    ## + size:stories                       1  1059981 61182
    ## + class_b:Gas_Costs                  1  1060014 61182
    ## - Precipitation:size                 1  1060557 61182
    ## + leasing_rate:Gas_Costs             1  1060030 61182
    ## + leasing_rate:total_dd_07           1  1060072 61183
    ## + net:total_dd_07                    1  1060076 61183
    ## - size:total_dd_07                   1  1060639 61183
    ## + class_a:Gas_Costs                  1  1060107 61183
    ## - LEED:Energystar                    1  1060673 61183
    ## <none>                                  1060411 61183
    ## + leasing_rate:stories               1  1060158 61183
    ## + Energystar:total_dd_07             1  1060163 61183
    ## - Precipitation:class_b              1  1060704 61183
    ## + LEED:net                           1  1060175 61183
    ## - Gas_Costs:net                      1  1060729 61184
    ## + Energystar:Gas_Costs               1  1060206 61184
    ## + size:net                           1  1060211 61184
    ## + age                                1  1060213 61184
    ## + Energystar:size                    1  1060221 61184
    ## - class_a:Precipitation              1  1060769 61184
    ## + LEED:total_dd_07                   1  1060235 61184
    ## + renovated:net                      1  1060241 61184
    ## + LEED:Energystar:Electricity_Costs  1  1060245 61184
    ## - Electricity_Costs:Precipitation    1  1060784 61184
    ## + Energystar:renovated               1  1060249 61184
    ## + leasing_rate:class_a               1  1060252 61184
    ## + Energystar:class_b                 1  1060267 61184
    ## + LEED:Precipitation                 1  1060283 61184
    ## - Energystar:Electricity_Costs       1  1060827 61184
    ## - LEED:Electricity_Costs             1  1060830 61184
    ## + LEED:leasing_rate                  1  1060298 61184
    ## + size:class_a                       1  1060301 61184
    ## + Energystar:stories                 1  1060307 61184
    ## + leasing_rate:renovated             1  1060321 61185
    ## + Energystar:class_a                 1  1060331 61185
    ## + Energystar:net                     1  1060333 61185
    ## + leasing_rate:class_b               1  1060334 61185
    ## + LEED:class_b                       1  1060342 61185
    ## + renovated:class_b                  1  1060354 61185
    ## + size:class_b                       1  1060360 61185
    ## + stories:net                        1  1060363 61185
    ## + class_a:net                        1  1060365 61185
    ## + class_b:net                        1  1060372 61185
    ## + leasing_rate:net                   1  1060377 61185
    ## + Energystar:leasing_rate            1  1060385 61185
    ## + net:Electricity_Costs              1  1060386 61185
    ## + net:Precipitation                  1  1060388 61185
    ## + renovated:class_a                  1  1060392 61185
    ## + LEED:class_a                       1  1060395 61185
    ## + LEED:size                          1  1060401 61185
    ## + stories:class_b                    1  1060402 61185
    ## + amenities                          1  1060406 61185
    ## + LEED:stories                       1  1060406 61185
    ## + size:Gas_Costs                     1  1060407 61185
    ## + LEED:renovated                     1  1060410 61185
    ## + LEED:Gas_Costs                     1  1060411 61185
    ## - Electricity_Costs:total_dd_07      1  1061037 61186
    ## - Precipitation:renovated            1  1061220 61187
    ## - Electricity_Costs:renovated        1  1061391 61189
    ## - Gas_Costs:total_dd_07              1  1061458 61189
    ## - Precipitation:stories              1  1061676 61191
    ## - Electricity_Costs:stories          1  1061745 61191
    ## - renovated:total_dd_07              1  1061908 61192
    ## - total_dd_07:stories                1  1061917 61192
    ## - size:renovated                     1  1062260 61195
    ## - Energystar:Precipitation           1  1062593 61197
    ## - leasing_rate:Precipitation         1  1062887 61200
    ## - class_a:stories                    1  1063316 61203
    ## - Precipitation:Gas_Costs            1  1063428 61204
    ## - Electricity_Costs:leasing_rate     1  1064118 61209
    ## - renovated:stories                  1  1064323 61210
    ## - leasing_rate:size                  1  1064822 61214
    ## - Gas_Costs:renovated                1  1065204 61217
    ## - Gas_Costs:stories                  1  1066799 61229
    ## - class_a:total_dd_07                1  1066930 61230
    ## - class_b:total_dd_07                1  1067269 61232
    ## - Precipitation:total_dd_07          1  1067952 61237
    ## - Electricity_Costs:Gas_Costs        1  1069052 61245
    ## - Electricity_Costs:class_b          1  1071377 61262
    ## - Electricity_Costs:class_a          1  1071727 61265
    ## - Electricity_Costs:size             1  1074229 61283
    ## 
    ## Step:  AIC=61182.04
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + stories + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + Precipitation:size + 
    ##     leasing_rate:size + Electricity_Costs:class_b + Energystar:Precipitation + 
    ##     Gas_Costs:renovated + class_a:Precipitation + Precipitation:class_b + 
    ##     leasing_rate:Precipitation + Electricity_Costs:leasing_rate + 
    ##     Precipitation:total_dd_07 + size:total_dd_07 + class_a:total_dd_07 + 
    ##     class_b:total_dd_07 + Gas_Costs:net + total_dd_07:stories + 
    ##     Gas_Costs:stories + Electricity_Costs:stories + Precipitation:stories + 
    ##     Gas_Costs:total_dd_07 + renovated:stories + size:renovated + 
    ##     Precipitation:renovated + class_a:stories + Electricity_Costs:total_dd_07 + 
    ##     Energystar:Electricity_Costs + LEED:Electricity_Costs + renovated:total_dd_07 + 
    ##     Electricity_Costs:renovated + size:stories
    ## 
    ##                                     Df Deviance   AIC
    ## + leasing_rate:Gas_Costs             1  1059541 61181
    ## - Precipitation:size                 1  1060121 61181
    ## + class_a:Gas_Costs                  1  1059606 61181
    ## + class_b:Gas_Costs                  1  1059609 61181
    ## + leasing_rate:total_dd_07           1  1059652 61182
    ## + net:total_dd_07                    1  1059655 61182
    ## - LEED:Energystar                    1  1060232 61182
    ## - size:total_dd_07                   1  1060237 61182
    ## + leasing_rate:stories               1  1059709 61182
    ## <none>                                  1059981 61182
    ## - Precipitation:class_b              1  1060273 61182
    ## + Energystar:total_dd_07             1  1059747 61182
    ## + LEED:net                           1  1059752 61182
    ## - Gas_Costs:net                      1  1060296 61182
    ## + Energystar:Gas_Costs               1  1059772 61182
    ## + Energystar:size                    1  1059774 61182
    ## + size:net                           1  1059777 61183
    ## - Electricity_Costs:Precipitation    1  1060318 61183
    ## + age                                1  1059804 61183
    ## + renovated:net                      1  1059806 61183
    ## + LEED:total_dd_07                   1  1059814 61183
    ## + LEED:Energystar:Electricity_Costs  1  1059814 61183
    ## - class_a:Precipitation              1  1060352 61183
    ## + Energystar:class_b                 1  1059823 61183
    ## + leasing_rate:class_a               1  1059829 61183
    ## + Energystar:renovated               1  1059835 61183
    ## - Energystar:Electricity_Costs       1  1060396 61183
    ## + LEED:Precipitation                 1  1059863 61183
    ## + Energystar:stories                 1  1059867 61183
    ## + leasing_rate:renovated             1  1059870 61183
    ## - size:stories                       1  1060411 61183
    ## + LEED:leasing_rate                  1  1059874 61183
    ## - LEED:Electricity_Costs             1  1060422 61183
    ## + Energystar:class_a                 1  1059891 61183
    ## + Energystar:net                     1  1059893 61183
    ## + leasing_rate:class_b               1  1059903 61183
    ## + LEED:class_b                       1  1059916 61184
    ## + renovated:class_a                  1  1059931 61184
    ## + stories:net                        1  1059931 61184
    ## + class_a:net                        1  1059934 61184
    ## + leasing_rate:net                   1  1059943 61184
    ## + class_b:net                        1  1059943 61184
    ## + renovated:class_b                  1  1059947 61184
    ## + Energystar:leasing_rate            1  1059957 61184
    ## + net:Precipitation                  1  1059958 61184
    ## + net:Electricity_Costs              1  1059958 61184
    ## + LEED:class_a                       1  1059965 61184
    ## + stories:class_b                    1  1059969 61184
    ## + LEED:size                          1  1059972 61184
    ## + LEED:stories                       1  1059977 61184
    ## + size:class_a                       1  1059977 61184
    ## + LEED:renovated                     1  1059980 61184
    ## + size:Gas_Costs                     1  1059981 61184
    ## + size:class_b                       1  1059981 61184
    ## + amenities                          1  1059981 61184
    ## + LEED:Gas_Costs                     1  1059981 61184
    ## - Electricity_Costs:total_dd_07      1  1060574 61184
    ## - Precipitation:renovated            1  1060802 61186
    ## - Electricity_Costs:renovated        1  1060952 61187
    ## - Gas_Costs:total_dd_07              1  1061098 61188
    ## - Electricity_Costs:stories          1  1061324 61190
    ## - Precipitation:stories              1  1061381 61190
    ## - renovated:total_dd_07              1  1061427 61191
    ## - total_dd_07:stories                1  1061589 61192
    ## - class_a:stories                    1  1061598 61192
    ## - size:renovated                     1  1061931 61195
    ## - Energystar:Precipitation           1  1062118 61196
    ## - leasing_rate:Precipitation         1  1062403 61198
    ## - Precipitation:Gas_Costs            1  1062914 61202
    ## - Electricity_Costs:leasing_rate     1  1063723 61208
    ## - renovated:stories                  1  1063793 61208
    ## - leasing_rate:size                  1  1064357 61213
    ## - Gas_Costs:renovated                1  1064799 61216
    ## - class_a:total_dd_07                1  1066701 61230
    ## - Gas_Costs:stories                  1  1066709 61230
    ## - class_b:total_dd_07                1  1066906 61231
    ## - Precipitation:total_dd_07          1  1067646 61237
    ## - Electricity_Costs:Gas_Costs        1  1068544 61244
    ## - Electricity_Costs:class_b          1  1070864 61261
    ## - Electricity_Costs:class_a          1  1071239 61263
    ## - Electricity_Costs:size             1  1074029 61284
    ## 
    ## Step:  AIC=61180.76
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + stories + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + Precipitation:size + 
    ##     leasing_rate:size + Electricity_Costs:class_b + Energystar:Precipitation + 
    ##     Gas_Costs:renovated + class_a:Precipitation + Precipitation:class_b + 
    ##     leasing_rate:Precipitation + Electricity_Costs:leasing_rate + 
    ##     Precipitation:total_dd_07 + size:total_dd_07 + class_a:total_dd_07 + 
    ##     class_b:total_dd_07 + Gas_Costs:net + total_dd_07:stories + 
    ##     Gas_Costs:stories + Electricity_Costs:stories + Precipitation:stories + 
    ##     Gas_Costs:total_dd_07 + renovated:stories + size:renovated + 
    ##     Precipitation:renovated + class_a:stories + Electricity_Costs:total_dd_07 + 
    ##     Energystar:Electricity_Costs + LEED:Electricity_Costs + renovated:total_dd_07 + 
    ##     Electricity_Costs:renovated + size:stories + leasing_rate:Gas_Costs
    ## 
    ##                                     Df Deviance   AIC
    ## - Precipitation:size                 1  1059698 61180
    ## + net:total_dd_07                    1  1059200 61180
    ## + class_a:Gas_Costs                  1  1059238 61181
    ## - size:total_dd_07                   1  1059786 61181
    ## - Gas_Costs:net                      1  1059786 61181
    ## + leasing_rate:stories               1  1059249 61181
    ## - LEED:Energystar                    1  1059802 61181
    ## <none>                                  1059541 61181
    ## + LEED:net                           1  1059302 61181
    ## + Energystar:total_dd_07             1  1059312 61181
    ## + Energystar:size                    1  1059328 61181
    ## + class_b:Gas_Costs                  1  1059334 61181
    ## + size:net                           1  1059341 61181
    ## + LEED:total_dd_07                   1  1059349 61181
    ## + LEED:Energystar:Electricity_Costs  1  1059360 61181
    ## - Precipitation:class_b              1  1059898 61181
    ## - Electricity_Costs:Precipitation    1  1059899 61181
    ## + renovated:net                      1  1059368 61181
    ## + age                                1  1059371 61181
    ## + Energystar:Gas_Costs               1  1059372 61182
    ## + Energystar:class_b                 1  1059377 61182
    ## + leasing_rate:class_a               1  1059381 61182
    ## + Energystar:renovated               1  1059392 61182
    ## - Energystar:Electricity_Costs       1  1059933 61182
    ## + leasing_rate:total_dd_07           1  1059404 61182
    ## - class_a:Precipitation              1  1059946 61182
    ## + LEED:Precipitation                 1  1059413 61182
    ## + Energystar:stories                 1  1059424 61182
    ## + LEED:leasing_rate                  1  1059435 61182
    ## - leasing_rate:Gas_Costs             1  1059981 61182
    ## + Energystar:class_a                 1  1059447 61182
    ## + Energystar:net                     1  1059449 61182
    ## + leasing_rate:class_b               1  1059464 61182
    ## + leasing_rate:renovated             1  1059470 61182
    ## + LEED:class_b                       1  1059480 61182
    ## + renovated:class_a                  1  1059490 61182
    ## - size:stories                       1  1060030 61182
    ## + stories:net                        1  1059494 61182
    ## - leasing_rate:Precipitation         1  1060041 61182
    ## + class_a:net                        1  1059504 61182
    ## + leasing_rate:net                   1  1059507 61183
    ## + renovated:class_b                  1  1059508 61183
    ## + class_b:net                        1  1059510 61183
    ## - LEED:Electricity_Costs             1  1060053 61183
    ## + Energystar:leasing_rate            1  1059518 61183
    ## + net:Electricity_Costs              1  1059521 61183
    ## + LEED:class_a                       1  1059521 61183
    ## + net:Precipitation                  1  1059523 61183
    ## + stories:class_b                    1  1059523 61183
    ## + LEED:size                          1  1059533 61183
    ## + LEED:stories                       1  1059536 61183
    ## + size:class_a                       1  1059537 61183
    ## + LEED:Gas_Costs                     1  1059538 61183
    ## + size:Gas_Costs                     1  1059538 61183
    ## + LEED:renovated                     1  1059540 61183
    ## + amenities                          1  1059540 61183
    ## + size:class_b                       1  1059540 61183
    ## - Electricity_Costs:total_dd_07      1  1060149 61183
    ## - Precipitation:renovated            1  1060404 61185
    ## - Electricity_Costs:renovated        1  1060491 61186
    ## - Gas_Costs:total_dd_07              1  1060639 61187
    ## - Electricity_Costs:stories          1  1060943 61189
    ## - renovated:total_dd_07              1  1060946 61189
    ## - class_a:stories                    1  1061095 61190
    ## - Precipitation:stories              1  1061149 61191
    ## - total_dd_07:stories                1  1061167 61191
    ## - size:renovated                     1  1061512 61193
    ## - Electricity_Costs:leasing_rate     1  1061629 61194
    ## - Energystar:Precipitation           1  1061649 61194
    ## - Precipitation:Gas_Costs            1  1062522 61201
    ## - renovated:stories                  1  1063374 61207
    ## - leasing_rate:size                  1  1064031 61212
    ## - Gas_Costs:renovated                1  1064575 61216
    ## - Gas_Costs:stories                  1  1066388 61230
    ## - class_a:total_dd_07                1  1066676 61232
    ## - class_b:total_dd_07                1  1066896 61233
    ## - Precipitation:total_dd_07          1  1067316 61236
    ## - Electricity_Costs:Gas_Costs        1  1068005 61242
    ## - Electricity_Costs:class_b          1  1070658 61261
    ## - Electricity_Costs:class_a          1  1071167 61265
    ## - Electricity_Costs:size             1  1073570 61283
    ## 
    ## Step:  AIC=61179.93
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + stories + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + leasing_rate:size + 
    ##     Electricity_Costs:class_b + Energystar:Precipitation + Gas_Costs:renovated + 
    ##     class_a:Precipitation + Precipitation:class_b + leasing_rate:Precipitation + 
    ##     Electricity_Costs:leasing_rate + Precipitation:total_dd_07 + 
    ##     size:total_dd_07 + class_a:total_dd_07 + class_b:total_dd_07 + 
    ##     Gas_Costs:net + total_dd_07:stories + Gas_Costs:stories + 
    ##     Electricity_Costs:stories + Precipitation:stories + Gas_Costs:total_dd_07 + 
    ##     renovated:stories + size:renovated + Precipitation:renovated + 
    ##     class_a:stories + Electricity_Costs:total_dd_07 + Energystar:Electricity_Costs + 
    ##     LEED:Electricity_Costs + renovated:total_dd_07 + Electricity_Costs:renovated + 
    ##     size:stories + leasing_rate:Gas_Costs
    ## 
    ##                                     Df Deviance   AIC
    ## + class_a:Gas_Costs                  1  1059390 61180
    ## + net:total_dd_07                    1  1059397 61180
    ## + leasing_rate:stories               1  1059398 61180
    ## - LEED:Energystar                    1  1059950 61180
    ## - Gas_Costs:net                      1  1059965 61180
    ## <none>                                  1059698 61180
    ## + Energystar:total_dd_07             1  1059431 61180
    ## + size:net                           1  1059450 61180
    ## + LEED:net                           1  1059456 61180
    ## + class_b:Gas_Costs                  1  1059497 61180
    ## + Energystar:size                    1  1059503 61180
    ## - Precipitation:class_b              1  1060041 61180
    ## + renovated:net                      1  1059506 61180
    ## + LEED:total_dd_07                   1  1059508 61181
    ## - Electricity_Costs:Precipitation    1  1060046 61181
    ## + LEED:Energystar:Electricity_Costs  1  1059514 61181
    ## - class_a:Precipitation              1  1060058 61181
    ## + leasing_rate:class_a               1  1059529 61181
    ## + Energystar:Gas_Costs               1  1059534 61181
    ## + Energystar:class_b                 1  1059535 61181
    ## + age                                1  1059536 61181
    ## + size:Precipitation                 1  1059541 61181
    ## - Energystar:Electricity_Costs       1  1060081 61181
    ## + Energystar:renovated               1  1059549 61181
    ## + leasing_rate:total_dd_07           1  1059571 61181
    ## + LEED:Precipitation                 1  1059582 61181
    ## - leasing_rate:Gas_Costs             1  1060121 61181
    ## + Energystar:stories                 1  1059587 61181
    ## + LEED:leasing_rate                  1  1059590 61181
    ## + Energystar:class_a                 1  1059606 61181
    ## + Energystar:net                     1  1059607 61181
    ## + leasing_rate:class_b               1  1059619 61181
    ## + leasing_rate:renovated             1  1059623 61181
    ## + stories:net                        1  1059630 61181
    ## + LEED:class_b                       1  1059637 61181
    ## + class_a:net                        1  1059654 61182
    ## + renovated:class_a                  1  1059656 61182
    ## - size:stories                       1  1060193 61182
    ## + renovated:class_b                  1  1059660 61182
    ## + class_b:net                        1  1059661 61182
    ## + leasing_rate:net                   1  1059663 61182
    ## + size:Gas_Costs                     1  1059673 61182
    ## + Energystar:leasing_rate            1  1059675 61182
    ## + net:Electricity_Costs              1  1059676 61182
    ## - LEED:Electricity_Costs             1  1060213 61182
    ## + LEED:class_a                       1  1059679 61182
    ## + stories:class_b                    1  1059680 61182
    ## + net:Precipitation                  1  1059688 61182
    ## + LEED:size                          1  1059691 61182
    ## - leasing_rate:Precipitation         1  1060229 61182
    ## + LEED:stories                       1  1059693 61182
    ## + LEED:Gas_Costs                     1  1059696 61182
    ## + size:class_a                       1  1059696 61182
    ## + size:class_b                       1  1059697 61182
    ## + LEED:renovated                     1  1059697 61182
    ## + amenities                          1  1059698 61182
    ## - Electricity_Costs:total_dd_07      1  1060320 61183
    ## - size:total_dd_07                   1  1060526 61184
    ## - Precipitation:renovated            1  1060548 61184
    ## - Electricity_Costs:renovated        1  1060685 61185
    ## - Gas_Costs:total_dd_07              1  1060766 61186
    ## - Electricity_Costs:stories          1  1060976 61187
    ## - renovated:total_dd_07              1  1061135 61189
    ## - total_dd_07:stories                1  1061189 61189
    ## - class_a:stories                    1  1061258 61190
    ## - size:renovated                     1  1061578 61192
    ## - Energystar:Precipitation           1  1061702 61193
    ## - Electricity_Costs:leasing_rate     1  1061801 61194
    ## - Precipitation:Gas_Costs            1  1062558 61199
    ## - renovated:stories                  1  1063443 61206
    ## - leasing_rate:size                  1  1064176 61211
    ## - Gas_Costs:renovated                1  1064748 61215
    ## - Precipitation:stories              1  1065080 61218
    ## - Gas_Costs:stories                  1  1066551 61229
    ## - class_a:total_dd_07                1  1066953 61232
    ## - class_b:total_dd_07                1  1067107 61233
    ## - Precipitation:total_dd_07          1  1067384 61235
    ## - Electricity_Costs:Gas_Costs        1  1068489 61243
    ## - Electricity_Costs:class_b          1  1070847 61261
    ## - Electricity_Costs:class_a          1  1071428 61265
    ## - Electricity_Costs:size             1  1077078 61306
    ## 
    ## Step:  AIC=61179.64
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + stories + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + leasing_rate:size + 
    ##     Electricity_Costs:class_b + Energystar:Precipitation + Gas_Costs:renovated + 
    ##     class_a:Precipitation + Precipitation:class_b + leasing_rate:Precipitation + 
    ##     Electricity_Costs:leasing_rate + Precipitation:total_dd_07 + 
    ##     size:total_dd_07 + class_a:total_dd_07 + class_b:total_dd_07 + 
    ##     Gas_Costs:net + total_dd_07:stories + Gas_Costs:stories + 
    ##     Electricity_Costs:stories + Precipitation:stories + Gas_Costs:total_dd_07 + 
    ##     renovated:stories + size:renovated + Precipitation:renovated + 
    ##     class_a:stories + Electricity_Costs:total_dd_07 + Energystar:Electricity_Costs + 
    ##     LEED:Electricity_Costs + renovated:total_dd_07 + Electricity_Costs:renovated + 
    ##     size:stories + leasing_rate:Gas_Costs + class_a:Gas_Costs
    ## 
    ##                                     Df Deviance   AIC
    ## + class_b:Gas_Costs                  1  1056751 61162
    ## - Gas_Costs:net                      1  1059595 61179
    ## + net:total_dd_07                    1  1059076 61179
    ## + leasing_rate:stories               1  1059100 61179
    ## - LEED:Energystar                    1  1059645 61180
    ## <none>                                  1059390 61180
    ## + size:net                           1  1059140 61180
    ## + LEED:net                           1  1059153 61180
    ## - class_a:Gas_Costs                  1  1059698 61180
    ## + Energystar:total_dd_07             1  1059164 61180
    ## - Energystar:Electricity_Costs       1  1059715 61180
    ## - Electricity_Costs:Precipitation    1  1059718 61180
    ## - Precipitation:class_b              1  1059720 61180
    ## + renovated:net                      1  1059186 61180
    ## + Energystar:size                    1  1059197 61180
    ## + LEED:total_dd_07                   1  1059198 61180
    ## + age                                1  1059202 61180
    ## - leasing_rate:Gas_Costs             1  1059742 61180
    ## + LEED:Energystar:Electricity_Costs  1  1059218 61180
    ## + Energystar:class_b                 1  1059223 61180
    ## + leasing_rate:class_a               1  1059231 61180
    ## + size:Precipitation                 1  1059238 61181
    ## + Energystar:renovated               1  1059243 61181
    ## + Energystar:Gas_Costs               1  1059246 61181
    ## + leasing_rate:total_dd_07           1  1059254 61181
    ## + LEED:Precipitation                 1  1059275 61181
    ## + LEED:leasing_rate                  1  1059277 61181
    ## + Energystar:stories                 1  1059279 61181
    ## + Energystar:class_a                 1  1059293 61181
    ## + Energystar:net                     1  1059298 61181
    ## + leasing_rate:class_b               1  1059300 61181
    ## + leasing_rate:renovated             1  1059311 61181
    ## + LEED:class_b                       1  1059321 61181
    ## + class_a:net                        1  1059327 61181
    ## + stories:net                        1  1059328 61181
    ## + class_b:net                        1  1059334 61181
    ## + renovated:class_a                  1  1059341 61181
    ## - LEED:Electricity_Costs             1  1059879 61181
    ## + renovated:class_b                  1  1059358 61181
    ## + net:Electricity_Costs              1  1059367 61181
    ## + Energystar:leasing_rate            1  1059367 61181
    ## + stories:class_b                    1  1059368 61181
    ## + leasing_rate:net                   1  1059369 61181
    ## + LEED:class_a                       1  1059375 61182
    ## + LEED:stories                       1  1059383 61182
    ## + net:Precipitation                  1  1059384 61182
    ## + LEED:size                          1  1059384 61182
    ## + size:class_b                       1  1059386 61182
    ## + size:Gas_Costs                     1  1059387 61182
    ## + LEED:renovated                     1  1059389 61182
    ## + LEED:Gas_Costs                     1  1059390 61182
    ## + size:class_a                       1  1059390 61182
    ## + amenities                          1  1059390 61182
    ## - size:stories                       1  1059949 61182
    ## - leasing_rate:Precipitation         1  1059977 61182
    ## - class_a:Precipitation              1  1059983 61182
    ## - Electricity_Costs:total_dd_07      1  1060021 61182
    ## - Precipitation:renovated            1  1060112 61183
    ## - size:total_dd_07                   1  1060277 61184
    ## - Electricity_Costs:renovated        1  1060331 61185
    ## - Gas_Costs:total_dd_07              1  1060529 61186
    ## - class_a:stories                    1  1060678 61187
    ## - Electricity_Costs:stories          1  1060768 61188
    ## - renovated:total_dd_07              1  1060783 61188
    ## - total_dd_07:stories                1  1060927 61189
    ## - size:renovated                     1  1061272 61192
    ## - Energystar:Precipitation           1  1061294 61192
    ## - Electricity_Costs:leasing_rate     1  1061565 61194
    ## - Precipitation:Gas_Costs            1  1062029 61197
    ## - renovated:stories                  1  1063124 61205
    ## - Gas_Costs:renovated                1  1063795 61210
    ## - leasing_rate:size                  1  1063853 61211
    ## - Precipitation:stories              1  1065066 61220
    ## - Gas_Costs:stories                  1  1065973 61227
    ## - class_b:total_dd_07                1  1066676 61232
    ## - class_a:total_dd_07                1  1066941 61234
    ## - Precipitation:total_dd_07          1  1067177 61235
    ## - Electricity_Costs:Gas_Costs        1  1068063 61242
    ## - Electricity_Costs:class_b          1  1070297 61258
    ## - Electricity_Costs:class_a          1  1071334 61266
    ## - Electricity_Costs:size             1  1077013 61308
    ## 
    ## Step:  AIC=61161.95
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + stories + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + leasing_rate:size + 
    ##     Electricity_Costs:class_b + Energystar:Precipitation + Gas_Costs:renovated + 
    ##     class_a:Precipitation + Precipitation:class_b + leasing_rate:Precipitation + 
    ##     Electricity_Costs:leasing_rate + Precipitation:total_dd_07 + 
    ##     size:total_dd_07 + class_a:total_dd_07 + class_b:total_dd_07 + 
    ##     Gas_Costs:net + total_dd_07:stories + Gas_Costs:stories + 
    ##     Electricity_Costs:stories + Precipitation:stories + Gas_Costs:total_dd_07 + 
    ##     renovated:stories + size:renovated + Precipitation:renovated + 
    ##     class_a:stories + Electricity_Costs:total_dd_07 + Energystar:Electricity_Costs + 
    ##     LEED:Electricity_Costs + renovated:total_dd_07 + Electricity_Costs:renovated + 
    ##     size:stories + leasing_rate:Gas_Costs + class_a:Gas_Costs + 
    ##     Gas_Costs:class_b
    ## 
    ##                                     Df Deviance   AIC
    ## - Gas_Costs:net                      1  1056786 61160
    ## - leasing_rate:Gas_Costs             1  1056875 61161
    ## + net:total_dd_07                    1  1056424 61162
    ## + leasing_rate:stories               1  1056437 61162
    ## + stories:class_b                    1  1056451 61162
    ## + leasing_rate:class_a               1  1056469 61162
    ## - Electricity_Costs:Precipitation    1  1057012 61162
    ## - LEED:Energystar                    1  1057015 61162
    ## <none>                                  1056751 61162
    ## + LEED:net                           1  1056486 61162
    ## - Energystar:Electricity_Costs       1  1057056 61162
    ## + LEED:total_dd_07                   1  1056537 61162
    ## + size:net                           1  1056540 61162
    ## + renovated:net                      1  1056543 61162
    ## + Energystar:total_dd_07             1  1056548 61162
    ## + LEED:Energystar:Electricity_Costs  1  1056556 61162
    ## + Energystar:size                    1  1056569 61163
    ## + Energystar:class_b                 1  1056584 61163
    ## + size:Precipitation                 1  1056588 61163
    ## + Energystar:Gas_Costs               1  1056598 61163
    ## + Energystar:renovated               1  1056604 61163
    ## + leasing_rate:renovated             1  1056616 61163
    ## + LEED:Precipitation                 1  1056625 61163
    ## + age                                1  1056630 61163
    ## + LEED:leasing_rate                  1  1056632 61163
    ## + Energystar:class_a                 1  1056648 61163
    ## + Energystar:stories                 1  1056652 61163
    ## + Energystar:net                     1  1056653 61163
    ## + leasing_rate:class_b               1  1056656 61163
    ## + leasing_rate:total_dd_07           1  1056666 61163
    ## + LEED:class_b                       1  1056682 61163
    ## - Precipitation:renovated            1  1057218 61163
    ## + renovated:class_a                  1  1056699 61164
    ## + class_b:net                        1  1056715 61164
    ## + class_a:net                        1  1056722 61164
    ## + stories:net                        1  1056724 61164
    ## + Energystar:leasing_rate            1  1056729 61164
    ## + LEED:class_a                       1  1056732 61164
    ## + net:Electricity_Costs              1  1056737 61164
    ## + renovated:class_b                  1  1056737 61164
    ## + size:class_b                       1  1056740 61164
    ## + LEED:stories                       1  1056742 61164
    ## + LEED:Gas_Costs                     1  1056746 61164
    ## + LEED:size                          1  1056747 61164
    ## + size:class_a                       1  1056747 61164
    ## + amenities                          1  1056749 61164
    ## + size:Gas_Costs                     1  1056750 61164
    ## + net:Precipitation                  1  1056750 61164
    ## + leasing_rate:net                   1  1056751 61164
    ## + LEED:renovated                     1  1056751 61164
    ## - LEED:Electricity_Costs             1  1057321 61164
    ## - Electricity_Costs:total_dd_07      1  1057393 61165
    ## - size:stories                       1  1057398 61165
    ## - Electricity_Costs:renovated        1  1057579 61166
    ## - size:total_dd_07                   1  1057671 61167
    ## - class_a:stories                    1  1057895 61168
    ## - renovated:total_dd_07              1  1058014 61169
    ## - Gas_Costs:total_dd_07              1  1058028 61169
    ## - leasing_rate:Precipitation         1  1058062 61170
    ## - Electricity_Costs:stories          1  1058252 61171
    ## - total_dd_07:stories                1  1058323 61172
    ## - Energystar:Precipitation           1  1058662 61174
    ## - Electricity_Costs:leasing_rate     1  1058728 61175
    ## - size:renovated                     1  1058812 61175
    ## - Precipitation:class_b              1  1058836 61176
    ## - class_a:Precipitation              1  1058879 61176
    ## - Precipitation:Gas_Costs            1  1059186 61178
    ## - Gas_Costs:class_b                  1  1059390 61180
    ## - class_a:Gas_Costs                  1  1059497 61180
    ## - Gas_Costs:renovated                1  1060283 61186
    ## - renovated:stories                  1  1060695 61189
    ## - leasing_rate:size                  1  1061029 61192
    ## - Precipitation:stories              1  1063010 61207
    ## - Gas_Costs:stories                  1  1064293 61216
    ## - Electricity_Costs:Gas_Costs        1  1064776 61220
    ## - Precipitation:total_dd_07          1  1064844 61220
    ## - class_a:total_dd_07                1  1065219 61223
    ## - class_b:total_dd_07                1  1065260 61223
    ## - Electricity_Costs:class_b          1  1069911 61258
    ## - Electricity_Costs:class_a          1  1070584 61263
    ## - Electricity_Costs:size             1  1074210 61289
    ## 
    ## Step:  AIC=61160.2
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + stories + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + leasing_rate:size + 
    ##     Electricity_Costs:class_b + Energystar:Precipitation + Gas_Costs:renovated + 
    ##     class_a:Precipitation + Precipitation:class_b + leasing_rate:Precipitation + 
    ##     Electricity_Costs:leasing_rate + Precipitation:total_dd_07 + 
    ##     size:total_dd_07 + class_a:total_dd_07 + class_b:total_dd_07 + 
    ##     total_dd_07:stories + Gas_Costs:stories + Electricity_Costs:stories + 
    ##     Precipitation:stories + Gas_Costs:total_dd_07 + renovated:stories + 
    ##     size:renovated + Precipitation:renovated + class_a:stories + 
    ##     Electricity_Costs:total_dd_07 + Energystar:Electricity_Costs + 
    ##     LEED:Electricity_Costs + renovated:total_dd_07 + Electricity_Costs:renovated + 
    ##     size:stories + leasing_rate:Gas_Costs + class_a:Gas_Costs + 
    ##     Gas_Costs:class_b
    ## 
    ##                                     Df Deviance   AIC
    ## - leasing_rate:Gas_Costs             1  1056912 61159
    ## + net:total_dd_07                    1  1056452 61160
    ## + leasing_rate:stories               1  1056467 61160
    ## + stories:class_b                    1  1056479 61160
    ## + leasing_rate:class_a               1  1056505 61160
    ## - Electricity_Costs:Precipitation    1  1057041 61160
    ## <none>                                  1056786 61160
    ## - LEED:Energystar                    1  1057054 61160
    ## + LEED:net                           1  1056518 61160
    ## - Energystar:Electricity_Costs       1  1057089 61160
    ## + renovated:net                      1  1056555 61160
    ## + size:net                           1  1056557 61160
    ## + LEED:total_dd_07                   1  1056567 61161
    ## + Energystar:total_dd_07             1  1056581 61161
    ## + LEED:Energystar:Electricity_Costs  1  1056587 61161
    ## + Energystar:size                    1  1056606 61161
    ## + size:Precipitation                 1  1056615 61161
    ## + Energystar:class_b                 1  1056618 61161
    ## + Energystar:Gas_Costs               1  1056634 61161
    ## + Energystar:renovated               1  1056637 61161
    ## + leasing_rate:renovated             1  1056650 61161
    ## + LEED:Precipitation                 1  1056659 61161
    ## + LEED:leasing_rate                  1  1056667 61161
    ## + age                                1  1056667 61161
    ## + Energystar:class_a                 1  1056682 61161
    ## + Energystar:stories                 1  1056688 61161
    ## + leasing_rate:class_b               1  1056692 61162
    ## + Energystar:net                     1  1056698 61162
    ## + leasing_rate:total_dd_07           1  1056701 61162
    ## + LEED:class_b                       1  1056718 61162
    ## + renovated:class_a                  1  1056735 61162
    ## + net:Electricity_Costs              1  1056738 61162
    ## - Precipitation:renovated            1  1057280 61162
    ## + net:Gas_Costs                      1  1056751 61162
    ## + class_b:net                        1  1056754 61162
    ## + stories:net                        1  1056758 61162
    ## + class_a:net                        1  1056763 61162
    ## + Energystar:leasing_rate            1  1056763 61162
    ## + LEED:class_a                       1  1056766 61162
    ## + renovated:class_b                  1  1056772 61162
    ## + size:class_b                       1  1056774 61162
    ## + LEED:stories                       1  1056776 61162
    ## + LEED:Gas_Costs                     1  1056779 61162
    ## + net:Precipitation                  1  1056779 61162
    ## + size:class_a                       1  1056781 61162
    ## + LEED:size                          1  1056782 61162
    ## + amenities                          1  1056784 61162
    ## + LEED:renovated                     1  1056785 61162
    ## + size:Gas_Costs                     1  1056785 61162
    ## + leasing_rate:net                   1  1056786 61162
    ## - LEED:Electricity_Costs             1  1057376 61163
    ## - size:stories                       1  1057441 61163
    ## - Electricity_Costs:total_dd_07      1  1057446 61163
    ## - Electricity_Costs:renovated        1  1057598 61164
    ## - size:total_dd_07                   1  1057717 61165
    ## - class_a:stories                    1  1057928 61167
    ## - renovated:total_dd_07              1  1058031 61167
    ## - Gas_Costs:total_dd_07              1  1058067 61168
    ## - leasing_rate:Precipitation         1  1058095 61168
    ## - Electricity_Costs:stories          1  1058264 61169
    ## - total_dd_07:stories                1  1058337 61170
    ## - Energystar:Precipitation           1  1058697 61172
    ## - Electricity_Costs:leasing_rate     1  1058759 61173
    ## - size:renovated                     1  1058846 61174
    ## - Precipitation:class_b              1  1058922 61174
    ## - class_a:Precipitation              1  1058968 61174
    ## - Precipitation:Gas_Costs            1  1059213 61176
    ## - Gas_Costs:class_b                  1  1059595 61179
    ## - class_a:Gas_Costs                  1  1059764 61180
    ## - Gas_Costs:renovated                1  1060529 61186
    ## - renovated:stories                  1  1060755 61188
    ## - leasing_rate:size                  1  1061069 61190
    ## - net                                1  1061485 61193
    ## - Precipitation:stories              1  1063244 61206
    ## - Gas_Costs:stories                  1  1064745 61217
    ## - Electricity_Costs:Gas_Costs        1  1064810 61218
    ## - Precipitation:total_dd_07          1  1064920 61219
    ## - class_a:total_dd_07                1  1065233 61221
    ## - class_b:total_dd_07                1  1065263 61221
    ## - Electricity_Costs:class_b          1  1069912 61256
    ## - Electricity_Costs:class_a          1  1070584 61261
    ## - Electricity_Costs:size             1  1074402 61289
    ## 
    ## Step:  AIC=61159.15
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + stories + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + leasing_rate:size + 
    ##     Electricity_Costs:class_b + Energystar:Precipitation + Gas_Costs:renovated + 
    ##     class_a:Precipitation + Precipitation:class_b + leasing_rate:Precipitation + 
    ##     Electricity_Costs:leasing_rate + Precipitation:total_dd_07 + 
    ##     size:total_dd_07 + class_a:total_dd_07 + class_b:total_dd_07 + 
    ##     total_dd_07:stories + Gas_Costs:stories + Electricity_Costs:stories + 
    ##     Precipitation:stories + Gas_Costs:total_dd_07 + renovated:stories + 
    ##     size:renovated + Precipitation:renovated + class_a:stories + 
    ##     Electricity_Costs:total_dd_07 + Energystar:Electricity_Costs + 
    ##     LEED:Electricity_Costs + renovated:total_dd_07 + Electricity_Costs:renovated + 
    ##     size:stories + class_a:Gas_Costs + Gas_Costs:class_b
    ## 
    ##                                     Df Deviance   AIC
    ## + net:total_dd_07                    1  1056576 61159
    ## + leasing_rate:stories               1  1056587 61159
    ## + leasing_rate:class_a               1  1056642 61159
    ## + LEED:net                           1  1056643 61159
    ## <none>                                  1056912 61159
    ## + stories:class_b                    1  1056647 61159
    ## - LEED:Energystar                    1  1057184 61159
    ## - Electricity_Costs:Precipitation    1  1057184 61159
    ## - Energystar:Electricity_Costs       1  1057212 61159
    ## + size:net                           1  1056680 61159
    ## + renovated:net                      1  1056683 61159
    ## + LEED:total_dd_07                   1  1056684 61159
    ## + Energystar:total_dd_07             1  1056704 61160
    ## + LEED:Energystar:Electricity_Costs  1  1056709 61160
    ## + Energystar:size                    1  1056729 61160
    ## + size:Precipitation                 1  1056734 61160
    ## + Energystar:class_b                 1  1056742 61160
    ## + Energystar:renovated               1  1056762 61160
    ## + Energystar:Gas_Costs               1  1056777 61160
    ## + LEED:Precipitation                 1  1056783 61160
    ## + leasing_rate:Gas_Costs             1  1056786 61160
    ## + age                                1  1056791 61160
    ## + LEED:leasing_rate                  1  1056795 61160
    ## + leasing_rate:renovated             1  1056807 61160
    ## + Energystar:class_a                 1  1056808 61160
    ## + Energystar:stories                 1  1056813 61160
    ## + leasing_rate:class_b               1  1056820 61160
    ## + Energystar:net                     1  1056824 61160
    ## + LEED:class_b                       1  1056847 61161
    ## + renovated:class_a                  1  1056862 61161
    ## + net:Electricity_Costs              1  1056864 61161
    ## + net:Gas_Costs                      1  1056875 61161
    ## + leasing_rate:total_dd_07           1  1056879 61161
    ## + stories:net                        1  1056882 61161
    ## + class_b:net                        1  1056883 61161
    ## + Energystar:leasing_rate            1  1056890 61161
    ## + LEED:class_a                       1  1056891 61161
    ## + class_a:net                        1  1056891 61161
    ## + renovated:class_b                  1  1056897 61161
    ## + LEED:Gas_Costs                     1  1056902 61161
    ## + LEED:stories                       1  1056903 61161
    ## + size:class_b                       1  1056904 61161
    ## + net:Precipitation                  1  1056905 61161
    ## + LEED:size                          1  1056909 61161
    ## + size:class_a                       1  1056909 61161
    ## + amenities                          1  1056910 61161
    ## + LEED:renovated                     1  1056912 61161
    ## + leasing_rate:net                   1  1056912 61161
    ## + size:Gas_Costs                     1  1056912 61161
    ## - Precipitation:renovated            1  1057459 61161
    ## - LEED:Electricity_Costs             1  1057535 61162
    ## - Electricity_Costs:total_dd_07      1  1057579 61162
    ## - size:stories                       1  1057587 61162
    ## - Electricity_Costs:renovated        1  1057730 61163
    ## - size:total_dd_07                   1  1057840 61164
    ## - class_a:stories                    1  1058058 61166
    ## - renovated:total_dd_07              1  1058157 61166
    ## - Gas_Costs:total_dd_07              1  1058165 61167
    ## - Electricity_Costs:stories          1  1058394 61168
    ## - leasing_rate:Precipitation         1  1058439 61169
    ## - total_dd_07:stories                1  1058454 61169
    ## - Electricity_Costs:leasing_rate     1  1058782 61171
    ## - Energystar:Precipitation           1  1058812 61171
    ## - Precipitation:class_b              1  1058922 61172
    ## - size:renovated                     1  1058959 61172
    ## - class_a:Precipitation              1  1058968 61172
    ## - Precipitation:Gas_Costs            1  1059393 61176
    ## - Gas_Costs:class_b                  1  1059996 61180
    ## - class_a:Gas_Costs                  1  1060079 61181
    ## - renovated:stories                  1  1060865 61187
    ## - Gas_Costs:renovated                1  1060976 61187
    ## - leasing_rate:size                  1  1061284 61190
    ## - net                                1  1061583 61192
    ## - Precipitation:stories              1  1063787 61208
    ## - Electricity_Costs:Gas_Costs        1  1064975 61217
    ## - Precipitation:total_dd_07          1  1065058 61218
    ## - class_a:total_dd_07                1  1065609 61222
    ## - class_b:total_dd_07                1  1065673 61222
    ## - Gas_Costs:stories                  1  1065828 61223
    ## - Electricity_Costs:class_b          1  1070546 61258
    ## - Electricity_Costs:class_a          1  1071049 61262
    ## - Electricity_Costs:size             1  1074559 61288
    ## 
    ## Step:  AIC=61158.64
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + stories + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + leasing_rate:size + 
    ##     Electricity_Costs:class_b + Energystar:Precipitation + Gas_Costs:renovated + 
    ##     class_a:Precipitation + Precipitation:class_b + leasing_rate:Precipitation + 
    ##     Electricity_Costs:leasing_rate + Precipitation:total_dd_07 + 
    ##     size:total_dd_07 + class_a:total_dd_07 + class_b:total_dd_07 + 
    ##     total_dd_07:stories + Gas_Costs:stories + Electricity_Costs:stories + 
    ##     Precipitation:stories + Gas_Costs:total_dd_07 + renovated:stories + 
    ##     size:renovated + Precipitation:renovated + class_a:stories + 
    ##     Electricity_Costs:total_dd_07 + Energystar:Electricity_Costs + 
    ##     LEED:Electricity_Costs + renovated:total_dd_07 + Electricity_Costs:renovated + 
    ##     size:stories + class_a:Gas_Costs + Gas_Costs:class_b + net:total_dd_07
    ## 
    ##                                     Df Deviance   AIC
    ## + size:net                           1  1056037 61157
    ## + leasing_rate:stories               1  1056249 61158
    ## + renovated:net                      1  1056249 61158
    ## + net:Electricity_Costs              1  1056265 61158
    ## + LEED:net                           1  1056291 61159
    ## - LEED:Energystar                    1  1056843 61159
    ## <none>                                  1056576 61159
    ## + leasing_rate:class_a               1  1056311 61159
    ## + stories:class_b                    1  1056315 61159
    ## - Electricity_Costs:Precipitation    1  1056863 61159
    ## - Energystar:Electricity_Costs       1  1056873 61159
    ## + LEED:total_dd_07                   1  1056344 61159
    ## + size:Precipitation                 1  1056353 61159
    ## + Energystar:total_dd_07             1  1056366 61159
    ## - net:total_dd_07                    1  1056912 61159
    ## + LEED:Energystar:Electricity_Costs  1  1056380 61159
    ## + stories:net                        1  1056389 61159
    ## + Energystar:size                    1  1056398 61159
    ## + Energystar:class_b                 1  1056407 61159
    ## + Energystar:renovated               1  1056426 61160
    ## + LEED:Precipitation                 1  1056438 61160
    ## + Energystar:Gas_Costs               1  1056443 61160
    ## + age                                1  1056444 61160
    ## + leasing_rate:Gas_Costs             1  1056452 61160
    ## + LEED:leasing_rate                  1  1056466 61160
    ## + Energystar:class_a                 1  1056472 61160
    ## + leasing_rate:renovated             1  1056472 61160
    ## + Energystar:stories                 1  1056477 61160
    ## + leasing_rate:class_b               1  1056484 61160
    ## + class_a:net                        1  1056491 61160
    ## + Energystar:net                     1  1056496 61160
    ## + class_b:net                        1  1056503 61160
    ## + LEED:class_b                       1  1056513 61160
    ## + renovated:class_a                  1  1056526 61160
    ## + leasing_rate:total_dd_07           1  1056540 61160
    ## + net:Gas_Costs                      1  1056545 61160
    ## + LEED:class_a                       1  1056552 61160
    ## + Energystar:leasing_rate            1  1056553 61160
    ## + net:Precipitation                  1  1056560 61161
    ## + renovated:class_b                  1  1056561 61161
    ## + size:class_b                       1  1056562 61161
    ## + LEED:Gas_Costs                     1  1056563 61161
    ## + LEED:stories                       1  1056565 61161
    ## + leasing_rate:net                   1  1056567 61161
    ## + size:class_a                       1  1056568 61161
    ## + amenities                          1  1056572 61161
    ## + LEED:size                          1  1056573 61161
    ## + LEED:renovated                     1  1056575 61161
    ## + size:Gas_Costs                     1  1056576 61161
    ## - Precipitation:renovated            1  1057150 61161
    ## - LEED:Electricity_Costs             1  1057200 61161
    ## - size:stories                       1  1057244 61162
    ## - Electricity_Costs:total_dd_07      1  1057250 61162
    ## - Electricity_Costs:renovated        1  1057391 61163
    ## - size:total_dd_07                   1  1057446 61163
    ## - class_a:stories                    1  1057754 61165
    ## - renovated:total_dd_07              1  1057805 61166
    ## - Gas_Costs:total_dd_07              1  1057841 61166
    ## - Electricity_Costs:stories          1  1058044 61168
    ## - total_dd_07:stories                1  1058087 61168
    ## - leasing_rate:Precipitation         1  1058098 61168
    ## - Electricity_Costs:leasing_rate     1  1058436 61171
    ## - Energystar:Precipitation           1  1058454 61171
    ## - Precipitation:class_b              1  1058589 61172
    ## - size:renovated                     1  1058614 61172
    ## - class_a:Precipitation              1  1058644 61172
    ## - Precipitation:Gas_Costs            1  1059065 61175
    ## - Gas_Costs:class_b                  1  1059674 61180
    ## - class_a:Gas_Costs                  1  1059780 61181
    ## - renovated:stories                  1  1060508 61186
    ## - Gas_Costs:renovated                1  1060673 61187
    ## - leasing_rate:size                  1  1060961 61189
    ## - Precipitation:stories              1  1063587 61209
    ## - Electricity_Costs:Gas_Costs        1  1064578 61216
    ## - Precipitation:total_dd_07          1  1064683 61217
    ## - class_a:total_dd_07                1  1065270 61221
    ## - class_b:total_dd_07                1  1065304 61222
    ## - Gas_Costs:stories                  1  1065552 61223
    ## - Electricity_Costs:class_b          1  1070185 61258
    ## - Electricity_Costs:class_a          1  1070722 61262
    ## - Electricity_Costs:size             1  1074290 61288
    ## 
    ## Step:  AIC=61156.61
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + stories + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + leasing_rate:size + 
    ##     Electricity_Costs:class_b + Energystar:Precipitation + Gas_Costs:renovated + 
    ##     class_a:Precipitation + Precipitation:class_b + leasing_rate:Precipitation + 
    ##     Electricity_Costs:leasing_rate + Precipitation:total_dd_07 + 
    ##     size:total_dd_07 + class_a:total_dd_07 + class_b:total_dd_07 + 
    ##     total_dd_07:stories + Gas_Costs:stories + Electricity_Costs:stories + 
    ##     Precipitation:stories + Gas_Costs:total_dd_07 + renovated:stories + 
    ##     size:renovated + Precipitation:renovated + class_a:stories + 
    ##     Electricity_Costs:total_dd_07 + Energystar:Electricity_Costs + 
    ##     LEED:Electricity_Costs + renovated:total_dd_07 + Electricity_Costs:renovated + 
    ##     size:stories + class_a:Gas_Costs + Gas_Costs:class_b + net:total_dd_07 + 
    ##     size:net
    ## 
    ##                                     Df Deviance   AIC
    ## + stories:net                        1  1055654 61156
    ## + leasing_rate:stories               1  1055705 61156
    ## + LEED:net                           1  1055751 61156
    ## + leasing_rate:class_a               1  1055756 61157
    ## - LEED:Energystar                    1  1056296 61157
    ## <none>                                  1056037 61157
    ## - Electricity_Costs:Precipitation    1  1056311 61157
    ## + stories:class_b                    1  1055778 61157
    ## - Energystar:Electricity_Costs       1  1056339 61157
    ## + LEED:total_dd_07                   1  1055805 61157
    ## + Energystar:size                    1  1055817 61157
    ## + net:Electricity_Costs              1  1055819 61157
    ## + Energystar:total_dd_07             1  1055828 61157
    ## + LEED:Energystar:Electricity_Costs  1  1055839 61157
    ## + renovated:net                      1  1055844 61157
    ## + Energystar:class_b                 1  1055866 61157
    ## + size:Precipitation                 1  1055891 61158
    ## + LEED:Precipitation                 1  1055897 61158
    ## + Energystar:Gas_Costs               1  1055901 61158
    ## + Energystar:renovated               1  1055906 61158
    ## + Energystar:stories                 1  1055913 61158
    ## + age                                1  1055913 61158
    ## + leasing_rate:Gas_Costs             1  1055921 61158
    ## + LEED:leasing_rate                  1  1055926 61158
    ## + leasing_rate:renovated             1  1055929 61158
    ## + Energystar:net                     1  1055931 61158
    ## + Energystar:class_a                 1  1055933 61158
    ## + leasing_rate:class_b               1  1055943 61158
    ## + LEED:class_b                       1  1055971 61158
    ## + renovated:class_a                  1  1055980 61158
    ## + net:Precipitation                  1  1055998 61158
    ## + leasing_rate:total_dd_07           1  1056004 61158
    ## + LEED:class_a                       1  1056015 61158
    ## + Energystar:leasing_rate            1  1056018 61158
    ## - Precipitation:renovated            1  1056557 61159
    ## + renovated:class_b                  1  1056023 61159
    ## + LEED:Gas_Costs                     1  1056026 61159
    ## + LEED:stories                       1  1056030 61159
    ## + net:Gas_Costs                      1  1056031 61159
    ## + LEED:size                          1  1056032 61159
    ## + size:class_b                       1  1056033 61159
    ## + amenities                          1  1056034 61159
    ## + class_a:net                        1  1056035 61159
    ## + size:class_a                       1  1056036 61159
    ## + leasing_rate:net                   1  1056036 61159
    ## + LEED:renovated                     1  1056036 61159
    ## + size:Gas_Costs                     1  1056037 61159
    ## + class_b:net                        1  1056037 61159
    ## - size:net                           1  1056576 61159
    ## - LEED:Electricity_Costs             1  1056651 61159
    ## - net:total_dd_07                    1  1056680 61159
    ## - Electricity_Costs:total_dd_07      1  1056691 61159
    ## - size:stories                       1  1056706 61160
    ## - Electricity_Costs:renovated        1  1056864 61161
    ## - size:total_dd_07                   1  1056877 61161
    ## - class_a:stories                    1  1057058 61162
    ## - renovated:total_dd_07              1  1057259 61164
    ## - Gas_Costs:total_dd_07              1  1057369 61165
    ## - Electricity_Costs:stories          1  1057511 61166
    ## - leasing_rate:Precipitation         1  1057575 61166
    ## - total_dd_07:stories                1  1057675 61167
    ## - Energystar:Precipitation           1  1057903 61169
    ## - Electricity_Costs:leasing_rate     1  1057904 61169
    ## - Precipitation:class_b              1  1057987 61169
    ## - class_a:Precipitation              1  1058076 61170
    ## - size:renovated                     1  1058166 61171
    ## - Precipitation:Gas_Costs            1  1058499 61173
    ## - Gas_Costs:class_b                  1  1059016 61177
    ## - class_a:Gas_Costs                  1  1059142 61178
    ## - renovated:stories                  1  1059897 61183
    ## - Gas_Costs:renovated                1  1060022 61184
    ## - leasing_rate:size                  1  1060242 61186
    ## - Precipitation:stories              1  1062648 61204
    ## - Electricity_Costs:Gas_Costs        1  1064116 61215
    ## - Precipitation:total_dd_07          1  1064247 61216
    ## - Gas_Costs:stories                  1  1064757 61220
    ## - class_b:total_dd_07                1  1064858 61220
    ## - class_a:total_dd_07                1  1064868 61220
    ## - Electricity_Costs:class_b          1  1069663 61256
    ## - Electricity_Costs:class_a          1  1070174 61260
    ## - Electricity_Costs:size             1  1073709 61286
    ## 
    ## Step:  AIC=61155.75
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + stories + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + leasing_rate:size + 
    ##     Electricity_Costs:class_b + Energystar:Precipitation + Gas_Costs:renovated + 
    ##     class_a:Precipitation + Precipitation:class_b + leasing_rate:Precipitation + 
    ##     Electricity_Costs:leasing_rate + Precipitation:total_dd_07 + 
    ##     size:total_dd_07 + class_a:total_dd_07 + class_b:total_dd_07 + 
    ##     total_dd_07:stories + Gas_Costs:stories + Electricity_Costs:stories + 
    ##     Precipitation:stories + Gas_Costs:total_dd_07 + renovated:stories + 
    ##     size:renovated + Precipitation:renovated + class_a:stories + 
    ##     Electricity_Costs:total_dd_07 + Energystar:Electricity_Costs + 
    ##     LEED:Electricity_Costs + renovated:total_dd_07 + Electricity_Costs:renovated + 
    ##     size:stories + class_a:Gas_Costs + Gas_Costs:class_b + net:total_dd_07 + 
    ##     size:net + net:stories
    ## 
    ##                                     Df Deviance   AIC
    ## + LEED:net                           1  1055275 61155
    ## + leasing_rate:stories               1  1055334 61155
    ## + leasing_rate:class_a               1  1055372 61156
    ## - Electricity_Costs:Precipitation    1  1055916 61156
    ## <none>                                  1055654 61156
    ## - LEED:Energystar                    1  1055927 61156
    ## + stories:class_b                    1  1055401 61156
    ## - Energystar:Electricity_Costs       1  1055948 61156
    ## + LEED:total_dd_07                   1  1055421 61156
    ## + Energystar:size                    1  1055444 61156
    ## + LEED:Energystar:Electricity_Costs  1  1055449 61156
    ## + Energystar:total_dd_07             1  1055454 61156
    ## + Energystar:class_b                 1  1055485 61156
    ## - net:stories                        1  1056037 61157
    ## + LEED:Precipitation                 1  1055519 61157
    ## + Energystar:Gas_Costs               1  1055520 61157
    ## + Energystar:renovated               1  1055522 61157
    ## + leasing_rate:Gas_Costs             1  1055523 61157
    ## + age                                1  1055530 61157
    ## + size:Precipitation                 1  1055531 61157
    ## + Energystar:net                     1  1055540 61157
    ## + net:Precipitation                  1  1055540 61157
    ## + Energystar:stories                 1  1055541 61157
    ## + LEED:leasing_rate                  1  1055545 61157
    ## + renovated:net                      1  1055546 61157
    ## + leasing_rate:renovated             1  1055548 61157
    ## + Energystar:class_a                 1  1055551 61157
    ## + leasing_rate:class_b               1  1055560 61157
    ## + net:Electricity_Costs              1  1055586 61157
    ## - net:total_dd_07                    1  1056127 61157
    ## + LEED:class_b                       1  1055594 61157
    ## + renovated:class_a                  1  1055599 61157
    ## - Precipitation:renovated            1  1056149 61157
    ## + leasing_rate:total_dd_07           1  1055626 61158
    ## + LEED:class_a                       1  1055628 61158
    ## + Energystar:leasing_rate            1  1055634 61158
    ## + renovated:class_b                  1  1055639 61158
    ## + class_b:net                        1  1055643 61158
    ## + LEED:Gas_Costs                     1  1055644 61158
    ## + size:Gas_Costs                     1  1055646 61158
    ## + LEED:stories                       1  1055647 61158
    ## + net:Gas_Costs                      1  1055647 61158
    ## + LEED:size                          1  1055650 61158
    ## + class_a:net                        1  1055650 61158
    ## + amenities                          1  1055651 61158
    ## + leasing_rate:net                   1  1055653 61158
    ## + LEED:renovated                     1  1055654 61158
    ## + size:class_b                       1  1055654 61158
    ## + size:class_a                       1  1055654 61158
    ## - LEED:Electricity_Costs             1  1056244 61158
    ## - Electricity_Costs:total_dd_07      1  1056302 61159
    ## - size:stories                       1  1056326 61159
    ## - size:net                           1  1056389 61159
    ## - Electricity_Costs:renovated        1  1056481 61160
    ## - size:total_dd_07                   1  1056555 61160
    ## - class_a:stories                    1  1056702 61162
    ## - renovated:total_dd_07              1  1056895 61163
    ## - Gas_Costs:total_dd_07              1  1056992 61164
    ## - Electricity_Costs:stories          1  1057004 61164
    ## - leasing_rate:Precipitation         1  1057182 61165
    ## - total_dd_07:stories                1  1057215 61165
    ## - Electricity_Costs:leasing_rate     1  1057480 61167
    ## - Energystar:Precipitation           1  1057485 61167
    ## - Precipitation:class_b              1  1057614 61168
    ## - class_a:Precipitation              1  1057720 61169
    ## - size:renovated                     1  1057846 61170
    ## - Precipitation:Gas_Costs            1  1058045 61172
    ## - Gas_Costs:class_b                  1  1058681 61176
    ## - class_a:Gas_Costs                  1  1058849 61178
    ## - Gas_Costs:renovated                1  1059498 61182
    ## - renovated:stories                  1  1059500 61182
    ## - leasing_rate:size                  1  1059856 61185
    ## - Precipitation:stories              1  1062248 61203
    ## - Electricity_Costs:Gas_Costs        1  1063809 61214
    ## - Precipitation:total_dd_07          1  1063919 61215
    ## - class_b:total_dd_07                1  1064515 61220
    ## - class_a:total_dd_07                1  1064531 61220
    ## - Gas_Costs:stories                  1  1064590 61220
    ## - Electricity_Costs:class_b          1  1069264 61255
    ## - Electricity_Costs:class_a          1  1069850 61259
    ## - Electricity_Costs:size             1  1073648 61287
    ## 
    ## Step:  AIC=61154.91
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + stories + LEED:Energystar + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + leasing_rate:size + 
    ##     Electricity_Costs:class_b + Energystar:Precipitation + Gas_Costs:renovated + 
    ##     class_a:Precipitation + Precipitation:class_b + leasing_rate:Precipitation + 
    ##     Electricity_Costs:leasing_rate + Precipitation:total_dd_07 + 
    ##     size:total_dd_07 + class_a:total_dd_07 + class_b:total_dd_07 + 
    ##     total_dd_07:stories + Gas_Costs:stories + Electricity_Costs:stories + 
    ##     Precipitation:stories + Gas_Costs:total_dd_07 + renovated:stories + 
    ##     size:renovated + Precipitation:renovated + class_a:stories + 
    ##     Electricity_Costs:total_dd_07 + Energystar:Electricity_Costs + 
    ##     LEED:Electricity_Costs + renovated:total_dd_07 + Electricity_Costs:renovated + 
    ##     size:stories + class_a:Gas_Costs + Gas_Costs:class_b + net:total_dd_07 + 
    ##     size:net + net:stories + LEED:net
    ## 
    ##                                     Df Deviance   AIC
    ## - LEED:Energystar                    1  1055486 61154
    ## + leasing_rate:stories               1  1054957 61155
    ## + leasing_rate:class_a               1  1054991 61155
    ## - Electricity_Costs:Precipitation    1  1055540 61155
    ## <none>                                  1055275 61155
    ## + stories:class_b                    1  1055021 61155
    ## - Energystar:Electricity_Costs       1  1055580 61155
    ## + Energystar:size                    1  1055066 61155
    ## + LEED:total_dd_07                   1  1055068 61155
    ## + Energystar:total_dd_07             1  1055072 61155
    ## + Energystar:class_b                 1  1055103 61156
    ## + LEED:Energystar:Electricity_Costs  1  1055108 61156
    ## - LEED:net                           1  1055654 61156
    ## + net:Precipitation                  1  1055131 61156
    ## + leasing_rate:Gas_Costs             1  1055144 61156
    ## + LEED:class_b                       1  1055146 61156
    ## + Energystar:Gas_Costs               1  1055147 61156
    ## + age                                1  1055148 61156
    ## + LEED:leasing_rate                  1  1055153 61156
    ## + size:Precipitation                 1  1055157 61156
    ## + LEED:Precipitation                 1  1055159 61156
    ## + Energystar:renovated               1  1055161 61156
    ## + Energystar:stories                 1  1055161 61156
    ## + leasing_rate:renovated             1  1055168 61156
    ## + Energystar:class_a                 1  1055170 61156
    ## + Energystar:net                     1  1055174 61156
    ## + leasing_rate:class_b               1  1055180 61156
    ## + renovated:net                      1  1055185 61156
    ## - net:stories                        1  1055751 61156
    ## - net:total_dd_07                    1  1055751 61156
    ## + renovated:class_a                  1  1055219 61156
    ## + net:Electricity_Costs              1  1055225 61157
    ## - Precipitation:renovated            1  1055763 61157
    ## + class_b:net                        1  1055247 61157
    ## + leasing_rate:total_dd_07           1  1055247 61157
    ## + Energystar:leasing_rate            1  1055252 61157
    ## + class_a:net                        1  1055257 61157
    ## + renovated:class_b                  1  1055259 61157
    ## + net:Gas_Costs                      1  1055262 61157
    ## + LEED:stories                       1  1055264 61157
    ## + size:Gas_Costs                     1  1055266 61157
    ## + LEED:size                          1  1055270 61157
    ## + amenities                          1  1055270 61157
    ## + LEED:Gas_Costs                     1  1055271 61157
    ## + leasing_rate:net                   1  1055272 61157
    ## + LEED:class_a                       1  1055273 61157
    ## + size:class_a                       1  1055274 61157
    ## + size:class_b                       1  1055274 61157
    ## + LEED:renovated                     1  1055275 61157
    ## - size:stories                       1  1055937 61158
    ## - Electricity_Costs:total_dd_07      1  1055945 61158
    ## - LEED:Electricity_Costs             1  1056014 61158
    ## - Electricity_Costs:renovated        1  1056093 61159
    ## - size:net                           1  1056125 61159
    ## - size:total_dd_07                   1  1056182 61160
    ## - class_a:stories                    1  1056315 61161
    ## - renovated:total_dd_07              1  1056515 61162
    ## - Electricity_Costs:stories          1  1056611 61163
    ## - Gas_Costs:total_dd_07              1  1056621 61163
    ## - leasing_rate:Precipitation         1  1056791 61164
    ## - total_dd_07:stories                1  1056826 61165
    ## - Electricity_Costs:leasing_rate     1  1057098 61167
    ## - Energystar:Precipitation           1  1057157 61167
    ## - Precipitation:class_b              1  1057256 61168
    ## - class_a:Precipitation              1  1057353 61168
    ## - size:renovated                     1  1057469 61169
    ## - Precipitation:Gas_Costs            1  1057669 61171
    ## - Gas_Costs:class_b                  1  1058351 61176
    ## - class_a:Gas_Costs                  1  1058503 61177
    ## - renovated:stories                  1  1059111 61182
    ## - Gas_Costs:renovated                1  1059133 61182
    ## - leasing_rate:size                  1  1059453 61184
    ## - Precipitation:stories              1  1061886 61202
    ## - Electricity_Costs:Gas_Costs        1  1063382 61213
    ## - Precipitation:total_dd_07          1  1063577 61215
    ## - class_b:total_dd_07                1  1064127 61219
    ## - class_a:total_dd_07                1  1064151 61219
    ## - Gas_Costs:stories                  1  1064273 61220
    ## - Electricity_Costs:class_b          1  1068892 61254
    ## - Electricity_Costs:class_a          1  1069507 61259
    ## - Electricity_Costs:size             1  1073321 61287
    ## 
    ## Step:  AIC=61154.49
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + stories + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + leasing_rate:size + 
    ##     Electricity_Costs:class_b + Energystar:Precipitation + Gas_Costs:renovated + 
    ##     class_a:Precipitation + Precipitation:class_b + leasing_rate:Precipitation + 
    ##     Electricity_Costs:leasing_rate + Precipitation:total_dd_07 + 
    ##     size:total_dd_07 + class_a:total_dd_07 + class_b:total_dd_07 + 
    ##     total_dd_07:stories + Gas_Costs:stories + Electricity_Costs:stories + 
    ##     Precipitation:stories + Gas_Costs:total_dd_07 + renovated:stories + 
    ##     size:renovated + Precipitation:renovated + class_a:stories + 
    ##     Electricity_Costs:total_dd_07 + Energystar:Electricity_Costs + 
    ##     LEED:Electricity_Costs + renovated:total_dd_07 + Electricity_Costs:renovated + 
    ##     size:stories + class_a:Gas_Costs + Gas_Costs:class_b + net:total_dd_07 + 
    ##     size:net + net:stories + LEED:net
    ## 
    ##                                   Df Deviance   AIC
    ## + leasing_rate:stories             1  1055173 61154
    ## + leasing_rate:class_a             1  1055198 61154
    ## - Electricity_Costs:Precipitation  1  1055748 61154
    ## <none>                                1055486 61154
    ## + stories:class_b                  1  1055230 61155
    ## + Energystar:size                  1  1055263 61155
    ## + LEED:Energystar                  1  1055275 61155
    ## + LEED:total_dd_07                 1  1055279 61155
    ## - Energystar:Electricity_Costs     1  1055821 61155
    ## + Energystar:total_dd_07           1  1055296 61155
    ## + LEED:Precipitation               1  1055306 61155
    ## + Energystar:class_b               1  1055312 61155
    ## + LEED:leasing_rate                1  1055334 61155
    ## + Energystar:Gas_Costs             1  1055337 61155
    ## + leasing_rate:Gas_Costs           1  1055352 61155
    ## + net:Precipitation                1  1055354 61156
    ## + age                              1  1055363 61156
    ## + Energystar:renovated             1  1055366 61156
    ## + Energystar:stories               1  1055373 61156
    ## + size:Precipitation               1  1055376 61156
    ## + Energystar:net                   1  1055377 61156
    ## + Energystar:class_a               1  1055378 61156
    ## + leasing_rate:renovated           1  1055380 61156
    ## + LEED:class_b                     1  1055390 61156
    ## + leasing_rate:class_b             1  1055392 61156
    ## - LEED:net                         1  1055927 61156
    ## + renovated:net                    1  1055407 61156
    ## - net:stories                      1  1055955 61156
    ## + net:Electricity_Costs            1  1055432 61156
    ## + renovated:class_a                1  1055435 61156
    ## - net:total_dd_07                  1  1055971 61156
    ## - Precipitation:renovated          1  1055974 61156
    ## + class_b:net                      1  1055457 61156
    ## + leasing_rate:total_dd_07         1  1055459 61156
    ## + Energystar:leasing_rate          1  1055466 61156
    ## + class_a:net                      1  1055467 61156
    ## + renovated:class_b                1  1055468 61156
    ## + LEED:stories                     1  1055472 61156
    ## + net:Gas_Costs                    1  1055475 61156
    ## + LEED:class_a                     1  1055477 61156
    ## + size:Gas_Costs                   1  1055477 61156
    ## + LEED:Gas_Costs                   1  1055478 61156
    ## + amenities                        1  1055482 61156
    ## + leasing_rate:net                 1  1055484 61156
    ## + LEED:size                        1  1055484 61156
    ## + LEED:renovated                   1  1055485 61156
    ## + size:class_a                     1  1055485 61156
    ## + size:class_b                     1  1055485 61156
    ## - Electricity_Costs:total_dd_07    1  1056157 61158
    ## - size:stories                     1  1056160 61158
    ## - LEED:Electricity_Costs           1  1056237 61158
    ## - Electricity_Costs:renovated      1  1056292 61159
    ## - size:net                         1  1056332 61159
    ## - size:total_dd_07                 1  1056391 61159
    ## - class_a:stories                  1  1056531 61160
    ## - renovated:total_dd_07            1  1056713 61162
    ## - Electricity_Costs:stories        1  1056813 61162
    ## - Gas_Costs:total_dd_07            1  1056848 61163
    ## - leasing_rate:Precipitation       1  1057012 61164
    ## - total_dd_07:stories              1  1057051 61164
    ## - Electricity_Costs:leasing_rate   1  1057325 61166
    ## - Energystar:Precipitation         1  1057338 61166
    ## - Precipitation:class_b            1  1057462 61167
    ## - class_a:Precipitation            1  1057563 61168
    ## - size:renovated                   1  1057689 61169
    ## - Precipitation:Gas_Costs          1  1057859 61170
    ## - Gas_Costs:class_b                1  1058550 61175
    ## - class_a:Gas_Costs                1  1058700 61176
    ## - Gas_Costs:renovated              1  1059321 61181
    ## - renovated:stories                1  1059324 61181
    ## - leasing_rate:size                1  1059655 61184
    ## - Precipitation:stories            1  1062060 61202
    ## - Electricity_Costs:Gas_Costs      1  1063620 61213
    ## - Precipitation:total_dd_07        1  1063785 61214
    ## - class_b:total_dd_07              1  1064354 61219
    ## - class_a:total_dd_07              1  1064358 61219
    ## - Gas_Costs:stories                1  1064463 61219
    ## - Electricity_Costs:class_b        1  1069115 61254
    ## - Electricity_Costs:class_a        1  1069678 61258
    ## - Electricity_Costs:size           1  1073553 61286
    ## 
    ## Step:  AIC=61154.15
    ## Rent ~ LEED + Energystar + Electricity_Costs + class_a + leasing_rate + 
    ##     Precipitation + Gas_Costs + size + renovated + class_b + 
    ##     net + total_dd_07 + stories + Electricity_Costs:Gas_Costs + 
    ##     Electricity_Costs:Precipitation + Precipitation:Gas_Costs + 
    ##     Electricity_Costs:class_a + Electricity_Costs:size + leasing_rate:size + 
    ##     Electricity_Costs:class_b + Energystar:Precipitation + Gas_Costs:renovated + 
    ##     class_a:Precipitation + Precipitation:class_b + leasing_rate:Precipitation + 
    ##     Electricity_Costs:leasing_rate + Precipitation:total_dd_07 + 
    ##     size:total_dd_07 + class_a:total_dd_07 + class_b:total_dd_07 + 
    ##     total_dd_07:stories + Gas_Costs:stories + Electricity_Costs:stories + 
    ##     Precipitation:stories + Gas_Costs:total_dd_07 + renovated:stories + 
    ##     size:renovated + Precipitation:renovated + class_a:stories + 
    ##     Electricity_Costs:total_dd_07 + Energystar:Electricity_Costs + 
    ##     LEED:Electricity_Costs + renovated:total_dd_07 + Electricity_Costs:renovated + 
    ##     size:stories + class_a:Gas_Costs + Gas_Costs:class_b + net:total_dd_07 + 
    ##     size:net + net:stories + LEED:net + leasing_rate:stories
    ## 
    ##                                   Df Deviance   AIC
    ## <none>                                1055173 61154
    ## + leasing_rate:class_a             1  1054907 61154
    ## - Electricity_Costs:Precipitation  1  1055457 61154
    ## + stories:class_b                  1  1054948 61154
    ## - leasing_rate:stories             1  1055486 61154
    ## + LEED:Energystar                  1  1054957 61155
    ## + Energystar:size                  1  1054962 61155
    ## + LEED:total_dd_07                 1  1054963 61155
    ## - Energystar:Electricity_Costs     1  1055510 61155
    ## + Energystar:total_dd_07           1  1054982 61155
    ## + LEED:Precipitation               1  1054985 61155
    ## + leasing_rate:renovated           1  1054993 61155
    ## + Energystar:class_b               1  1054994 61155
    ## + LEED:leasing_rate                1  1055008 61155
    ## + Energystar:Gas_Costs             1  1055020 61155
    ## + net:Precipitation                1  1055039 61155
    ## + leasing_rate:Gas_Costs           1  1055046 61155
    ## + leasing_rate:class_b             1  1055055 61155
    ## + age                              1  1055058 61155
    ## + Energystar:class_a               1  1055060 61155
    ## + Energystar:renovated             1  1055062 61155
    ## + Energystar:net                   1  1055065 61155
    ## + size:Precipitation               1  1055070 61155
    ## + Energystar:stories               1  1055073 61155
    ## + LEED:class_b                     1  1055074 61155
    ## - LEED:net                         1  1055612 61155
    ## - net:stories                      1  1055630 61156
    ## + renovated:net                    1  1055097 61156
    ## + renovated:class_a                1  1055114 61156
    ## - Precipitation:renovated          1  1055650 61156
    ## + net:Electricity_Costs            1  1055123 61156
    ## - net:total_dd_07                  1  1055664 61156
    ## + class_b:net                      1  1055144 61156
    ## + leasing_rate:total_dd_07         1  1055152 61156
    ## + class_a:net                      1  1055155 61156
    ## + renovated:class_b                1  1055158 61156
    ## + net:Gas_Costs                    1  1055160 61156
    ## + LEED:stories                     1  1055163 61156
    ## + Energystar:leasing_rate          1  1055163 61156
    ## + LEED:class_a                     1  1055165 61156
    ## + LEED:Gas_Costs                   1  1055165 61156
    ## + amenities                        1  1055168 61156
    ## + LEED:size                        1  1055170 61156
    ## + size:Gas_Costs                   1  1055170 61156
    ## + leasing_rate:net                 1  1055171 61156
    ## + size:class_b                     1  1055172 61156
    ## + size:class_a                     1  1055172 61156
    ## + LEED:renovated                   1  1055172 61156
    ## - Electricity_Costs:total_dd_07    1  1055823 61157
    ## - size:stories                     1  1055873 61157
    ## - LEED:Electricity_Costs           1  1055930 61158
    ## - Electricity_Costs:renovated      1  1055979 61158
    ## - size:net                         1  1056005 61158
    ## - leasing_rate:size                1  1056060 61159
    ## - size:total_dd_07                 1  1056202 61160
    ## - Electricity_Costs:stories        1  1056240 61160
    ## - class_a:stories                  1  1056339 61161
    ## - renovated:total_dd_07            1  1056399 61161
    ## - Gas_Costs:total_dd_07            1  1056527 61162
    ## - leasing_rate:Precipitation       1  1056550 61162
    ## - total_dd_07:stories              1  1056581 61163
    ## - Electricity_Costs:leasing_rate   1  1057045 61166
    ## - Energystar:Precipitation         1  1057056 61166
    ## - Precipitation:class_b            1  1057139 61167
    ## - class_a:Precipitation            1  1057186 61167
    ## - size:renovated                   1  1057381 61169
    ## - Precipitation:Gas_Costs          1  1057586 61170
    ## - Gas_Costs:class_b                1  1058281 61175
    ## - class_a:Gas_Costs                1  1058401 61176
    ## - Gas_Costs:renovated              1  1059034 61181
    ## - renovated:stories                1  1059062 61181
    ## - Precipitation:stories            1  1061687 61201
    ## - Electricity_Costs:Gas_Costs      1  1063270 61212
    ## - Precipitation:total_dd_07        1  1063413 61214
    ## - class_a:total_dd_07              1  1064079 61218
    ## - class_b:total_dd_07              1  1064091 61219
    ## - Gas_Costs:stories                1  1064096 61219
    ## - Electricity_Costs:class_b        1  1068872 61254
    ## - Electricity_Costs:class_a        1  1069382 61258
    ## - Electricity_Costs:size           1  1073503 61288

    ## Warning: package 'finalfit' was built under R version 3.6.3

    ##  Dependent: Rent           unit       value     Coefficient (univariable)
    ##             LEED    0 Mean (sd) 28.4 (15.1)                             -
    ##                     1 Mean (sd) 29.7 (16.4)                             -
    ##       Energystar    0 Mean (sd) 28.3 (15.3)                             -
    ##                     1 Mean (sd) 30.1 (12.7)                             -
    ##     green_rating    0 Mean (sd) 28.3 (15.3)                             -
    ##                     1 Mean (sd) 30.0 (13.0)                             -
    ##             <NA> <NA>      <NA>        <NA>  1.80 (0.58 to 3.02, p=0.004)
    ##             <NA> <NA>      <NA>        <NA>  1.75 (0.57 to 2.93, p=0.004)
    ##             <NA> <NA>      <NA>        <NA> 1.29 (-2.74 to 5.33, p=0.531)
    ##       Coefficient (multivariable)
    ##                                 -
    ##                                 -
    ##                                 -
    ##                                 -
    ##                                 -
    ##                                 -
    ##    3.78 (-8.19 to 15.75, p=0.536)
    ##  -2.00 (-14.03 to 10.03, p=0.744)
    ##    2.95 (-8.28 to 14.17, p=0.607)

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
