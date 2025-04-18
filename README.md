# Defunding-Police-Crime-Thesis-Sample

*THIS CODE IS A SNIPPET OF THE ANALYSIS SECTION OF MY MASTER DO-FILE FOR MY THESIS ON THE EMPIRICAL RELATIONSHIP BETWEEN DEFUNDING POLICE AND CRIME. 
*MOST COMMANDS ARE COMMENTED OUT SO THEY WOULD NOT RUN ON TOP OF EACH OTHER.

/*CREATE A SUMMARY STATISTICS TABLE 
collapse (mean) OverallCrimeRate ViolentCR PropertyCR Officers UnempRate MedHHInc Poverty White Black Treatment P2, by(PoliceDept Year monthyear)

local append "replace"
foreach t in 1 0 {
	foreach p in 0 1 {
		outsum OverallCrimeRate ViolentCR PropertyCR Officers UnempRate MedHHInc Poverty White Black if Treatment==`t'&P2==`p' using ///
		TableMeans2.xls, `append' bracket
		local append "append"
		
	}
}  
*/


/*CREATE THE BALANCE TEST
collapse (mean) OverallCrimeRate ViolentCR PropertyCR Treatment Officers Post UnempRate MedHHInc Poverty White Black PDID, by(PoliceDept Year)
gen TreatPost = Treatment*Post

foreach var in OverallCrimeRate ViolentCR PropertyCR Officers UnempRate MedHHInc Poverty White Black {
    reg `var' TreatPost i.Year i.PDID, robust  
    outreg2 using BalanceTest.xls, excel ///
        addstat(Observations, e(N), R-squared, e(r2))
}
*/


/*TWFE GRAPH 
collapse (mean) OverallCrimeRate Officers_Lag UnempRate MedHHInc Poverty White Black (semean) se_OverallCrimeRate=OverallCrimeRate, by(PDID Year)
foreach var in OverallCrimeRate Officers_Lag {
	reg `var' UnempRate MedHHInc Poverty White Black i.Year i.PDID
	predict `var'R, resid
}

twoway (scatter OverallCrimeRateR Officers_LagR, mcolor(blue) msymbol(O)) ///
       (lfit OverallCrimeRateR Officers_LagR, lcolor(red) lwidth(medium)), ///
       xtitle("Residualized Number of Officers (per 1000 people) in Year Y-1") ///
       ytitle("Residualized Crimes Committed per 100,000 people per Month (Averaged by Year)") ///
       title("Crime Rate vs. Lagged Officers") ///
       legend(order(1 "Cities by Year" 2 "Fitted Line") col(1) region(lcolor(none)))


*/


/*DID AVERAGES GRAPH
collapse (mean) OverallCrimeRate (semean) se_OverallCrimeRate=OverallCrimeRate, by(Treatment Year)
gen upper= OverallCrimeRate+1.96*se_OverallCrimeRate
gen lower= OverallCrimeRate-1.96*se_OverallCrimeRate
 
gr twoway (line OverallCrimeRate Year if Treatment==1, lcolor(blue)) ///
          (line OverallCrimeRate Year if Treatment==0, lcolor(red)) ///
          (rcap upper lower Year if Treatment==1, lc(blue)) ///
          (rcap upper lower Year if Treatment==0, lc(red)), ///
          xline(2010, lp(dash)) ///
          ytitle("Crimes Committed per 100,000 people per Month (Averaged by Year)") ///
          title("Average Crime Rate by City Type", size(medium)) ///
          legend(order(1 "Treatment Cities" 2 "Comparison Cities") ///
                 col(1) region(lcolor(none)))

*/


