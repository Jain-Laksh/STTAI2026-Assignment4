# Assignment 4: PropTech Startup Strategy - Rent Prediction Pipeline

**Course**: Software Tools and Techniques for AI

**Total Marks**: 20

**Submission**: Submit your GitHub repository link containing all code, logs, and screenshots.

## Problem Statement: UrbanNest Analytics

You have just been hired as the Lead MLOps Engineer at **UrbanNest Analytics**, a fast-growing Property Technology (PropTech) startup. The company wants to launch a **Dynamic House Rent Prediction Engine** for four key cities: Mumbai, Pune, Delhi, and Hisar. 

Currently, the data science team has built a basic Random Forest model to predict rent (`price`), but as user demand grows, the startup faces two challenges:
1. **Cloud Computing Budgets:** Training large models exhaustively is too expensive. The founders want you to rigorously compare Hyperparameter Optimization techniques (Grid Search vs. Random Search vs. Bayesian Optimization) to find the most cost-effective method for achieving high accuracy without wasting compute time.
2. **Seamless Deployment:** The model needs to be accessible to end-users via a clean UI and safely packaged into Docker containers to avoid "it works on my machine" issues.

Your objective is to finalize the model optimization, track your experiments, build an API, create a user-friendly frontend, and deploy it all using Docker and Hugging Face Spaces.

## Dataset
You are provided with a dataset in the `Dataset` folder:
*   `train.csv`: Use this to train and perform Cross-Validation hyperparameter tuning.
*   `test.csv`: Use this to report your final model metrics.

**Important Note on Data:** The data contains categorical features like `city`, `Status`, and `property_type`. You must appropriately encode these into numerical formats before feeding them to the Random Forest model and ensure your Streamlit frontend reverses this mapping properly.

---

## Tasks & Mark Distribution 

### Task 1: Data Preprocessing, Optimization & Tracking (6 Marks)
**Goal:** Prove to the founders which optimization strategy gives the best "bang for buck".
*   **Preprocessing:** You must keep all features from the dataset for the final UI predictions. Create a preprocessing pipeline that converts string categorical variables (like `city`, `location`, `Status`, `property_type`) into numbers. You must strictly use `sklearn.preprocessing.LabelEncoder` or manual mapping and **save these mappings** so your frontend can use them later.
*   **Optimization Comparison:** Using strictly `train.csv` and 5-fold Cross-Validation (`sklearn.model_selection.cross_val_score`), tune the hyperparameters of a **Random Forest Regressor** (`sklearn.ensemble.RandomForestRegressor`). You must strictly implement and compare:
    1. **Grid Search** (using `sklearn.model_selection.GridSearchCV`)
    2. **Random Search** (using `sklearn.model_selection.RandomizedSearchCV`)
    3. **Bayesian Optimization** (using the `optuna` library, specifically creating a study with `optuna.create_study` and calling `.optimize`)
*   **Search Space Definition:** You **must** restrict your hyperparameter search to the following bounds for all three methods. This ensures a fair comparison:
    *   `n_estimators`: Integer values from `50` to `200`.
    *   `max_depth`: Integer values from `10` to `30`.
    *   `min_samples_split`: Integer values from `2` to `10`.
*   **Evaluation & Plots:** You must generate and **save** (in the `plots/` folder) the following plots:
    1.  `budget_vs_error.png`: A line plot comparing the compute budget vs. the error. 
        *   **X-axis:** Number of iterations/trials.
        *   **Y-axis:** Best mean cross-validation error found up to that iteration. 
        *   *Plot all three strategies (Grid, Random, Bayesian) overlaid on the same graph.*
    2.  `optuna_hyperparameter_space.png`: A plot of the hyperparameter space (using `optuna.visualization.plot_optimization_history` or `optuna.visualization.plot_contour`) to demonstrate how the Bayesian method explored the parameters.
*   **Final Testing & Reporting:** 
    *   Clearly **print/report** the best hyperparameters found by each of the three methods in your notebook.
    *   Retrain your overall *best* model (the one with the lowest CV error) on the entire `train.csv`. 
    *   Predict on the untouched `test.csv` and report the final Mean Absolute Error (MAE). 

**Expected Outputs for Task 1:**
*   `train.ipynb` notebook containing your preprocessing, Cross-Validation tuning, final evaluation on `test.csv`, clearly reported best hyperparameters, and plot generating code.
*   The `models/` folder containing the final saved model file (e.g., `best_rf_model.pkl`) and necessary preprocessing encoders.
*   The `plots/` folder containing the two requested PNG plot images.

---

