# AI-ML_challenge

For this telematics chatbot project, the implementation requires several layers of technology: database setup, a back-end API to interface with the database and NLP model, a dynamic query generator, and a front-end interface. Below is a full Python-based implementation plan using Flask for the backend, SQLAlchemy for database interaction, and Rasa for the NLP chatbot framework. This setup will also include a simple front-end using HTML and JavaScript to interact with the bot.

Let’s dive in.

---

### **Step 1: Setting up the Project**

- **Requirements**:
  - **Database**: MySQL
  - **Backend**: Flask
  - **NLP Engine**: Rasa (for intent recognition and entity extraction)
  - **ORM**: SQLAlchemy
  - **Front-end**: HTML/CSS with JavaScript for the chat interface

#### Install the necessary libraries:
```bash
pip install Flask SQLAlchemy mysql-connector-python rasa
```

### **Step 2: Database Setup and Telematics Data Loading**

1. **Create the Database and Tables**

Use SQLAlchemy to define the tables. For this example, assume the data tables for telematics are:
   - `ride_statistics`: Holds information about distances and rides.
   - `battery_statistics`: Stores battery health data.
   - `motor_data`: Contains information about motor speed and temperature.
   - `fault_codes`: Logs any fault codes.

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql+mysqlconnector://user:password@localhost/telematics'
db = SQLAlchemy(app)

class RideStatistics(db.Model):
    __tablename__ = 'ride_statistics'
    id = db.Column(db.Integer, primary_key=True)
    distance = db.Column(db.Float)
    time = db.Column(db.DateTime)

class BatteryStatistics(db.Model):
    __tablename__ = 'battery_statistics'
    id = db.Column(db.Integer, primary_key=True)
    charge_level = db.Column(db.Float)
    health_status = db.Column(db.String(50))

class MotorData(db.Model):
    __tablename__ = 'motor_data'
    id = db.Column(db.Integer, primary_key=True)
    speed = db.Column(db.Float)
    temperature = db.Column(db.Float)

class FaultCodes(db.Model):
    __tablename__ = 'fault_codes'
    id = db.Column(db.Integer, primary_key=True)
    error_message = db.Column(db.String(255))
    status = db.Column(db.String(50))
```

### **Step 3: NLP Model with Rasa**

To create the chatbot, set up **intents** in Rasa to recognize user queries related to battery health, distance traveled, fault codes, etc.

1. **Install Rasa**:
   ```bash
   rasa init
   ```

2. **Define Intents and Responses in Rasa**

Update `domain.yml` with intents and responses. Example:

```yaml
intents:
  - battery_status
  - ride_statistics
  - check_fault

responses:
  utter_battery_status:
    - text: "The current battery status is: {charge_level}% and health is {health_status}."

  utter_ride_statistics:
    - text: "The vehicle traveled {distance} km."

  utter_fault_codes:
    - text: "Current fault code: {error_message}, Status: {status}."
```

3. **Create Stories** in `data/stories.yml` to train Rasa on the conversation flow.

4. **Train the Model**:
   ```bash
   rasa train
   ```

### **Step 4: Backend - Flask API for Query Handling and Response**

1. **Define Flask API Endpoints** to interact with the Rasa bot and retrieve data from MySQL.

```python
from flask import request, jsonify
from rasa.core.agent import Agent
import asyncio

agent = Agent.load("models")  # Load trained Rasa model

@app.route('/chat', methods=['POST'])
def chat():
    user_message = request.json.get("message")
    response = asyncio.run(agent.handle_text(user_message))
    
    intent = response[0]['intent']['name']

    if intent == 'battery_status':
        battery = BatteryStatistics.query.order_by(BatteryStatistics.id.desc()).first()
        return jsonify({
            "response": f"The current battery status is {battery.charge_level}% and health is {battery.health_status}."
        })
    elif intent == 'ride_statistics':
        ride = RideStatistics.query.order_by(RideStatistics.id.desc()).first()
        return jsonify({
            "response": f"The vehicle traveled {ride.distance} km."
        })
    elif intent == 'check_fault':
        fault = FaultCodes.query.order_by(FaultCodes.id.desc()).first()
        return jsonify({
            "response": f"Current fault code: {fault.error_message}, Status: {fault.status}."
        })
    else:
        return jsonify({"response": "I'm not sure how to help with that."})
```

### **Step 5: Front-end - Web Interface with HTML & JavaScript**

1. **HTML** for chat interface (`templates/index.html`):

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Vehicle Telematics Chat-bot</title>
    <style>
        #chat-container { width: 300px; margin: auto; }
        #chat-box { height: 400px; overflow-y: scroll; border: 1px solid #ccc; padding: 10px; }
    </style>
</head>
<body>
    <div id="chat-container">
        <div id="chat-box"></div>
        <input type="text" id="userInput" placeholder="Ask about your vehicle..." />
        <button onclick="sendMessage()">Send</button>
    </div>

    <script>
        async function sendMessage() {
            let message = document.getElementById('userInput').value;
            document.getElementById('chat-box').innerHTML += `<p><b>User:</b> ${message}</p>`;
            
            let response = await fetch('/chat', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ message: message })
            });
            
            let data = await response.json();
            document.getElementById('chat-box').innerHTML += `<p><b>Bot:</b> ${data.response}</p>`;
            document.getElementById('userInput').value = '';
        }
    </script>
</body>
</html>
```

2. **Run Flask**:
   ```bash
   flask run
   ```

Now you have a functioning vehicle telematics chatbot system with NLP capabilities to handle queries and retrieve information from a MySQL database. This system can be further enhanced by adding more intents or optimizing the Rasa bot's training data. 

Let me know if you’d like more details on any part!
