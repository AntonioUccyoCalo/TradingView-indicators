In questa cartella creeremo degli script che saranno utilizzati su trading view.
Il linguaggio da usare è pinescript, trovi la documentazione su https://www.tradingview.com/pine-script-docs

## Struttura progetto

### DCA/ — Dollar Cost Average (DCA - PAC)
Indicatore che simula un piano di accumulo (PAC/DCA) su un asset, con acquisti mensili a importo fisso.

**Funzionalità:**
- Investimento mensile configurabile con periodo di inizio/fine
- Calcolo P/L nominale e percentuale, rendimento annualizzato
- Max Drawdown del portafoglio
- Tassazione sulle plusvalenze (aliquota configurabile)
- **Aggiustamento per inflazione (CPI):** usa dati CPI reali da FRED tramite `request.security()` con supporto multi-paese (USA, EU, UK, Japan, Canada, Australia). Mostra una colonna "Real (CPI)" accanto ai valori nominali nella tabella riepilogativa.

**Dati CPI utilizzati:**
- USA: `FRED:CPIAUCSL`
- EU: `FRED:CP0000EZ19M086NEST`
- UK: `FRED:GBRCPIALLMINMEI`
- Japan: `FRED:JPNCPIALLMINMEI`
- Canada: `FRED:CANCPIALLMINMEI`
- Australia: `FRED:AUSCPIALLMINMEI`

