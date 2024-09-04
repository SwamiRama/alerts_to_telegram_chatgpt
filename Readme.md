from flask import Flask, request, jsonify
import requests
import os
import logging
from ollama import Client
from typing import Any, Dict, Tuple

app = Flask(__name__)

# LLaMA-Server URL
LLAMA_SERVER_URL = os.getenv("LLAMA_SERVER_URL", "http://10.10.10.53:11434")

# Telegram Bot Token und Chat ID
TELEGRAM_BOT_TOKEN = os.getenv("TELEGRAM_BOT_TOKEN")
TELEGRAM_CHAT_ID = os.getenv("TELEGRAM_CHAT_ID")

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

@app.route("/alerts", methods=["POST"])
def receive_alert() -> Tuple[Dict[str, str], int]:
    alert_data = request.json
    if alert_data:
        summary, description = extract_alert_details(alert_data)
        llama_response = get_llama_response(summary, description)
        send_to_telegram(llama_response)
        return jsonify({"status": "success", "message": "Alert processed"}), 200
    else:
        return jsonify({"status": "error", "message": "No alert data received"}), 400

def extract_alert_details(alert_data: Dict[str, Any]) -> Tuple[str, str]:
    """Extracts relevant details from the alert data."""
    alert = alert_data.get("alerts", [{}])[0]
    annotations = alert.get("annotations", {})
    summary = annotations.get("summary", "No summary available")
    description = annotations.get("description", "No description available")
    return summary, description

def get_llama_response(summary: str, description: str) -> str:
    prompt = f"Incoming Alert:\n\nSummary: {summary}\nDescription: {description}\n\nHow can I fix the error? Please answer so that the formatting in Telegram looks good."

    client = Client(host=LLAMA_SERVER_URL)
    
    try:
        response = client.chat(
            model="llama3",
            messages=[
                {
                    "role": "user",
                    "content": prompt,
                },
            ],
        )
        message = response.get('message', {}).get('content', None)
        if message:
            return message
        else:
            return "No response from LLaMA server."
    except Exception as e:
        logger.error(f"Error occurred: {str(e)}")
        return f"Error occurred: {str(e)}"

def send_to_telegram(message: str) -> None:
    if not TELEGRAM_BOT_TOKEN or not TELEGRAM_CHAT_ID:
        logger.error("Telegram bot token or chat ID not set.")
        return

    telegram_api_url = f"https://api.telegram.org/bot{TELEGRAM_BOT_TOKEN}/sendMessage"
    payload = {"chat_id": TELEGRAM_CHAT_ID, "text": message}
    try:
        response = requests.post(telegram_api_url, data=payload)
        response.raise_for_status()  # Raise an exception for HTTP errors
    except requests.exceptions.RequestException as e:
        logger.error(f"Failed to send message to Telegram: {e}")

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5123)