# Credit Default Swap Pricer
Credit Default Swap Pricer project brings together the [ISDA CDS pricer](http://www.cdsmodel.com/cdsmodel/) and some new IMM date modules that are needed to make quick use of the underlying C library functions. This wrapper is aimed at analysts whom want to get up and running very quickly to price and compute risk on CDS using either Python or C++ calling code. The measures computed support a range of potential analysis including:

 + PVDirty, PVClean & Accrued Interest to support NAV calculations & back tests.
 + CS01 & DV01 sensitivities for risk exposure & limit monitoring analysis.
 + PVBP sensitivities to support credit risk hedging analysis.

Potential future measures might include Equivalent Notional, Par Spread and Risky CS01, these measures are likely to be added as part of the next full release candidate.

## Why create another CDS Pricing library?

The idea behind this library is ease of use, the underlying [ISDA C functions](http://www.cdsmodel.com/cdsmodel/) whilst usable are pretty difficult to integrate and often folks revert to either [3rd party](https://www.google.co.uk/search?q=fincad+cds+pricer&oq=fincad+cds+pricer&aqs=chrome..69i57j0.3457j0j7&sourceid=chrome&ie=UTF-8) or [open source CDS pricing libraries](http://quantlib.org/index.shtml). Whilst this is fine for most uses; when you need precision pricing quickly and easily that conforms exactly to the ISDA CDS model then this wrapper allows you to very quickly build and start writing code Python and price and compute risk on real CDS positions.

1. Is this not just another CDS pricer?

   This library is really only a thin wrapper around the underlying [ISDA CDS Pricing library](http://www.cdsmodel.com/cdsmodel/). The complexity of wiring the spread, interest rate and pricing routines together with some array passing and imm date logic completes an integration task. None of these steps is particularly difficult but together they build a barrier to adoption of the ISDA CDS pricer. By making this library available to use along side the existing [ISDA CDS pricer](http://www.cdsmodel.com/cdsmodel/) it is hoped to lower the barrier and make adoption much easier.

2. Is the only system that can model the weather really only the weather?

   If what you need is safe accurate ISDA pricing then why settle for anything other than the ISDA pricer? however using this CDS pricer avoids the hastle of figuring out all the correct C functions to call and how to pass objects easily into these extern "C" style functions with double* and variety of custom typedef objects like TDateInterval. I just want to create a datetime and pass this into a function right!

## How do I get started? 

The module can be downloaded along with a suitable version of the [ISDA CDS Pricing library](http://www.cdsmodel.com/cdsmodel/) using the make.sh script to invoke the swig and gcc builds needed to generate and compile the wrapper and underlying code modules. The g++ invoke is also managed by this file which in turn builds the C++ wrapper ahead of linking the entire module into a library called isda. This libray can then be easily imported into the Python C runtime as shown below.

```python
from isda import cds_all_in_one
```

### CDS All In One

Once you have downloaded and built the project a simple function cds_all_in_one will provide a swig wrapped C++ function that invokes the underlying C library functions from the [ISDA CDS model](http://www.cdsmodel.com/cdsmodel/). The interface has been constructed to make the usage as simple and easy as possible. Python native types are used and no custom objects are used. 


```python

from isda import cds_all_in_one
from imm import imm_date_vector

# EUR interest rate curve
swap_rates = [-0.00369, -0.00341, -0.00328, -0.00274, -0.00223, -0.00186, 
            -0.00128, 0.00046, 0.00217, 0.003, 0.00504,
            0.00626, 0.00739, 0.00844, 0.00941, 0.01105, 
            0.01281, 0.01436, 0.01506]

swap_tenors = ['1M', '2M', '3M', '6M', '9M', '1Y', '2Y', '3Y', 
            '4Y', '5Y', '6Y', '7Y', '8Y', '9Y, 
            '10Y', '15Y', '20Y', '30Y']

# spread curve
credit_spreads = [0.00141154155739384] * 8
credit_spread_tenors = ['6M', '1Y', '2Y', '3Y', '4Y', '5Y', '7Y', '10Y']

# value asofdate
sdate = datetime.datetime(2018, 1, 23)
value_date = sdate.strftime('%d/%m/%Y')

# economics of trade
recovery_rate = 0.4
coupon = 100
trade_date = '14/12/2017'
effective_date = '20/09/2017'
maturity_date = '20/12/2019'
accrual_start_date = '20/09/2017'
notional = 1.0 # 1MM EUR
is_buy_protection = 0

tenor_list = [0.5, 1, 2, 3, 4, 5, 7, 10]

# build imm_dates
imm_dates = [f[1] for f in imm_date_helper(start_date=sdate,
                                           tenor_list=tenor_list)]

value_date = sdate.strftime('%d/%m/%Y')
pv_dirty, pv_clean, ai, cs01, dv01, 
    pvbp6m, pvbp1y, pvbp2y, pvbp3y, pvbp4y, 
    pvbp5y, pvbp7y, pvbp10y, duration_in_milliseconds 
            = cds_all_in_one(trade_date,
                   effective_date,
                   maturity_date,
                   value_date,
                   accrual_start_date,
                   recovery_rate,
                   coupon,
                   notional,
                   is_buy_protection,
                   swap_rates,
                   swap_tenors,
                   credit_spreads,
                   credit_spread_tenors,
                   imm_dates,
                   verbose)
```

#### Pricing & Risk Measures ####

The cds_all_in_one function call returns a tuple of measures in a positional format, these are detailed as below.

+ pv_dirty - net present value of the CDS, including accrued interest from the current coupon period.
+ cs01 - change in net present value of the CDS, based on a parallel shift of 1bps across the whole CDS spread curve.
+ dv01 - change in net present value of the CDS, based on a parallel shift of 1bps across the whole Interest Rate curve.
+ pvbp6m - present value of a basis point based on a 1bps shift of 6M IMM tenor date.
+ pvbp1y - present value of a basis point based on a 1bps shift of 1Y IMM tenor date.
+ pvbp2y - present value of a basis point based on a 1bps shift of 2Y IMM tenor date.
+ pvbp3y - present value of a basis point based on a 1bps shift of 3y IMM tenor date.
+ pvbp4y - present value of a basis point based on a 1bps shift of 4Y IMM tenor date.
+ pvbp5y - present value of a basis point based on a 1bps shift of 5Y IMM tenor date.
+ pvbp7y - present value of a basis point based on a 1bps shift of 7Y IMM tenor date.
+ pvbp10y - present value of a basis point based on a 1bps shift of 10Y IMM tenor date.
+ duration_in_milliseconds - total wall time in terms of execution of the routine

### IMM CDS Dates

Quite often the first hurdle when computing anything realted to CDS contracts is how to compute and make available accurate [IMM dates](https://en.wikipedia.org/wiki/IMM_dates) that play nicely with all CDS contracts and business date rules? For this reason this module ships with an imm_date_helper class that takes all the effort away. 

1. imm_date_helper 

   The imm_date_helper function has been written and tested for the explicit purpose of providing accurate IMM dates that comply fully with the CDS market convention. Using the imm_date_helper function you can easily bootstrap the necessary IMM date vector for any business date. 
   
2. [semi-annual roll](https://www.isda.org//2015/12/10/updated-faq-amend-single-name-on-the-run-frequency) 
 
   Since 2015 IMM date logic for CDS contracts has changed to a [semi-annual roll](https://www.isda.org//2015/12/10/updated-faq-amend-single-name-on-the-run-frequency); this change impacts all future tenors along the CDS curve and should be accurately applied to ensure consistent CDS contract pricing.
   
#### Example Semi-Annual IMM Date Roll

The example below shows how the IMM date roll logic is embedded accurately into the helper based on the semi annual roll, with a before and after roll date vector generated along the entire swap curve tenors. If you are pricing and need IMM dates before the ISDA 2015 semi annual roll change then this is automatially applied in the helper function. The function looks at the value of start_date parameter to determine if this latest rule needs to be applied.


```python
    def test_single_rolldate_day_before_rolldate(self):

        # accepted results
        real_result = [('6M', '20/06/2017'),
                   ('1Y', '20/12/2017'),
                   ('2Y', '20/12/2018'),
                   ('3Y', '20/12/2019'),
                   ('5Y', '20/12/2021'),
                   ('7Y', '20/12/2023')]

        sdate = datetime.datetime(2017, 3, 17)
        tenor_list = [0.5, 1, 2, 3, 5, 7]
        local_result = imm_date_helper(start_date=sdate,
                                 tenor_list=tenor_list,
                                 format='%d/%m/%Y')


        for (r,l) in zip(real_result, local_result):
            self.assertTrue(r[0] == l[0] and r[1] == l[1])

    def test_single_rolldate_day_after_rolldate(self):

        # accepted results
        real_result = [('6M', '20/12/2017'),
                   ('1Y', '20/06/2018'),
                   ('2Y', '20/06/2019'),
                   ('3Y', '20/06/2020'),
                   ('5Y', '20/06/2022'),
                   ('7Y', '20/06/2024')]

        sdate = datetime.datetime(2017, 3, 20)
        tenor_list = [0.5, 1, 2, 3, 5, 7]
        local_result = imm_date_helper(start_date=sdate,
                                 tenor_list=tenor_list,
                                 format='%d/%m/%Y')

        for (r,l) in zip(real_result, local_result):
            self.assertTrue(r[0] == l[0] and r[1] == l[1])
            
```
