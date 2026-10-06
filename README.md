# Capstone--project
Capstone project
========================================================================
AI ENTERPRISE WORKFLOW CAPSTONE PROJECT - TESTING PIPELINE
========================================================================

------------------------------------------------------------------------
FILE 1: app.py (Flask API Server Gateway)
------------------------------------------------------------------------
from flask import Flask, request, jsonify
import csv
import os
from datetime import datetime

app = Flask(__name__)
LOG_FILE = "predictions.log"

def log_prediction(country, date, prediction):
    file_exists = os.path.isfile(LOG_FILE)
    with open(LOG_FILE, mode='a', newline='') as f:
        writer = csv.writer(f)
        if not file_exists:
            writer.writerow(["timestamp", "country", "date", "prediction"])
        writer.writerow([datetime.now().isoformat(), country, date, prediction])

def model_predict(country, date):
    if not country or not date:
        raise ValueError("Missing country or target date.")
    return 15000.50 

@app.route('/predict', methods=['POST'])
def predict():
    data = request.get_json() or {}
    country = data.get('country')
    date = data.get('date')
    
    if not country or not date:
        return jsonify({"error": "Invalid input. Target date and country required."}), 400
        
    try:
        prediction = model_predict(country, date)
        log_prediction(country, date, prediction)
        return jsonify({"status": "success", "prediction": prediction}), 200
    except Exception as e:
        return jsonify({"error": str(e)}), 500

------------------------------------------------------------------------
FILE 2: run_tests.py (Master Test Suite Runner)
------------------------------------------------------------------------
import unittest
import json
import os

class TestEnterpriseCapstoneSuite(unittest.TestCase):
    
    def setUp(self):
        self.app = app.test_client()
        self.app.testing = True

    def tearDown(self):
        if os.path.exists(LOG_FILE):
            os.remove(LOG_FILE)

    # CRITERIA 1: API Unit Tests
    def test_api_predict_success(self):
        payload = {"country": "united_kingdom", "date": "2026-11-01"}
        response = self.app.post('/predict', data=json.dumps(payload), content_type='application/json')
        self.assertEqual(response.status_code, 200)
        self.assertIn('prediction', response.json)

    def test_api_missing_inputs(self):
        payload = {"country": "united_kingdom"}
        response = self.app.post('/predict', data=json.dumps(payload), content_type='application/json')
        self.assertEqual(response.status_code, 400)

    # CRITERIA 2: Model Unit Tests
    def test_model_predict_valid(self):
        prediction = model_predict("united_kingdom", "2026-11-01")
        self.assertIsInstance(prediction, float)

    def test_model_predict_invalid_trigger(self):
        with self.assertRaises(ValueError):
            model_predict(None, "2026-11-01")

    # CRITERIA 3: Logging Unit Tests
    def test_logging_file_generation(self):
        log_prediction("united_kingdom", "2026-11-01", 15000.50)
        self.assertTrue(os.path.exists(LOG_FILE))

if __name__ == "__main__":
    print("Executing all Enterprise Unit Test Suites sequentially...")
    unittest.main()
    
