# LeafLens AI — Vercel edition

This version replaces the original Streamlit interface with a responsive static website that Vercel can serve. Vercel runs a small Python API proxy; TensorFlow inference runs in the separate FastAPI service in `backend/`.

**Why split it?** Vercel Python Functions have a 500 MB uncompressed bundle limit, and TensorFlow plus model files can make a serverless deployment too large. The proxy keeps the Vercel deployment light. See [Vercel Python runtime documentation](https://vercel.com/docs/functions/runtimes/python) and [function limits](https://vercel.com/docs/functions/limitations).

## Deploy the Vercel frontend

1. Upload the contents of this `LeafLensAI_Vercel` folder to a GitHub repository (the files inside this folder should be at the repository root, not inside another nested folder).
2. In Vercel, select **Add New → Project** and import that repository.
3. Use the default settings; no build command is needed for this static HTML/CSS/JS site.
4. Deploy. Visit `/api` on the resulting domain; it should return JSON with `status: ok`.

## Enable real AI predictions

The Vercel site is deployed separately from the TensorFlow backend. The page can upload and preview an image without a model, but real prediction requires a trained model and backend.

1. Deploy the `backend/` folder as a Python service on a host that supports FastAPI and TensorFlow (for example, a Python web service on Render). Use Python 3.11 and the start command `uvicorn main:app --host 0.0.0.0 --port $PORT`.
2. Put the trained `models/leaf_lens_model.keras` and `models/class_names.txt` files in the backend's `models/` folder. Generate them by placing labeled images under `backend/dataset/<class_name>/` and running `python train.py` in `backend/`.
3. Test the backend's `/health` endpoint. It should report `model_ready: true`.
4. In Vercel → Project → Settings → Environment Variables, add `LEAFLENS_API_URL` with the full backend prediction URL, e.g. `https://YOUR-BACKEND-DOMAIN/predict`. Do not include a trailing slash after `/predict`.
5. Redeploy the Vercel project so the environment variable takes effect.
6. Open the Vercel site, upload a real leaf photo, and select **Analyze leaf**.

Until `LEAFLENS_API_URL` is configured, the app deliberately shows a clear message instead of pretending to predict. Until the model files exist, the backend returns HTTP 503.

## Important notes

- Use a real leaf image, not a logo or unrelated image.
- Predictions depend on the classes and quality of the labeled training data; this is educational screening, not a confirmed agricultural diagnosis.
- The Vercel proxy accepts uploads up to 4 MB. Vercel has a 4.5 MB function request/response payload limit.
- `app.py` is the original Streamlit app and is not used by this Vercel frontend. It is not included at the project root to avoid the earlier `PYTHON_ENTRYPOINT_NOT_FOUND` error.

## Folder structure

```text
LeafLensAI_Vercel/
├── index.html
├── styles.css
├── app.js
├── vercel.json
├── api/
│   ├── index.py
│   └── predict.py
└── backend/
    ├── main.py
    ├── train.py
    ├── evaluate.py
    ├── requirements.txt
    ├── Procfile
    ├── runtime.txt
    ├── README.md
    ├── dataset/       # add labeled training images
    └── models/        # add generated model files
```
