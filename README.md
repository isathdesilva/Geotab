# EV Battery Health

COS4000 capstone with Geotab. EV battery State of Health prediction with a MyGeotab add-in dashboard.

The project has two parts that run together.

`backend` is the prediction service, written in Python with Flask. It loads the two trained LSTM models (State of Health regression and driving style classification) and the cleaned dataset, and answers the dashboard's requests with JSON. Details are in `backend/README.md`.

`addin` is the dashboard, a MyGeotab add-in built with React, TypeScript and Geotab's Zenith components. It was created with Geotab's generator-addin and shows the backend's results. It does no modelling itself. Details are in `addin/README.md`.

## Running it

You need Python 3.11 or newer and Node.js. Use two terminals, both opened in this folder.

Terminal 1, the backend:

```
cd backend
pip install -r requirements.txt
python app.py
```

Wait until it prints `Dashboard: http://127.0.0.1:5000`. The first start takes a few minutes while the models score the fleet.

Terminal 2, the add-in:

```
cd addin
npm install
npm run preview
```

The dashboard opens at http://localhost:3001. `pip install` and `npm install` are only needed the first time.

## Notes

The add-in finds the backend by its address, http://127.0.0.1:5000, so the backend must be running first.

`backend/templates` and `backend/static` hold the earlier plain HTML version of the dashboard. Flask still serves it at http://127.0.0.1:5000, but the add-in does not use it.

Putting the add-in into MyGeotab needs HTTPS hosting for both parts. The steps are at the end of `addin/README.md`.