### Task 2: API & Frontend Development (4 Marks)
**Goal:** Build a user-facing dashboard for real estate agents.
*   **Backend (FastAPI):** Expose your best tuned model via a FastAPI app. 
    *   Initialize `app = FastAPI()`.
    *   Create a POST endpoint (e.g., `@app.post("/predict")`) that strictly accepts a Pydantic `BaseModel` corresponding to all your dataset features.
    *   Load your saved model and Label Encoders using `pickle.load`.
*   **Frontend (Streamlit):** Build a Streamlit dashboard. 
    *   Use `st.selectbox`, `st.number_input`, or `st.slider` for ALL input features corresponding to the dataset.
    *   Upon button click (`st.button("Predict")`), use the `requests` library (`requests.post`) to query your FastAPI backend.
    *   Display the result using `st.success` or `st.write`.

**Expected Outputs for Task 2:**
*   `api.py` (FastAPI backend).
*   `app.py` (Streamlit frontend).
*   `requirements.txt` containing all required libraries.

---

### Task 3: Docker Containerization & Networking (5 Marks)
**Goal:** Package the microservices so they can be easily scaled.
*   **Backend Dockerfile:** Write a `Dockerfile` in the `backend/` folder spanning a Python base image setup for the FastAPI app. Ensure it installs requirements and uses `uvicorn` bounded to `0.0.0.0`.
*   **Frontend Dockerfile:** Write a separate `Dockerfile` in the `frontend/` folder for the Streamlit dashboard application. Make sure the container exposes the correct Streamlit port (typically `8501`).
*   **Docker Compose:** Write a `docker-compose.yml` file in the root directory that networks these two containers together. The Streamlit container must be able to communicate with the FastAPI container internally using Docker's service name networking (e.g., `http://backend:8000`), but both services must also be accessible locally on your host machine via explicit **port forwarding**.

**Expected Outputs for Task 3:**
*   `backend/Dockerfile` and `frontend/Dockerfile`.
*   `docker-compose.yml`.
*   A folder named `screenshots/` explicitly containing the following **three screenshots**:
    1.  `docker_compose_up.png`: A screenshot showing the successful output of running `docker-compose up --build` sequentially in your terminal and starting the containers.
    2.  `docker_ps.png`: A screenshot of the `docker ps` terminal command showing both containers actively running with the correct port mappings.
    3.  `streamlit_working.png`: A screenshot of your web browser accessing the Streamlit app at `localhost:8501` to successfully get a prediction.

---

### Task 4: Cloud Deployment via Hugging Face Spaces (5 Marks)
**Goal:** Show stakeholders a live, working distributed prototype.
*   Because you have separated your application into two microservices, you will deploy them as two separate Docker Spaces on Hugging Face:
    *   **Backend Deployment (FastAPI):** Deploy your FastAPI backend using a **Docker Space template** on Hugging Face. The Space will build your backend container and give you a public API URL.
    *   **Frontend Deployment (Streamlit):** Create a *second* Hugging Face **Docker Space** for your Streamlit application (using your `frontend/Dockerfile`). *Crucially*, you must configure your Streamlit app so that its `requests.post()` calls point to the public URL of your new Hugging Face backend API Space, rather than `localhost:8000`.
*   The final Streamlit Space application must be publicly accessible, successfully communicate with the Backend Space, and return predictions.

**Expected Outputs for Task 4:**
*   The **Streamlit Frontend Space URL** prominently placed at the very top of your GitHub repository's README.
*   The **FastAPI Backend Space URL** placed right below it.

---

## Submission Guidelines
Create a GitHub repository and push all your files. Your repository MUST strictly look like this:

```text
Assignment_4/
├── README.md                 # Project description and HF Space URL
├── requirements.txt          # Python dependencies
├── train.ipynb               # Task 1: Tuning and Model Evaluation Notebook
├── api.py                    # Task 2: FastAPI application code
├── app.py                    # Task 2: Streamlit application code
├── docker-compose.yml        # Task 3: Docker Compose configuration
├── backend/
│   └── Dockerfile            # Task 3: FastAPI Dockerfile
├── frontend/
│   └── Dockerfile            # Task 3: Streamlit Dockerfile
├── models/
│   └── best_rf_model.pkl     # Task 1: Saved model and encoders
├── plots/
│   ├── budget_vs_error.png          # Task 1 plot
│   └── optuna_hyperparameter_space.png  # Task 1 plot
├── Dataset/                  # Provided CSVs
└── screenshots/
    ├── docker_compose_up.png # Task 3 screenshot
    ├── docker_ps.png         # Task 3 screenshot
    └── streamlit_working.png # Task 3 screenshot
```

Submit the link to your GitHub repository when the google form is shared. Ensure the repository is Public so the TAs can review your code. Good luck, Lead Engineer!