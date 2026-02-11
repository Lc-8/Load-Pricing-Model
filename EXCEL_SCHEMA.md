# Excel Workbook Schema — Advanced Load Pricing Model.xlsx

Generated: **2026-02-11T02:46:35Z**  
Scan window: **A1:AZ100** (for formula/content audit)

## Sheet inventory

| Sheet | Tables | Non-empty cells (A1:AZ100) | Formulas (A1:AZ100) |
|---|---:|---:|---:|
| Quick Estimate | 0 | 112 | 43 |
| README | 0 | 7 | 0 |
| Assumptions | 0 | 131 | 1 |
| Accessorials | 1 | 92 | 0 |
| Rate Cards | 1 | 304 | 108 |
| Fuel Surcharge | 1 | 342 | 163 |
| Load Input | 1 | 155 | 75 |
| Cost Model | 0 | 62 | 28 |
| Pricing Summary | 1 | 114 | 56 |
| Blank 6 Read Me | 0 | 11 | 0 |
| Blank 6 Assumptions | 0 | 43 | 0 |
| Blank 6 Rate Card | 0 | 32 | 0 |
| Blank 6 Loads | 1 | 2668 | 2619 |
| Blank 6 Load Quote | 0 | 43 | 19 |
| Blank 6 Dashboard | 0 | 59 | 47 |

## Excel tables (preferred parsing targets)

### Accessorials
- **tblAccessorials** — `A3:G15`
  - Headers: `#`, `Accessorial`, `Default fee`, `Unit`, `Included`, `Overage fee`, `Notes`

### Rate Cards
- **tblRateCards** — `A6:I42`
  - Headers: `Equipment type`, `Miles band start`, `Miles band end`, `Reference rate per loaded mile`, `Minimum linehaul`, `Notes`, `EquipKey`, `BandStartNum`, `BandEndNum`

### Fuel Surcharge
- **tblFuelSurcharge** — `A13:E94`
  - Headers: `Fuel price (from)`, `Fuel price (to)`, `Fuel surcharge per mile`, `Computation`, `Notes`

### Load Input
- **tblLoadAccessorials** — `A43:H55`
  - Headers: `Include?`, `Accessorial`, `Quantity`, `Unit`, `Default fee`, `Included`, `Overage fee`, `Calculated charge`

### Pricing Summary
- **tblCostBreakdown** — `A49:C61`
  - Headers: `Cost category`, `Amount`, `Percent of price`

### Blank 6 Loads
- **B6_LoadsTable** — `A3:AV123`
  - Headers: `Load ID`, `Load date`, `Customer`, `Equipment type`, `Origin`, `Destination`, `Loaded miles`, `Deadhead miles`, `Fuel price (optional)`, `Base rate per loaded mile (optional)`, `Minimum linehaul (optional)`, `Detention`, `Layover`, `Tolls`, `Lumper fees`, `Tarp fee`, `Other accessorial fees`, `Other extra fees`, `Accessorial markup percent (optional)`, `Risk premium percent of cost (optional)`, `Notes`, `Total miles`, `Fuel price used`, `Base rate used`, `Minimum linehaul used`, `Market linehaul`, `Fuel surcharge per mile`, `Fuel surcharge total`, `Accessorial fees at cost`, `Accessorial charge (with markup)`, `Estimated trip hours`, `Estimated trip days`, `Fuel cost`, `Variable cost excluding fuel`, `Fixed cost allocated`, `Subtotal cost`, `Contingency cost`, `Risk premium cost`, `Total cost`, `Market total price`, `Profit at market price`, `Profit margin at market price`, `Recommended total price (cost-based)`, `Recommended linehaul`, `Profit at recommended price`, `Profit margin at recommended price`, `Recommended price per loaded mile`, `Recommended price per total mile`

## Defined names (named ranges) — stable keys for parsing

