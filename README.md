# ckaa173 
# Examining Gender Differentials in the Association of Low Control Work with Cognitive Performance in Older Workers
Code by Katherine Ford for doi:10.1093/eurpub/ckaa173

Please cite this paper when using the code: 

Ford, K. J., Batty, G. D., & Leist, A. K. (2021). Examining gender differentials in the association of low control work with cognitive performance in older workers. European Journal of Public Health, 31(1), 174-180.

## Data Availability
This paper uses publicly available data for research purposes from SHARE Waves
1, 2, 4, 5, 6 and the All Waves Coverscreen (DOI: 10.6103/SHARE.w1.700, 10.6103/SHARE.w2.700, 10.6103/SHARE.w4.700, 10.6103/SHARE.w5.700, 10.6103/SHARE.w6.700, 10.6103/SHARE.wXcvr.700). The datasets analyzed are accessible through the SHARE Research Data Center: <https://shareproject.centerdata.nl/sharedatadissemination/users/login>.
## Funding
K.J.F.’s doctoral training was supported by the Luxembourg National Research Fund under Grant 10949242. The research also received funding from the European Research Council (ERC) under the European Union’s Horizon 2020 research and innovation programme (grant agreement no. 803239, to A.K.L.). The SHARE data collection has been funded by the European Commission through FP5 (QLK6-CT-2001-00360), FP6 (SHARE-I3: RII-CT-2006-062193, COMPARE: CIT5-CT-2005-028857, SHARELIFE: CIT4-CT-2006-028812), FP7 (SHARE-PREP: GA No. 211909, SHARE-LEAP: GA No. 227822, SHARE M4: GA No. 261982) and Horizon 2020 (SHARE-DEV3: GA No. 676536, SERISS: GA No. 654221) and by DG Employment, Social Affairs & Inclusion. Additional funding from the German Ministry of Education and Research, the Max Planck Society for the Advancement of Science, the U.S. National Institute on Aging (U01_AG09740-13S2, P01_AG005842, P01_AG08291, P30_AG12815, R21_AG025169, Y1-AG-4553-01, IAG_BSR06-11, OGHA_04-064 and HHSN271201300071C) and from various national funding sources is gratefully acknowledged.
## Main Code Details

### Variable Descriptions
`lowctrlgrp` : Composite variable from `ep029_` and `ep030_` in the original work files
```
generate ctrl= ep030_ + (5-ep029_) if ep029_<=4 & ep030_<=4
gen lowctrlgrp=.
local wave "1 2 4 5 6"
foreach x of local wave {
bys country: egen tertctrl`x'= xtile(ctrl) if wave==`x', n(3)
replace lowctrlgrp = tertctrl`x' if wave==`x'
}
```
`newid` : `mergeid` in the original data sets

`gender` : From the all waves coverscreen file

`age_int` : From the all waves coverscreen file

`cohort` : Recode of `yrbirth` from the all waves coverscreen
```
gen cohort=1 if yrbirth <1945
replace cohort=2 if yrbirth>=1945 & yrbirth <1950
replace cohort=3 if yrbirth>=1950 & yrbirth <1955
replace cohort=4 if yrbirth>=1955
```
`country` : From the original demographics file

`hidemgrp` : Composite variable from `ep027_` and `ep028_` in the original work files
```
generate demand= (5-ep027) + (5-ep028) if ep027<=4 & ep028<=4 
gen hidemgrp=.
local wave "1 2 4 5 6"
foreach x of local wave {
egen tertdem`x'= xtile(demand) if wave==`x', n(3)
replace hidemgrp = tertdem`x' if wave==`x'
}
```
`privpubself` : `ep009` in the original work files
`jobsecure` : Recode of `ep035_` from the original work files
	`generate jobsecure_=1 if ep035_==3 | ep035_==4  
replace jobsecure_=0 if ep035_==1 | ep035_==2`

`hourgrp` : Recoding of `ep013_` from the original work files
```
  generate hourgrp = 1 if ep013_<30 & ep013_!=.
