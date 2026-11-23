# Business questions / Preguntas de negocio
 
**Phase:** Ask (Google Data Analytics capstone)
**Date:** 2026-11-23
 
## Business task / Tarea de negocio
 
**EN:** Understand how renewables (solar + wind) and gas prices drive the Spanish day-ahead electricity price, and when electricity is cheapest, so that a retailer, an energy-intensive business or a consumer on the regulated PVPC tariff can decide when to consume.
 
**ES:** Entender cómo las renovables (solar + eólica) y el precio del gas influyen en el precio del mercado diario de electricidad en España, y cuándo es más barata la electricidad, para que una comercializadora, una empresa con alto consumo o un consumidor con PVPC pueda decidir cuándo consumir.
 
**Stakeholders:** electricity retailer, energy-intensive business, household consumer (PVPC).
 
## Questions / Preguntas
 
### Q1 / P1: Renewables and price
 
**EN:** Between 2022 and 2025, in the Spanish peninsular system, how does the average day-ahead price (€/MWh) change with the solar + wind share of hourly generation, grouping hours into 20-point bins ([0, 20), [20, 40), [40, 60), [60, 80), [80, 100] %), and does this relationship change from year to year?
 
**ES:** Entre 2022 y 2025, en el sistema peninsular español, ¿cómo cambia el precio medio del mercado diario (€/MWh) según el peso de la solar + eólica en la generación de cada hora, agrupando las horas en tramos de 20 puntos ([0, 20), [20, 40), [40, 60), [60, 80), [80, 100] %), y cambia esa relación de un año a otro?
 
### Q2 / P2: When is electricity cheapest
 
**EN:** Between 2022 and 2025, in the Spanish peninsular system, at which hours of the day, days of the week and months is the average day-ahead price (€/MWh) lowest, and have the cheapest hours of the day shifted from year to year?
 
**ES:** Entre 2022 y 2025, en el sistema peninsular español, ¿en qué horas del día, días de la semana y meses es más bajo el precio medio del mercado diario (€/MWh), y han cambiado las horas más baratas del día de un año a otro?
 
### Q3 / P3: Gas and price
 
**EN:** Between 2022 and 2025, in the Spanish peninsular system, what is the correlation between the daily average day-ahead electricity price and the daily MIBGAS gas price (€/MWh), and how does this relationship change before, during and after the Iberian gas price cap?
 
**ES:** Entre 2022 y 2025, en el sistema peninsular español, ¿qué correlación hay entre el precio medio diario del mercado diario de electricidad y el precio diario del gas en MIBGAS (€/MWh), y cómo cambia esa relación antes, durante y después del mecanismo ibérico de tope al gas?
 
### Q4 / P4 (version 2)
 
**EN:** Can tomorrow's price be forecast better than with a simple baseline?
 
**ES:** ¿Se puede predecir el precio de mañana mejor que con una referencia simple?
 
## Key decisions / Decisiones clave
 
| Decision | Why |
|---|---|
| Wholesale day-ahead price, not the retail bill | Renewables and gas compete in the wholesale market. The bill adds taxes, network charges and margins, and most consumers have fixed prices, so the hourly effect would be diluted. Consumers come back in the conclusions (PVPC). |
| Renewable share = solar + wind as % of hourly peninsular generation | The price is hourly, so the share must be hourly too. |
| Half-open bins `[a, b)` | The share is continuous: every value falls in exactly one bin. |
| Bins are provisional | To be confirmed after checking how many hours fall in each bin (Prepare phase). |
| Compare year by year | Controls for gas: in 2022 expensive gas and few high-renewable hours would exaggerate the renewable effect (confounding). |
| Months instead of seasons | Finer granularity; months can always be aggregated into seasons later. |
| Q3 at daily frequency | Gas is published daily, so electricity is aggregated to a daily average (downsampling). |
| Split Q3 into before / during / after the gas cap | The cap broke the link between the real gas price and the electricity price (structural break). |
 
## Hypotheses / Hipótesis
 
- The cheapest hours may have moved from night to midday because of solar growth (duck curve).
- The effect of gas on the price depends on renewables: low in high-solar hours (interaction).
## Known limitations / Limitaciones conocidas
 
- Holidays are not separated from regular weekdays in v1.
- Correlation is not causation: comparing within years and splitting periods reduces confounding but does not remove it. The v2 model includes renewables and gas together.
 