### Assump_* (32)
- `Assump_BrokerPct` → `Assumptions!B35`
- `Assump_CardPct` → `Assumptions!B36`
- `Assump_DEFPer1000` → `Assumptions!B17`
- `Assump_DEFPrice` → `Assumptions!B16`
- `Assump_DieselPrice` → `Assumptions!B15`
- `Assump_EquipmentType` → `Assumptions!B5`
- `Assump_HoursPerDay` → `Assumptions!B8`
- `Assump_InsurancePerDay` → `Assumptions!B31`
- `Assump_MPG_Deadhead` → `Assumptions!B11`
- `Assump_MPG_Loaded` → `Assumptions!B10`
- `Assump_MaintDeadhead` → `Assumptions!B19`
- `Assump_MaintLoaded` → `Assumptions!B18`
- `Assump_MinCharge` → `Assumptions!B46`
- `Assump_OverheadPerDay` → `Assumptions!B34`
- `Assump_PayModel` → `Assumptions!B25`
- `Assump_PayPctLinehaul` → `Assumptions!B28`
- `Assump_PayPerHour` → `Assumptions!B27`
- `Assump_PayPerMile` → `Assumptions!B26`
- `Assump_PermitsPerDay` → `Assumptions!B32`
- `Assump_PricingMethod` → `Assumptions!B41`
- `Assump_RiskPct` → `Assumptions!B44`
- `Assump_SpeedDeadhead` → `Assumptions!B7`
- `Assump_SpeedLoaded` → `Assumptions!B6`
- `Assump_StopHours` → `Assumptions!B9`
- `Assump_TargetMargin` → `Assumptions!B42`
- `Assump_TargetMarkup` → `Assumptions!B43`
- `Assump_TechPerDay` → `Assumptions!B33`
- `Assump_Tires` → `Assumptions!B20`
- `Assump_TollsPerMile` → `Assumptions!B21`
- `Assump_TrailerPerDay` → `Assumptions!B30`
- `Assump_TruckPerDay` → `Assumptions!B29`
- `Assump_UrgencyPct` → `Assumptions!B45`

### B6_* (16)
- `B6_AverageSpeed` → `Blank 6 Assumptions!$B$7`
- `B6_BaseFuelPrice` → `Blank 6 Assumptions!$B$5`
- `B6_ContingencyPercent` → `Blank 6 Assumptions!$B$11`
- `B6_DefaultAccessorialMarkup` → `Blank 6 Assumptions!$B$16`
- `B6_DefaultFuelPrice` → `Blank 6 Assumptions!$B$4`
- `B6_DwellHours` → `Blank 6 Assumptions!$B$8`
- `B6_EquipmentList` → `Blank 6 Assumptions!$A$22:$A$26`
- `B6_FixedCostPerDay` → `Blank 6 Assumptions!$B$10`
- `B6_FuelEconomy` → `Blank 6 Assumptions!$B$6`
- `B6_FuelSurchargeAppliesTo` → `Blank 6 Assumptions!$B$17`
- `B6_MinPrice` → `Blank 6 Assumptions!$B$15`
- `B6_MinProfit` → `Blank 6 Assumptions!$B$14`
- `B6_OverheadPercent` → `Blank 6 Assumptions!$B$12`
- `B6_RateCardRange` → `Blank 6 Rate Card!$A$6:$E$10`
- `B6_TargetProfitMargin` → `Blank 6 Assumptions!$B$13`
- `B6_VariableCostPerMile` → `Blank 6 Assumptions!$B$9`

### Cost_* (7)
- `Cost_Accessorials` → `Cost Model!B42`
- `Cost_Days` → `Cost Model!B8`
- `Cost_FixedTotal` → `Cost Model!B34`
- `Cost_PassThrough` → `Cost Model!B41`
- `Cost_TotalBeforeFees` → `Cost Model!B44`
- `Cost_TotalHours` → `Cost Model!B7`
- `Cost_VariableOps` → `Cost Model!B38`

### Fuel_* (3)
- `Fuel_BaselinePrice` → `Fuel Surcharge!B6`
- `Fuel_BaselineRate` → `Fuel Surcharge!B7`
- `Fuel_MPG` → `Fuel Surcharge!B9`

### Load_* (17)
- `Load_BaseRateOverride` → `Load Input!B22`
- `Load_EquipmentNeeded` → `Load Input!$B$10`
- `Load_Escort` → `Load Input!B36`
- `Load_LaneFactorPct` → `Load Input!B25`
- `Load_LoadedMiles` → `Load Input!B15`
- `Load_MarketAdjPct` → `Load Input!B24`
- `Load_MinLinehaulOverride` → `Load Input!B26`
- `Load_OtherFees` → `Load Input!B39`
- `Load_Parking` → `Load Input!B38`
- `Load_Permits` → `Load Input!B35`
- `Load_ReferenceRate` → `Load Input!B23`
- `Load_RiskOverride` → `Load Input!B29`
- `Load_Scales` → `Load Input!B37`
- `Load_Tolls` → `Load Input!B34`
- `Load_TotalDeadhead` → `Load Input!B18`
- `Load_TotalMiles` → `Load Input!B19`
- `Load_UrgencyOverride` → `Load Input!B30`

### Price_* (1)
- `Price_FinalQuote` → `Pricing Summary!B32`