/*DID COEFFICIENTS GRAPH: EVENT STUDY MODEL
collapse (mean) OverallCrimeRate Officers Treatment Post TreatPost UnempRate MedHHInc Poverty White Black, by(PDID Year) 

*generate year dummies
foreach y of numlist 2015/2023 {
    gen Year`y' = (Year == `y')
}

*generate interaction terms between treatment and year 
foreach y of numlist 2021/2023 {
    gen Treat_Year`y' = Treatment * Year`y'
}

foreach y of numlist 2015/2019 {
    gen Treat_Year`y' = Treatment * Year`y'
}
*write the regression 
reg OverallCrimeRate Treat_Year* Treatment UnempRate MedHHInc Poverty White Black i.Year, robust
gen coef = 0 
gen se = . 
foreach Year in 2015 2016 2017 2018 2019 2021 2022 2023 {
	replace coef = _b[Treat_Year`Year'] if Year==`Year'
	replace se = _se[Treat_Year`Year'] if Year==`Year'
}
egen tag = tag(Year)
keep if tag==1 
keep Year coef se

*graph the coefficients 
gen ci_upper = coef + 1.96 * se
gen ci_lower = coef - 1.96 * se

twoway (scatter coef Year, msize(vsmall) mcolor(blue)) ///
       (rcap ci_upper ci_lower Year, lcolor(blue)), ///
       xtitle("Year") ytitle("Estimated Effect on Overall Monthly Crimes per 100,000 people") ///
       title("Event Study: Effect of Police on Crime Over Time") ///
       legend(off) xline(2020, lcolor(red) lpattern(dash)) ///
       graphregion(color(white))

gen crime_coef = .
gen crime_se = .
foreach Year in 2015 2016 2017 2018 2019 2021 2022 2023 {
    replace crime_coef = _b[Treat_Year`Year'] if Year == `Year'
    replace crime_se = _se[Treat_Year`Year'] if Year == `Year'
}

*/


/*DID AVERAGES GRAPH: OFFICERS
collapse (mean) Officers (semean) se_Officers=Officers, by(Treatment Year)
gen upper= Officers+1.96*se_Officers
gen lower= Officers-1.96*se_Officers
 
gr twoway (line Officers Year if Treatment==1, lcolor(blue)) ///
          (line Officers Year if Treatment==0, lcolor(red)) ///
          (rcap upper lower Year if Treatment==1, lc(blue)) ///
          (rcap upper lower Year if Treatment==0, lc(red)), ///
          xline(2010, lp(dash)) ///
          ytitle("Number of Officers per 1000 people per Year") ///
          title("Average Officers by City Type", size(medium)) ///
          legend(order(1 "Treatment Cities" 2 "Comparison Cities") ///
                 col(1) region(lcolor(none)))

*/


/*DID COEFFICIENTS GRAPH FOR OFFICERS
collapse (mean) Officers Treatment Post TreatPost UnempRate MedHHInc Poverty White Black, by(PDID Year) 

*generate year dummies
foreach y of numlist 2015/2023 {
    gen Year`y' = (Year == `y')
}

*generate interaction terms between treatment and year 
foreach y of numlist 2021/2023 {
    gen Treat_Year`y' = Treatment * Year`y'
}

foreach y of numlist 2015/2019 {
    gen Treat_Year`y' = Treatment * Year`y'
}
*write the regression 
reg Officers Treat_Year* Treatment UnempRate MedHHInc Poverty White Black i.Year, robust
gen coef = 0 
gen se = . 
foreach Year in 2015 2016 2017 2018 2019 2021 2022 2023 {
	replace coef = _b[Treat_Year`Year'] if Year==`Year'
	replace se = _se[Treat_Year`Year'] if Year==`Year'
}
egen tag = tag(Year)
keep if tag==1 
keep Year coef se

*graph the coefficients 
gen ci_upper = coef + 1.96 * se
gen ci_lower = coef - 1.96 * se

twoway (scatter coef Year, msize(vsmall) mcolor(blue)) ///
       (rcap ci_upper ci_lower Year, lcolor(blue)), ///
       xtitle("Year") ytitle("Estimated Effect on Average Yearly Officers per 1000 People") ///
       title("Event Study: Effect of Treatment Status on Police Over Time") ///
       legend(off) xline(2020, lcolor(red) lpattern(dash)) ///
       graphregion(color(white))
	   
gen officer_coef = .
gen officer_se = .
foreach Year in 2015 2016 2017 2018 2019 2021 2022 2023 {
    replace officer_coef = _b[Treat_Year`Year'] if Year == `Year'
    replace officer_se = _se[Treat_Year`Year'] if Year == `Year'
}
	   
*/