replace hourgrp = 2 if ep013_>=30 & ep013_<55
replace hourgrp = 3 if ep013_>=55 & ep013_!=.
```

`quininc` : Recode of `thinc` from the imputations file
```
gen thinc=(thinc1+thinc2+thinc3+thinc4+thinc5)/5
tab eq_hhsize country, m 	//only Israel has missing info for hhsize
tab eq_hhsize wave if country==25, m 	//they are all missing in wave1
bys mergeid (wave): replace eq_hhsize = eq_hhsize[_n+1] if country==25 & wave==1
gen inchh=thinc/eq_hhsize	
bys country: egen quininc= xtile (inchh) if wave==1, n(5)
bys country: egen quininc2= xtile (inchh) if wave==2, n(5)
bys country: egen quininc4= xtile (inchh) if wave==4, n(5)
bys country: egen quininc5= xtile (inchh) if wave==5, n(5)
bys country: egen quininc6= xtile (inchh) if wave==6, n(5)
replace quininc = quininc2 if wave==2 & quininc==.
replace quininc = quininc4 if wave==4 & quininc==.
replace quininc = quininc5 if wave==5 & quininc==.
replace quininc = quininc6 if wave==6 & quininc==.
```

`edugrp` : Recode of `isced` from the imputations file
```
gen isced=(isced1+isced2+isced3+isced4+isced5)/5
	generate edugrp=.
tab isced if firstwave==1, m
replace edugrp=1 if isced<=float(1) 
replace edugrp=2 if isced>=float(2) & isced<=float(3) 
replace edugrp=3 if isced>=float(4) & isced<=float(6)  
bysort mergeid (wave) : replace edugrp= edugrp[1]
```

`cohab` : Composite variable from `dn014_` and `dn044_` in the original demographics files
```
bysort mergeid (wave) : gen marstat= dn014_[1] 
gen dn044 = 0
replace dn044=1 if dn044_==1
gen lagdn044 = 0
bys mergeid (wave): replace lagdn044= dn044[_n-1]
gen lag2dn044 = 0
bys mergeid (wave): replace lag2dn044= dn044[_n-2]
gen lag3dn044 = 0
bys mergeid (wave): replace lag3dn044= dn044[_n-3]
gen marchange=0
replace marchange=1 if dn044==1 | lagdn044==1 | lag2dn044==1 | lag3dn044==1
gen cohab=1 if (marstat==1 | marstat==2) & marchange==0      
replace cohab=0 if marstat>2 & marstat<. & marchange==0      
replace cohab=1 if marstat>2 & marstat<. & marchange==1      
replace cohab=0 if (marstat==1 | marstat==2) & marchange==1
```

`hearloss` : Composite variable from `ph045_` and `ph046_` in the original physical health files
```
gen hearloss=0 if ph045_!=. & ph046_!=. 
replace hearloss=1 if ph045_==1 | ph046_==5
```

`chron` : Composite variable from `ph011d6` and `ph011d2` in the original physical health files
```
rename ph011d6 txdiab
rename ph011d2 txbp
gen chron=1 if txdiab==1 | txbp==1
replace chron=0 if txdiab==0 & txbp==0
```

`depress` : Recode of `eurod` from the generated health variables files
```
gen depress=1 if eurod>3 & eurod!=. 
replace depress=0 if eurod<=3 & eurod!=.
```

`phactiv` : From the generated health variables files

`bmi` :  Recode of `bmi2` from the generated health variables files 
```
 tab bmi2
recode bmi2 (1/2=1) (3=2) (4=3), gen(bmi)
```

`smoke` : Composite variable from `br002_` and `br001_` in the original risky behaviours files
```
rename br002_ cusmoke
rename br001_ eversmoke
recode eversmoke (1=1) (5=0)
 	bys mergeid (wave): replace eversmoke=eversmoke[1]
  	gen smoke=eversmoke			
  	replace smoke=0 if eversmoke==0 & cusmoke==5
 	replace smoke=1 if eversmoke==1 & cusmoke==5		
  	replace smoke=2 if cusmoke==1
```

`z_verb` : Standardizing of `cf010_` variable from the original cognition files 
```
  su raw_verb if wave==firstwave 	
