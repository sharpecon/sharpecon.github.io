/*==============================================================================

Author: 55027

Date: 6th May 2023

Task: Code of EC2C1 Final Report on Political Competition, Policy and Growth


==============================================================================*/


clear all 
set more off, perm
cap log close 					// close any log file in use

log using "55027_project.log", replace text


* Markers may have to change this codeblock
cd "C:\Users\Neha\Downloads\Metrics-Project\Option2"

use "C:\Users\Neha\Downloads\Metrics-Project\Option2\project2.dta", clear




*For producing results in Table1

reg share_taxes_inc compnorm i.stcode i.year, cluster(stcode)
reg share_cap_exp   compnorm i.stcode i.year, cluster(stcode)
reg rtw             compnorm i.stcode i.year, cluster(stcode)

reg share_taxes_inc compnorm i.stcode i.year so#i.year, cluster(stcode)
reg share_cap_exp   compnorm i.stcode i.year so#i.year, cluster(stcode)
reg rtw             compnorm i.stcode i.year so#i.year, cluster(stcode)

ivregress 2sls share_taxes_inc i.stcode i.year (compnorm = coreiv), cluster(stcode)
ivregress 2sls share_cap_ex    i.stcode i.year (compnorm = coreiv), cluster(stcode)
ivregress 2sls rtw             i.stcode i.year (compnorm = coreiv), cluster(stcode)

*For producing results in Table 2
reg share_taxes_inc compnorm gip demcontrol repcontrol so#i.year i.stcode i.year, cluster(stcode)
reg share_cap_exp   compnorm gip demcontrol repcontrol so#i.year i.stcode i.year, cluster(stcode)
reg rtw             compnorm gip demcontrol repcontrol so#i.year i.stcode i.year, cluster(stcode)

*reg share_taxes_inc compnorm gip demcontrol repcontrol normdem so#i.year i.stcode i.year, cluster(stcode)
*reg share_cap_exp   compnorm gip demcontrol repcontrol normdem so#i.year i.stcode i.year, cluster(stcode)
*reg rtw             compnorm gip demcontrol repcontrol normdem so#i.year i.stcode i.year, cluster(stcode)

reg share_taxes_inc demcomp repcomp gip demcontrol repcontrol so#i.year i.stcode i.year, cluster(stcode)
reg share_cap_exp   demcomp repcomp gip demcontrol repcontrol so#i.year i.stcode i.year, cluster(stcode)
reg rtw             demcomp repcomp gip demcontrol repcontrol so#i.year i.stcode i.year, cluster(stcode)

*reg share_taxes_inc com1 com2 com3 gip demcontrol repcontrol normdem so#i.year i.stcode i.year, cluster(stcode)
*reg share_cap_exp   com1 com2 com3 gip demcontrol repcontrol normdem so#i.year i.stcode i.year, cluster(stcode)
*reg rtw             com1 com2 com3 gip demcontrol repcontrol normdem so#i.year i.stcode i.year, cluster(stcode)

 
*reg share_taxes_inc compnorm gip demcontrol repcontrol i.stcode i.year if so==1, cluster(stcode)
*reg share_cap_exp   compnorm gip demcontrol repcontrol so#i.year i.stcode i.year if so==1, cluster(stcode)
*reg rtw             compnorm gip demcontrol repcontrol so#i.year i.stcode i.year if so==1, cluster(stcode)

*reg share_taxes_inc compnorm gip demcontrol repcontrol i.stcode i.year if so == 0, cluster(stcode)
*reg share_cap_exp   compnorm gip demcontrol repcontrol so#i.year i.stcode i.year if so == 0, cluster(stcode)
*reg rtw             compnorm gip demcontrol repcontrol so#i.year i.stcode i.year if so == 0, cluster(stcode)

* Columns 7 and 
preserve 

keep if year>=1950 & year<=1999
gen e = floor(year/5)
tostring e, gen(ex)
gen groupid = code + ex

