# Engine Spec — CFA Load Pricing Engine

This document maps the Excel workbook logic to backend code modules. The backend **re-implements** formulas deterministically (Excel is config, not runtime compute).

## Workbook→Engine mapping

### Mode 1: Quick Estimate (sheet `Quick Estimate`)
The Quick Estimate sheet contains an output block where labels are in **column D** and computed values are in **column E**.

| Row | Label (D) | Excel formula/value (E) | Engine stage |
|---:|---|---|---|
| 22 | Total miles | `=B12+B13` | |
| 23 | Estimated days | `=MAX(1,IFERROR(ROUNDUP(((B12/'Assumptions'!$B$6)+(B13/'Assumptions'!$B$7)+(2*'Assumptions'!$B$9))/'Assumptions'!$B$8,…` | |
| 24 | Market reference rate per loaded mile (Rate Cards) | `=IFERROR(SUMPRODUCT(('Rate Cards'!$D$7:$D$42)*(--(TRIM(SUBSTITUTE(SUBSTITUTE(SUBSTITUTE('Rate Cards'!$A$7:$A$42,CHAR(…` | |
| 25 | Market minimum linehaul (Rate Cards) | `=IFERROR(SUMPRODUCT(('Rate Cards'!$E$7:$E$42)*(--(TRIM(SUBSTITUTE(SUBSTITUTE(SUBSTITUTE('Rate Cards'!$A$7:$A$42,CHAR(…` | |
| 26 | Market linehaul (loaded miles × market rate, with minimum) | `=MAX($B$12*E24,E25)` | |
| 27 | Market fuel surcharge per mile (lookup) | `=MAX(0,IFERROR('Fuel Surcharge'!$B$7 + (E29-'Fuel Surcharge'!$B$6)/'Fuel Surcharge'!$B$9,0))` | |
| 28 | Market fuel surcharge total (loaded miles) | `=B12*E27` | |
| 29 | Diesel price used | `=IF(B17>0,B17,IFERROR('Assumptions'!$B$15,3.85))` | |
| 30 | Target margin used | `=IF(B18<>"",B18,IFERROR('Assumptions'!$B$42,0.22))` | |
| 31 | Contingency percent used | `=IF(B19<>"",B19,IFERROR('Assumptions'!$B$44,0.03))` | |
| 32 | Fuel cost (loaded + deadhead) | `=IFERROR(((B12/'Assumptions'!$B$10)+(B13/'Assumptions'!$B$11))*E29,0)` | |
| 33 | Diesel exhaust fluid cost | `=IFERROR((E22*'Assumptions'!$B$17/1000)*'Assumptions'!$B$16,0)` | |
| 34 | Maintenance + tires + baseline toll estimate | `=IFERROR((B12*'Assumptions'!$B$18)+(B13*'Assumptions'!$B$19)+(E22*'Assumptions'!$B$20)+(E22*'Assumptions'!$B$21),0)` | |
| 35 | Driver pay (expense) | `=IF(Assumptions!B25="Owner-operator (no separate wage)",0,IF(Assumptions!B25="Dollars per mile",$E$22*Assumptions!B26…` | |
| 36 | Allocated fixed cost | `=IFERROR(('Assumptions'!$B$29+'Assumptions'!$B$30+'Assumptions'!$B$31+'Assumptions'!$B$32+'Assumptions'!$B$33+'Assump…` | |
| 37 | Total cost (before contingency) | `=SUM(E32,E33,E34,E35,E36)` | |
| 38 | Contingency amount | `=E37*E31` | |
| 39 | Cost + contingency | `=E37+E38` | |
| 40 | Estimated operating cost (expenses) | `=E37` | |
| 41 | Estimated cost + contingency (expenses) | `=E39` | |
| 42 | Cost-based estimate (before minimum + other) | `=IFERROR(IF('Assumptions'!$B$41="Target margin on price",E37/(1-E30),E37*(1+'Assumptions'!$B$43)),0)` | |
| 43 | Market-based estimate (market linehaul + market fuel surcharge) × (1+contingency) | `=(E26+E28)*(1+E31)` | |
| 44 | Minimum load charge | `=IFERROR('Assumptions'!$B$46,350)` | |
| 45 | Estimate after minimum (choose higher of cost-based vs market-based) | `=MAX(E42,E43,E44)` | |
| 46 | FINAL ROUGH ESTIMATE (includes Other) | `=E45+$B$14` | |
| 47 | All-in rate per loaded mile | `=IF(B12>0,E42/B12,0)` | |
| 48 | All-in rate per total mile | `=IF(E22>0,E42/E22,0)` | |

