# Alert Processing Application

This Flask-based application processes incoming alerts, generates responses using a LLaMA language model, and sends the results to a Telegram chat. It's designed to help automate the handling of system alerts and provide quick, AI-generated solutions.

## Features

- Receives alerts via HTTP POST requests
- Processes alert data to extract summaries and descriptions
- Generates responses using a LLaMA language model
- Sends formatted responses to a specified Telegram chat
- Configurable using environment variables
- Dockerized for easy deployment

## Prerequisites

- Python 3.12+
- Flask
- Ollama client
- Requests library
- A running LLaMA server
- A Telegram bot token and chat ID
- Docker (for containerized deployment)

## Installation

### Local Installation

1. Clone the repository:
   ```
   git clone https://github.com/yourusername/alert-processor.git
   cd alert-processor
   ```

2. Install the required dependencies:
   ```
   pip install -r requirements.txt
   ```

### Docker Installation

1. Clone the repository:
   ```
   git clone https://github.com/yourusername/alert-processor.git
   cd alert-processor
   ```

2. Build the Docker image:
   ```
   docker build -t alert-processor .
   ```

## Configuration

Set the following environment variables:

- `LLAMA_SERVER_URL`: URL of your LLaMA server (default: "http://10.10.10.53:11434")
- `TELEGRAM_BOT_TOKEN`: Your Telegram bot token
- `TELEGRAM_CHAT_ID`: The Telegram chat ID where messages will be sent

You can set these in a `.env` file, export them in your shell, or pass them to the Docker container.

## Usage

### Local Usage

1. Start the application:
   ```
   python main.py
   ```

2. The server will start running on `http://0.0.0.0:5123`

### Docker Usage

1. Run the Docker container:
   ```
   docker run -p 5123:5123 -e LLAMA_SERVER_URL=<your_llama_server_url> -e TELEGRAM_BOT_TOKEN=<your_token> -e TELEGRAM_CHAT_ID=<your_chat_id> alert-processor
   ```

2. The server will be accessible at `http://localhost:5123`

### Sending Alerts

Send POST requests to `http://your-server-address:5123/alerts` with JSON payloads containing alert data.

Example payload:
```json
{
  "alerts": [
    {
      "annotations": {
        "summary": "High CPU Usage",
        "description": "Server XYZ is experiencing CPU usage above 90% for the last 15 minutes."
      }
    }
  ]
}
```

The application will process the alert, generate a response using the LLaMA model, and send it to the configured Telegram chat.

## API Endpoints

- `/alerts` (POST): Receives alert data and processes it

## Error Handling

- The application logs errors and exceptions
- If the LLaMA server is unreachable, an error message will be sent to Telegram
- If Telegram credentials are missing, errors will be logged

## Docker Details

The Dockerfile uses Python 3.12-slim as the base image and sets up the environment for running the Flask application. It installs necessary dependencies, including git for fetching any Git-based requirements.

Key points:
- The application runs on port 5123 inside the container
- The working directory in the container is set to `/app`
- All project files are copied into the container

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

[Specify your license here]

## Support

For support, please open an issue in the GitHub repository or contact [your contact information].