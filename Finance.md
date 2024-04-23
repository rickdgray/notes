---
title: Finance
lastmod: 2024-04-23T14:26:19-05:00
---
# Finance
## Investing/Retirement
### NPER
Age when I will reach target amount given starting amount and monthly payments
note pmt and pv are negative
```
=(NPER(0.05 / 12, -100000 / 12, -150000, 2500000) / 12) + 31
```

### PMT
Minimum payment needed to reach target amount given target retirement age and starting age
note pv and result are negative
```
=-PMT(0.05/12, (45-31)*12, -150000, 2500000) * 12
```

### PV
Minimum starting amount to reach target amount given monthly payments and retirement age
note pmt and result are negative
```
=-PV(0.05/12, (45-31)*12, -100000/12, 2500000)
```

### FV
Amount you will have at end of annuity given starting amount
note pmt and pv are negative
```
=FV(0.05/12, (45-31)*12, -100000/12, -150000)
```

### RATE
Annual ROI from stock market needed to reach target goal
note pmt and pv are negative
```
=RATE((45-31)*12, -100000/12, -150000, 2500000) * 12
```

## Loan
### NPER

### PMT

### PV

### FV

### RATE
