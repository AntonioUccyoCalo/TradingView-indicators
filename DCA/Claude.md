# Dollar Cost Average (DCA - PAC)

Indicatore PineScript v6 per TradingView che simula un Piano di Accumulo (PAC).
Ogni mese viene investita una somma fissa su un asset finanziario; l'indicatore mostra graficamente la crescita dell'importo investito e del controvalore nel tempo.

## Input

| Input | Tipo | Default | Gruppo |
|---|---|---|---|
| Importo Mensile | `input.int` | 100 | Impostazioni DCA |
| Data Inizio | `input.time` | 2024-12-01 | Periodo |
| Data Fine | `input.time` | 2025-12-31 | Periodo |
| Tasse Plusvalenza % | `input.float` | 26.0% | Tassazione |

## Logica

- Rileva l'inizio di ogni nuovo mese con `ta.change(time("M"))`
- Se la barra rientra nel periodo selezionato, accumula l'importo mensile e calcola le quote acquistate (`importo / close`)
- Calcola il controvalore del portafoglio (`totalShares * close`)
- Calcola P/L assoluto e percentuale
- Calcola il Prezzo Medio di Carico (PMC): `totalInvested / totalShares`
- Calcola il rendimento annualizzato (CAGR)
- Calcola il Max Drawdown: massima perdita percentuale dal picco del controvalore
- Calcola le tasse sulla plusvalenza (solo se P/L > 0)
- Calcola il netto dopo tasse: controvalore meno le tasse (se in guadagno)

## Output grafico

- **Linea arancione** ("Totale Investito"): cresce a gradini ad ogni investimento mensile
- **Linea verde/rossa** ("Controvalore"): segue l'andamento del mercato. Verde quando sopra la linea investita, rossa quando sotto
- **Fill tra le linee**: area colorata verde (in guadagno) o rossa (in perdita) con trasparenza 85%
- **Label "buy"**: diamante blu su ogni punto di acquisto mensile

## Tabella riepilogativa (top-right, 12 righe)

| Riga | Etichetta | Colore |
|---|---|---|
| 0 | P/L % | Verde/Rosso |
| 1 | P/L | Verde/Rosso |
| 2 | P/L % Ann. | Verde/Rosso |
| 3 | N. Investimenti | Bianco |
| 4 | Investito | Bianco |
| 5 | Controvalore | Bianco |
| 6 | PMC | Bianco |
| 7 | Max Drawdown | Arancione |
| 8 | Netto dopo tasse | Acqua |
| 9 | Aliquota Tasse | Rosso |
| 10 | Tasse su P/L | Rosso |

## Note

- L'indicatore usa `overlay=false` (pannello separato, perché i valori monetari hanno scala diversa dal prezzo)
- La tabella mostra i valori dell'ultima barra e non segue il cursore
- I valori delle due linee (Investito e Controvalore) sono visibili nella Data Window di TradingView muovendo il mouse sul grafico