/*CREATE MAIN OLS RESULTS TABLE 
collapse (mean) OverallCrimeRate ViolentCR PropertyCR Officers_Lag Treatment Post UnempRate MedHHInc Poverty White Black, by(PDID Year) 

* OLS no controls or fe 
reg OverallCrimeRate Officers_Lag, robust
outreg2 using OLSTable.xls, addstat(Observations, e(N), R-squared, e(r2)) replace 

* OLS with fe
reg OverallCrimeRate Officers_Lag i.Year i.PDID, robust
outreg2 using OLSTable.xls, addstat(Observations, e(N), R-squared, e(r2)) append

*OLS with controls and fe 
reg OverallCrimeRate Officers_Lag UnempRate MedHHInc Poverty White Black i.Year i.PDID, robust
outreg2 using OLSTable.xls, addstat(Observations, e(N), R-squared, e(r2)) append

*OLS with controls and fe (VIOLENT)
reg ViolentCR Officers_Lag UnempRate MedHHInc Poverty White Black i.Year i.PDID, robust
outreg2 using OLSTable.xls, addstat(Observations, e(N), R-squared, e(r2)) append

*OLS with controls and fe (PROPERTY)
reg PropertyCR Officers_Lag UnempRate MedHHInc Poverty White Black i.Year i.PDID, robust
outreg2 using OLSTable.xls, addstat(Observations, e(N), R-squared, e(r2)) append

*/


/*CREATE MAIN DID RESULTS TABLE 
collapse (mean) OverallCrimeRate ViolentCR PropertyCR Treatment Post TreatPost UnempRate MedHHInc Poverty White Black, by(PDID Year) 

* DID no controls or fe
reg OverallCrimeRate Treatment Post TreatPost, robust 
outreg2 using DIDTable.xls, addstat(Observations, e(N), R-squared, e(r2)) replace 

*DID with fe 
reg OverallCrimeRate TreatPost i.Year i.PDID, robust 
outreg2 using DIDTable.xls, addstat(Observations, e(N), R-squared, e(r2)) append 

*DID with controls and fe 
reg OverallCrimeRate TreatPost UnempRate MedHHInc Poverty White Black i.Year i.PDID, robust 
outreg2 using DIDTable.xls, addstat(Observations, e(N), R-squared, e(r2)) append 

*DID with controls and fe (VIOLENT)
reg ViolentCR TreatPost UnempRate MedHHInc Poverty White Black i.Year i.PDID, robust 
outreg2 using DIDTable.xls, addstat(Observations, e(N), R-squared, e(r2)) append 

*DID with controls and fe (PROPERTY)
reg PropertyCR TreatPost UnempRate MedHHInc Poverty White Black i.Year i.PDID, robust 
outreg2 using DIDTable.xls, addstat(Observations, e(N), R-squared, e(r2)) append 


*/



/* CREATE OFFICERS DID RESULTS TABLE 
collapse (mean) Officers Treatment Post TreatPost UnempRate MedHHInc Poverty White Black, by(PDID Year) 

* DID no controls or fe
reg Officers Treatment Post TreatPost, robust 
outreg2 using OfficerTable.xls, addstat(Observations, e(N), R-squared, e(r2)) replace 

*DID with fe 
reg Officers TreatPost i.Year i.PDID, robust 
outreg2 using OfficerTable.xls, addstat(Observations, e(N), R-squared, e(r2)) append 

* DID with controls and fe 
reg Officers TreatPost UnempRate MedHHInc Poverty White Black i.Year i.PDID, robust 
outreg2 using OfficerTable.xls, addstat(Observations, e(N), R-squared, e(r2)) append 

*/


/* CREATE TWFE LAGGED TABLE 
bysort PoliceDept (Year): gen Officers_Lag = Officers[_n-1]
bysort PoliceDept (Year): gen Officers_Lag2 = Officers[_n-2]
bysort PoliceDept (Year): gen Officers_Lag3 = Officers[_n-3]

collapse (mean) OverallCrimeRate ViolentCR PropertyCR Officers Officers_Lag Officers_Lag2 Officers_Lag3 UnempRate MedHHInc Poverty White Black, by(PDID Year) 

*no lag
reg OverallCrimeRate Officers UnempRate MedHHInc Poverty White Black i.Year i.PDID, robust
outreg2 using TWFEPlaceboTable.xls, addstat(Observations, e(N), R-squared, e(r2)) replace

*1 year lag
reg OverallCrimeRate Officers_Lag UnempRate MedHHInc Poverty White Black i.Year i.PDID, robust
outreg2 using TWFEPlaceboTable.xls, addstat(Observations, e(N), R-squared, e(r2)) append

*2 year lag
reg OverallCrimeRate Officers_Lag2 UnempRate MedHHInc Poverty White Black i.Year i.PDID, robust
outreg2 using TWFEPlaceboTable.xls, addstat(Observations, e(N), R-squared, e(r2)) append

*3 year lag 
reg OverallCrimeRate Officers_Lag3 UnempRate MedHHInc Poverty White Black i.Year i.PDID, robust
outreg2 using TWFEPlaceboTable.xls, addstat(Observations, e(N), R-squared, e(r2)) append

*/