gen z_verb = ((raw_verb - r(mean))/r(sd))
```

`z_imm` : Stardardizing of `cf008tot` variable from the original cognition files and the generated health variables files for wave 4, 5, and 6
```
su raw_imm if wave==firstwave  
gen z_imm = ((raw_imm - r(mean))/r(sd))
```

`z_delay` : Standardizing of `cf016tot` variable from the original cognition files and the generated health variables files for wave 4, 5, and 6
```
su raw_delay if wave==firstwave	
gen z_delay = ((raw_delay-r(mean))/r(sd))
```

### Main Analysis Code
#### MEN 
```
local cog "z_verb z_delay z_imm"
foreach x of local cog {
```
**Base** 
```
xtreg `x' ib2.lowctrlgrp age_int if gender==1, fe i(newid)
est store m_`x'_base
```
**Model 1**
```
xtreg `x' ib2.lowctrlgrp age_int i.hidemgrp i.privpubself i.jobsecure i.hourgrp i.quininc if gender==1, fe i(newid)
est store m_`x'_m1
```
**Model 2**
```
xtreg `x' ib2.lowctrlgrp age_int i.hidemgrp i.privpubself i.jobsecure i.hourgrp i.quininc cohab hearloss chron depress phactiv i.bmi i.smoke if gender==1, fe i(newid)
est store m_`x'_m2
```
**Random Effects**
```
xtreg `x' ib2.lowctrlgrp age_int i.hidemgrp i.privpubself i.jobsecure i.hourgrp i.quininc cohab hearloss chron depress phactiv i.bmi i.smoke i.country i.edugrp i.cohort if gender==1, re i(newid)
est store m_`x'_re
hausman m_`x'_m2 m_`x'_re
}
```
		
#### WOMEN 
```
local cog "z_verb z_delay z_imm"
foreach x of local cog {
```
**Base**
```
xtreg `x' ib2.lowctrlgrp age_int if gender==2, fe i(newid)
est store f_`x'_base
```
**Model 1**
```
xtreg `x' ib2.lowctrlgrp age_int i.hidemgrp i.privpubself i.jobsecure i.hourgrp i.quininc if gender==2, fe i(newid)
est store f_`x'_m1
```
**Model 2**
```
xtreg `x' ib2.lowctrlgrp age_int i.hidemgrp i.privpubself i.jobsecure i.hourgrp i.quininc cohab hearloss chron depress phactiv i.bmi i.smoke if gender==2, fe i(newid)
est store f_`x'_m2
```
**Random Effects**
```
xtreg `x' ib2.lowctrlgrp age_int i.hidemgrp i.privpubself i.jobsecure i.hourgrp i.quininc cohab hearloss chron depress phactiv i.bmi i.smoke i.country i.edugrp i.cohort if gender==2, re i(newid)
est store f_`x'_re 
hausman f_`x'_m2 f_`x'_re
}
```

#### POOLED 
```
local cog "z_verb z_delay z_imm"
foreach x of local cog {
```
**Base**
```
xtreg `x' ib2.lowctrlgrp##i.gender age_int , fe i(newid)
est store p_`x'_base
```
**Model 1**
```
xtreg `x' ib2.lowctrlgrp##i.gender age_int i.hidemgrp i.privpubself i.jobsecure i.hourgrp i.quininc, fe i(newid)
est store p_`x'_m1
```
**Model 2**
```
xtreg `x' ib2.lowctrlgrp##i.gender age_int i.hidemgrp i.privpubself i.jobsecure i.hourgrp i.quininc cohab hearloss chron depress phactiv i.bmi i.smoke, fe i(newid)
est store p_`x'_m2
```
**Random Effects**
```
xtreg `x' ib2.lowctrlgrp##i.gender age_int i.hidemgrp i.privpubself i.jobsecure i.hourgrp i.quininc cohab hearloss chron depress phactiv i.bmi i.smoke i.country i.edugrp i.cohort, re i(newid)
est store p_`x'_re
hausman p_`x'_m2 p_`x'_re
}
```