**Notes:**
- Rate card lookup uses `SUMPRODUCT` with equipment normalization (replacing em/en dashes).
- Fuel surcharge total uses a lookup from `Fuel Surcharge` schedule plus loaded miles.
- Owner-operator handling: the workbook can set driver pay to 0 for owner-op view; the engine should support **both** profit views.

### Mode 2: Full Quote (sheets `Load Input`, `Cost Model`, `Pricing Summary`)
This is the detailed quote pipeline. `Load Input` collects inputs and accessorial toggles, `Cost Model` computes costs, and `Pricing Summary` builds the final quote.

#### Cost Model key formulas (col A label → col B value)
| Row | Label | Excel formula/value |
|---:|---|---|
| 4 | Driving hours (loaded) | `=IFERROR(Load_LoadedMiles/Assump_SpeedLoaded,"")` |
| 5 | Driving hours (deadhead) | `=IFERROR(Load_TotalDeadhead/Assump_SpeedDeadhead,"")` |
| 6 | Stop hours (pickup and delivery) | `=IFERROR(2*Assump_StopHours,"")` |
| 7 | Total estimated hours | `=IFERROR(SUM(B4:B6),"")` |
| 8 | Estimated transit days | `=IFERROR(MAX(1,ROUNDUP(B7/Assump_HoursPerDay,0)),"")` |
| 12 | Fuel gallons (loaded) | `=IFERROR(Load_LoadedMiles/Assump_MPG_Loaded,"")` |
| 13 | Fuel gallons (deadhead) | `=IFERROR(Load_TotalDeadhead/Assump_MPG_Deadhead,"")` |
| 14 | Fuel cost | `=IFERROR((B12+B13)*Assump_DieselPrice,"")` |
| 15 | Diesel exhaust fluid gallons | `=IFERROR(Load_TotalMiles*Assump_DEFPer1000/1000,"")` |
| 16 | Diesel exhaust fluid cost | `=IFERROR(B15*Assump_DEFPrice,"")` |
| 17 | Maintenance and repairs cost | `=IFERROR(Load_LoadedMiles*Assump_MaintLoaded + Load_TotalDeadhead*Assump_MaintDeadhead,"")` |
| 18 | Tires cost | `=IFERROR(Load_TotalMiles*Assump_Tires,"")` |
| 19 | Baseline tolls and road fees (per-mile estimate) | `=IFERROR(Load_TotalMiles*Assump_TollsPerMile,"")` |
| 23 | Driver pay model | `=IFERROR(Assump_PayModel,"")` |
| 24 | Driver pay cost | `=IFERROR(IF(Assump_PayModel="Dollars per mile",Load_TotalMiles*Assump_PayPerMile,IF(Assump_PayModel="Dollars per hour",Cost_TotalHours*As…` |
| 28 | Truck payment / depreciation allocation | `=IFERROR(Assump_TruckPerDay*Cost_Days,"")` |
| 29 | Trailer payment / depreciation allocation | `=IFERROR(Assump_TrailerPerDay*Cost_Days,"")` |
| 30 | Insurance allocation | `=IFERROR(Assump_InsurancePerDay*Cost_Days,"")` |
| 31 | Permits and registrations allocation | `=IFERROR(Assump_PermitsPerDay*Cost_Days,"")` |
| 32 | Technology allocation | `=IFERROR(Assump_TechPerDay*Cost_Days,"")` |
| 33 | Office overhead allocation | `=IFERROR(Assump_OverheadPerDay*Cost_Days,"")` |
| 34 | Total allocated fixed costs | `=IFERROR(SUM(B28:B33),"")` |
| 38 | Total variable operating costs | `=IFERROR(SUM(B14:B19),"")` |
| 39 | Driver cost | `=IFERROR(B24,"")` |
| 40 | Allocated fixed costs | `=IFERROR(Cost_FixedTotal,"")` |
| 41 | Direct pass-through fees | `=IFERROR(SUM(Load_Tolls,Load_Permits,Load_Escort,Load_Scales,Load_Parking,Load_OtherFees),"")` |
| 42 | Accessorial charges | `=IFERROR(SUM(tblLoadAccessorials[Calculated charge]),"")` |
| 44 | Total fully burdened cost (excluding broker/card) | `=IFERROR(SUM(B38,B39,B40,B41,B42),"")` |

