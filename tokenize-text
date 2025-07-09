import json
import tiktoken
import functions_framework
import vertexai
from vertexai.generative_models import GenerativeModel

# --- Security Configuration ---
# Hardcoded API keys for different clients.
# In a real-world application, these should be stored securely (e.g., in a secret manager).
API_KEYS = {
    "client_key_abc123": "Client_A",
    "client_key_def456": "Client_B"
}

# --- Vertex AI Initialization ---
PROJECT_ID = "northwestern-sandbox"  # Replace with your project ID
vertexai.init(project=PROJECT_ID, location="us-central1") # Or your preferred region

@functions_framework.http
def tokenize_text(request):
    """HTTP Cloud Function for token counting with model routing and API key auth."""
    try:
        # --- API Key Authentication ---
        api_key = request.headers.get("X-API-Key")
        if not api_key:
            return (json.dumps({"error": "API key is missing from headers"}), 401, {"Content-Type": "application/json"})

        client_name = API_KEYS.get(api_key)
        if not client_name:
            return (json.dumps({"error": "Invalid API key"}), 403, {"Content-Type": "application/json"})

        # --- Request Body Parsing ---
        request_json = request.get_json(silent=True)
        if not request_json or "text" not in request_json:
            return (json.dumps({"error": "Missing 'text' in request body"}), 400, {"Content-Type": "application/json"})

        text = request_json["text"]
        model_name = request_json.get("model")

        # --- Token Counting ---
        token_count = count_tokens(text, model_name)
        model_used = model_name or "default (tiktoken)"

        # --- Structured Logging ---
        # Log key information about the successful request.
        # This will appear as a structured log in Google Cloud Logging.
        log_entry = {
            "message": f"Successfully processed token count request for {client_name}",
            "client": client_name,
            "model_used": model_used,
            "input_text_length": len(text),
            "token_count": token_count
        }
        print(json.dumps(log_entry))

        # --- Response ---
        response_data = {
            "token_count": token_count,
            "model_used": model_used,
            "client": client_name
        }
        return (json.dumps(response_data), 200, {"Content-Type": "application/json"})

    except Exception as e:
        # Log the error for debugging purposes
        error_log_entry = {"error_message": str(e)}
        print(json.dumps(error_log_entry))
        return (json.dumps({"error": str(e)}), 500, {"Content-Type": "application/json"})


def count_tokens(text, model_name=None):
    """
    Routes token counting to the appropriate library.
    Defaults to tiktoken if model_name is not provided.
    """
    if model_name:
        if model_name.startswith("gemini"):
            model = GenerativeModel(model_name)
            response = model.count_tokens(text)
            return response.total_tokens

        elif model_name.startswith("gpt-") or "turbo" in model_name:
            try:
                encoding = tiktoken.encoding_for_model(model_name)
            except KeyError:
                print(f"Warning: model '{model_name}' not found. Using cl100k_base encoding.")
                encoding = tiktoken.get_encoding("cl100k_base")
            return len(encoding.encode(text))
        
        else:
            raise ValueError(f"Unsupported model: {model_name}")
    else:
        # This print statement is for debugging and will also appear in logs.
        print("No model name provided. Defaulting to tiktoken with cl100k_base.")
        encoding = tiktoken.get_encoding("cl100k_base")
        return len(encoding.encode(text))
