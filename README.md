# AGRISAARTHI – prototype

Flask + SQLite + scikit-learn backend, HTML/CSS/JS + Chart.js front end.

## Run it (VS Code terminal)
```bash
python -m venv .venv
.venv\Scripts\activate          # Windows   (macOS/Linux: source .venv/bin/activate)
pip install -r requirements.txt
python collector.py --seed-demo # DEMO data so you can try the app (not real prices)
python app.py                   # open http://127.0.0.1:5000
python tests/test_app.py        # optional: 15 automated tests
```

## Use real Agmarknet data
Agmarknet's daily prices are published through data.gov.in. Get a free API key at https://data.gov.in
(My Account -> API key), then:
```bash
set DATA_GOV_IN_API_KEY=your_key        # Windows  (macOS/Linux: export ...)
python collector.py --live --purge-demo # fetch today's prices, remove demo rows
```
* The live API returns the **current day**. Run it daily (Task Scheduler / cron) to build history; the
  forecast needs at least 50 consecutive days per mandi and crop.
* To load history at once, export a CSV from Agmarknet / CEDA and run `python collector.py --csv file.csv`
  (columns: State, District, Market, Commodity, Arrival_Date, Min/Max/Modal Price).
* Agmarknet does not publish mandi coordinates. Markets are matched by name to `geo.PLACES`; unmatched
  ones are stored but left out of nearby search (the collector lists them). Add towns to `geo.PLACES` to include them.

## How "nearby" works
1. The typed place (or browser GPS) is resolved to latitude/longitude.
2. Distance to every mandi with data for the crop is computed (haversine); road distance is estimated at 1.3x.
3. Mandis inside the radius (default 200 km) are forecast with scikit-learn (Ridge model, 7 days,
   back-tested against a "price stays the same" baseline; the baseline is used if the model does not beat it).
4. Sellers: expected price minus transport. Buyers: expected price plus transport. Transport defaults to
   Rs 0.25 per quintal per km (change with env var TRANSPORT_RS_PER_QTL_KM).

Files: app.py (API), collector.py (data), geo.py (locations), model.py (ML), db.py (SQLite), static/ (front end).