#### Pricing Summary key formulas (col A label → col B value)
| Row | Label | Excel formula/value |
|---:|---|---|
| 4 | Base linehaul rate per loaded mile (used) | `=IFERROR(IF(Load_BaseRateOverride>0,Load_BaseRateOverride,Load_ReferenceRate),"")` |
| 5 | Loaded miles | `=IFERROR(Load_LoadedMiles,"")` |
| 6 | Base linehaul (before adjustments) | `=IFERROR(B4*B5,"")` |
| 7 | Adjusted linehaul (market and lane factors) | `=IFERROR(B6*(1+Load_MarketAdjPct)*(1+Load_LaneFactorPct),"")` |
| 8 | Fuel surcharge per mile (lookup) | `=IFERROR(LOOKUP(Assump_DieselPrice,tblFuelSurcharge[Fuel price (from)],tblFuelSurcharge[Fuel surcharge per mile]),0)` |
| 9 | Fuel surcharge total (loaded miles) | `=IFERROR(B8*Load_LoadedMiles,"")` |
| 13 | Minimum linehaul charge (used) | `=IFERROR(IF(Load_MinLinehaulOverride<>"",Load_MinLinehaulOverride,Assump_MinCharge),"")` |
| 14 | Linehaul after minimum check | `=IFERROR(MAX(B7,B13),"")` |
| 15 | Accessorials total | `=IFERROR(Cost_Accessorials,"")` |
| 16 | Direct pass-through fees total | `=IFERROR(Cost_PassThrough,"")` |
| 17 | Risk contingency percent (used) | `=IFERROR(IF(Load_RiskOverride<>"",Load_RiskOverride,Assump_RiskPct),"")` |
| 18 | Urgency premium percent (used) | `=IFERROR(IF(Load_UrgencyOverride<>"",Load_UrgencyOverride,Assump_UrgencyPct),"")` |
| 19 | Subtotal (linehaul + fuel + accessorials + pass-through) | `=IFERROR(SUM(B14,B9,B15,B16),"")` |
| 20 | Risk amount | `=IFERROR(B19*B17,"")` |
| 21 | Urgency amount | `=IFERROR(B19*B18,"")` |
| 22 | Subtotal after risk and urgency | `=IFERROR(B19+B20+B21,"")` |
| 26 | Total fully burdened cost (excluding broker/card) | `=IFERROR(Cost_TotalBeforeFees,"")` |
| 27 | Pricing method | `=IFERROR(Assump_PricingMethod,"")` |
| 28 | Recommended price before broker and card fees | `=IFERROR(IF(Assump_PricingMethod="Target margin on price",B26/(1-Assump_TargetMargin),B26*(1+Assump_TargetMarkup)),"")` |
| 29 | Broker / factoring percent | `=IFERROR(Assump_BrokerPct,"")` |
| 30 | Credit card processing percent | `=IFERROR(Assump_CardPct,"")` |
| 31 | Recommended price including broker and card fees | `=IFERROR(IF((B29+B30)>0,B28/(1-(B29+B30)),B28),"")` |
| 32 | Final quoted price (max of subtotal and cost-based recommendation) | `=IFERROR(MAX(B22,B31),"")` |
| 36 | All-in rate per loaded mile | `=IFERROR(IF(Load_LoadedMiles>0,Price_FinalQuote/Load_LoadedMiles,0),"")` |
| 37 | All-in rate per total mile | `=IFERROR(IF(Load_TotalMiles>0,Price_FinalQuote/Load_TotalMiles,0),"")` |
| 38 | Estimated profit (price - cost - broker - card) | `=IFERROR(Price_FinalQuote - Cost_TotalBeforeFees - (Price_FinalQuote*B29) - (Price_FinalQuote*B30),"")` |
| 39 | Estimated profit margin | `=IFERROR(IF(Price_FinalQuote>0,B38/Price_FinalQuote,0),"")` |
| 41 | Pricing narrative (copy into your quote) | `=IFERROR(TEXTJOIN(" ",TRUE,"Loaded miles:",TEXT(Load_LoadedMiles,"#,##0"),"Deadhead miles:",TEXT(Load_TotalDeadhead,"#,##0"),"Linehaul:",…` |
| 49 | Cost category | `Amount` |
| 50 | Fuel cost | `=IFERROR('Cost Model'!B14,"")` |
| 51 | Diesel exhaust fluid cost | `=IFERROR('Cost Model'!B16,"")` |
| 52 | Maintenance and repairs | `=IFERROR('Cost Model'!B17,"")` |
| 53 | Tires | `=IFERROR('Cost Model'!B18,"")` |
| 54 | Baseline toll estimate | `=IFERROR('Cost Model'!B19,"")` |
| 55 | Driver pay | `=IFERROR('Cost Model'!B24,"")` |
| 56 | Allocated fixed costs | `=IFERROR(Cost_FixedTotal,"")` |
| 57 | Direct pass-through fees | `=IFERROR(Cost_PassThrough,"")` |
| 58 | Accessorial charges | `=IFERROR(Cost_Accessorials,"")` |
| 59 | Broker / factoring fee | `=IFERROR(Price_FinalQuote*Assump_BrokerPct,"")` |
| 60 | Credit card processing fee | `=IFERROR(Price_FinalQuote*Assump_CardPct,"")` |
| 61 | Total (cost + fees) | `=IFERROR(SUM(B50:B60),"")` |
| 65 | Current diesel price | `=IFERROR(Assump_DieselPrice,"")` |
| 66 | What-if diesel price (input) | `3.85` |
| 67 | What-if target margin (input) | `0.22` |
| 68 | What-if fuel cost | `=IFERROR(('Cost Model'!B12+'Cost Model'!B13)*B66,"")` |
| 69 | What-if total cost (swap fuel only) | `=IFERROR(Cost_TotalBeforeFees - 'Cost Model'!B14 + B68,"")` |
| 70 | What-if recommended price (margin method) | `=IFERROR(B69/(1-B67),"")` |