bysort groupid: egen fiveyravg_share_taxes_inc = mean(share_taxes_inc)
bysort groupid: egen fiveyravg_share_cap_exp   = mean(share_cap_exp)
bysort groupid: egen fiveyravg_compnorm        = mean(compnorm)
bysort groupid: egen fiveyravg_gip             = mean(gip)
bysort groupid: egen fiveyravg_demcontrol      = mean(demcontrol)
bysort groupid: egen fiveyravg_repcontrol      = mean(repcontrol)

reg fiveyravg_share_taxes_inc fiveyravg_compnorm fiveyravg_gip fiveyravg_demcontrol fiveyravg_repcontrol i.stcode i.year so#i.year, cluster(stcode)
reg fiveyravg_share_cap_exp   fiveyravg_compnorm fiveyravg_gip fiveyravg_demcontrol fiveyravg_repcontrol i.stcode i.year so#i.year, cluster(stcode)

restore 

* Column 9
preserve

keep if year>=1932 & year<=2001
gen e = floor(year/5)
tostring e, gen(ex)
gen groupid = code + ex

bysort groupid: egen fiveyravg_rtw             = mean(rtw)
bysort groupid: egen fiveyravg_compnorm        = mean(compnorm)
bysort groupid: egen fiveyravg_gip             = mean(gip)
bysort groupid: egen fiveyravg_demcontrol      = mean(demcontrol)
bysort groupid: egen fiveyravg_repcontrol      = mean(repcontrol)

reg fiveyravg_rtw fiveyravg_compnorm fiveyravg_gip fiveyravg_demcontrol fiveyravg_repcontrol i.stcode i.year so#i.year, cluster(stcode)

restore


*For producing results in Table3

reg gstinc compnorm llstinc i.stcode i.year, cluster(stcode)
reg gstinc compnorm llstinc i.stcode i.year so#i.year, cluster(stcode)
ivregress 2sls gstinc llstinc i.stcode i.year (compnorm = coreiv), cluster(stcode)
ivregress 2sls gstinc llstinc i.stcode i.year so#i.year (compnorm = coreiv), cluster(stcode)


*reg gstinc compnorm gip demcontrol repcontrol llstinc so#i.year i.stcode i.year, cluster(stcode)
*reg gstinc compnorm gip demcontrol repcontrol normdem llstinc so#i.year i.stcode i.year, cluster(stcode)
*reg gstinc demcomp repcomp gip demcontrol repcontrol llstinc so#i.year i.stcode i.year, cluster(stcode)
*reg gstinc com1 com2 com3 gip demcontrol repcontrol normdem llstinc so#i.year i.stcode i.year, cluster(stcode)


*reg gstinc compnorm gip demcontrol repcontrol llstinc so#i.year i.stcode i.year if so == 1, cluster(stcode)
*reg gstinc compnorm gip demcontrol repcontrol llstinc so#i.year i.stcode i.year if so == 0, cluster(stcode)
*reg non_farm_share compnorm gip demcontrol repcontrol i.stcode i.year so#i.year if year>=1929 and year<=2000, cluster(stcode)


*For producing results in Table4
ivregress 2sls gstinc llstinc i.stcode i.year (share_taxes_inc = compnorm), cluster(stcode) 
ivregress 2sls gstinc llstinc i.stcode i.year (share_cap_exp = compnorm), cluster(stcode) level(90)
ivregress 2sls gstinc llstinc i.stcode i.year (rtw = compnorm), cluster(stcode) level(90)

*For checking the t-stat values from first-stage regression (checking IV model assumptions)
reg share_taxes_inc    compnorm llstinc i.stcode i.year, cluster(stcode)
reg share_cap_exp   compnorm llstinc i.stcode i.year, cluster(stcode)
reg rtw    compnorm llstinc i.stcode i.year, cluster(stcode)


*For producing results in Table5
reg share_taxes_inc compnorm i.stcode i.year [aweight=pop], cluster(stcode)
reg share_cap_exp   compnorm i.stcode i.year [aweight=pop], cluster(stcode)
reg rtw             compnorm i.stcode i.year [aweight=pop], cluster(stcode)

*To check if more populous states have an effect on share_cap_exp which is opposite from the effect observed nationally
reg share_cap_exp   compnorm i.stcode i.year if stcode == 4 | stcode == 8 | stcode == 30 | stcode == 41