### Mode 3: Blank 6 table engine (sheet `Blank 6 Loads`)
The `B6_LoadsTable` table contains both inputs and computed columns. The backend should be able to parse this table and compute the same derived values.

Key computed columns (first data row formulas):

| Column | Formula (row 4) | Meaning |
|---|---|---|
| Total miles | `=IF(OR(G4="",H4=""),"",G4+H4)` | |
| Fuel price used | `=IF(I4="",B6_DefaultFuelPrice,I4)` | |
| Base rate used | `=IF(J4="",IFERROR(VLOOKUP(D4,'Blank 6 Rate Card'!$A$6:$E$10,2,FALSE),""),J4)` | |
| Minimum linehaul used | `=IF(K4="",IFERROR(VLOOKUP(D4,'Blank 6 Rate Card'!$A$6:$E$10,3,FALSE),""),K4)` | |
| Market linehaul | `=IF(G4="","",MAX(G4*X4,Y4))` | |
| Fuel surcharge per mile | `=IF(G4="","",MAX(0,(W4-B6_BaseFuelPrice)/B6_FuelEconomy))` | |
| Fuel surcharge total | `=IF(G4="","",IF(B6_FuelSurchargeAppliesTo="All miles",AA4*V4,AA4*G4))` | |
| Accessorial fees at cost | `=IF(G4="","",(L4+M4+N4+O4+P4+Q4)+R4)` | |
| Accessorial charge (with markup) | `=IF(G4="","",AC4*(1+IF(S4="",B6_DefaultAccessorialMarkup,S4)))` | |
| Estimated trip hours | `=IF(V4="","",V4/B6_AverageSpeed + B6_DwellHours)` | |
| Estimated trip days | `=IF(AE4="","",AE4/24)` | |
| Fuel cost | `=IF(V4="","",(V4/B6_FuelEconomy)*W4)` | |
| Variable cost excluding fuel | `=IF(V4="","",V4*B6_VariableCostPerMile)` | |
| Fixed cost allocated | `=IF(AF4="","",AF4*B6_FixedCostPerDay)` | |
| Subtotal cost | `=IF(V4="","",AG4+AH4+AI4+AC4)` | |
| Contingency cost | `=IF(AJ4="","",AJ4*B6_ContingencyPercent)` | |
| Risk premium cost | `=IF(AJ4="","",(AJ4+AK4)*IF(T4="",0,T4))` | |
| Total cost | `=IF(AJ4="","",AJ4+AK4+AL4)` | |
| Market total price | `=IF(Z4="","",Z4+AB4+AD4)` | |
| Profit at market price | `=IF(AN4="","",AN4-(AM4+B6_OverheadPercent*AN4))` | |
| Profit margin at market price | `=IF(AN4="","",IF(AN4=0,"",AO4/AN4))` | |
| Recommended total price (cost-based) | `=IF(AM4="","",MAX(MAX(B6_MinPrice,AM4/(1-B6_OverheadPercent-B6_TargetProfitMargin)),Y4+AB4+AD4))` | |
| Recommended linehaul | `=IF(AQ4="","",AQ4-AB4-AD4)` | |
| Profit at recommended price | `=IF(AQ4="","",AQ4-(AM4+B6_OverheadPercent*AQ4))` | |
| Profit margin at recommended price | `=IF(AQ4="","",IF(AQ4=0,"",AS4/AQ4))` | |
| Recommended price per loaded mile | `=IF(G4="","",IF(G4=0,"",AQ4/G4))` | |
| Recommended price per total mile | `=IF(V4="","",IF(V4=0,"",AQ4/V4))` | |

## Implementation rules

1. **Never depend on Excel formula evaluation on the server.** Use the formulas above as reference and compute in Python.
2. Prefer parsing **Excel Tables** and **Defined Names** over hard-coded cell addresses.
3. Emit warnings for missing bands, missing config, or suspicious inputs.
4. Store request/response + config hash for audit and ML dataset creation.