***Testing the parallel trends assumption

*Time trend of political competition (figure 1(d))

preserve

tostring so, gen(sostr)
tostring year, gen(yearstr)

gen groupid = sostr + yearstr
bysort groupid: egen avg_compnorm   = mean(compnorm)
duplicates drop groupid, force

keep year so avg_compnorm
reshape wide avg_compnorm, i(year) j(so)
drop if missing(avg_compnorm0)

line avg_compnorm0 avg_compnorm1 year, legend(size(medsmall)) ytitle(Political Competition) xtitle(Year) legend(label(1 "Northern States") label(2 "Southern States"))
graph export dd_validity_compnorm.jpg, quality(100)

restore

*Time trend of share_taxes_inc (figure 1(b))
 
preserve

tostring so, gen(sostr)
tostring year, gen(yearstr)

gen groupid = sostr + yearstr
bysort groupid: egen avg_share_taxes_inc   = mean(share_taxes_inc)
duplicates drop groupid, force

keep year so avg_share_taxes_inc
reshape wide avg_share_taxes_inc, i(year) j(so)
drop if missing(avg_share_taxes_inc0)

line avg_share_taxes_inc0 avg_share_taxes_inc1 year, legend(size(medsmall)) tline(1965) ytitle(Tax revenue as % of state income) xtitle(Year) legend(label(1 "Northern States") label(2 "Southern States"))
graph export dd_validity_avg_shares_taxes_inc.jpg, quality(100)

restore

*Time trend of share_cap_exp (figure 1(a))

preserve

tostring so, gen(sostr)
tostring year, gen(yearstr)

gen groupid = sostr + yearstr
bysort groupid: egen avg_share_cap_exp  = mean(share_cap_exp)
duplicates drop groupid, force

keep year so avg_share_cap_exp
reshape wide avg_share_cap_exp, i(year) j(so)
drop if missing(avg_share_cap_exp0)

line avg_share_cap_exp0 avg_share_cap_exp1 year, legend(size(medsmall)) tline(1965) ytitle(Infra. spending as a % of state govt. exp.) xtitle(Year) legend(label(1 "Northern States") label(2 "Southern States"))
graph export dd_validity_avg_share_cap_exp.jpg, quality(100)

restore

*Time trend of rtw (figure 1(c))

preserve

tostring so, gen(sostr)
tostring year, gen(yearstr)

gen groupid = sostr + yearstr
bysort groupid: egen avg_rtw   = mean(rtw)
duplicates drop groupid, force

keep year so avg_rtw
reshape wide avg_rtw, i(year) j(so)
drop if missing(avg_rtw0)

line avg_rtw0 avg_rtw1 year, legend(size(medsmall)) tline(1965) ytitle(Right to work laws) xtitle(Year) legend(label(1 "Northern States") label(2 "Southern States"))
graph export dd_validity_avg_rtw.jpg, quality(100)

restore



*Time trend of gstinc ( not used in the final submission)

preserve

tostring so, gen(sostr)
tostring year, gen(yearstr)

gen groupid = sostr + yearstr
bysort groupid: egen avg_gstinc   = mean(gstinc)
duplicates drop groupid, force

replace avg_gstinc = avg_gstinc*100

keep year so avg_gstinc
reshape wide avg_gstinc, i(year) j(so)
drop if missing(avg_gstinc0)

line avg_gstinc0 avg_gstinc1 year, legend(size(medsmall)) tline(1965) ytitle(Growth(in %) of personal income) xtitle(Year) legend(label(1 "Northern States") label(2 "Southern States"))
graph export dd_validity_avg_gstinc.jpg, quality(100)

restore


*Extension2 (including state specific trends) (a potential success)

*reg share_taxes_inc compnorm i.stcode i.year i.stcode#c.year, cluster(stcode)
*reg share_cap_exp   compnorm i.stcode i.year i.stcode#c.year, cluster(stcode)
*reg rtw             compnorm i.stcode i.year i.stcode#c.year, cluster(stcode)


  

log off