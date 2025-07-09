# API Documentation: Token Counter

## Overview

This document provides instructions for using the Token Counter API. This API calculates the number of tokens in a given text string based on the specified generative model (e.g., Google's Gemini, OpenAI's GPT series). If no model is specified, it defaults to a standard `tiktoken` tokenizer.

This service is designed as an HTTP-triggered cloud function and requires an API key for access.

---

## Endpoint

**URL**: `https://us-central1-northwestern-sandbox.cloudfunctions.net/tokenize-text`  
**Method**: `POST`

---

## Authentication

This API is protected and requires a valid API key to be passed in the `X-API-Key` header of each request. Please use the API key that was assigned to you.

| Header      | Description              |
| :---------- | :----------------------- |
| `X-API-Key` | Your assigned API key.   |

---

## Request Body

The request body must be a JSON object.

### Parameters

| Parameter | Type   | Required | Description                                                                                                                                                           |
| :-------- | :----- | :------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `text`    | String | **Yes**  | The text string for which you want to count the tokens.                                                                                                               |
| `model`   | String | No       | The name of the model to use for tokenization. If omitted, the API defaults to `tiktoken` with the `cl100k_base` encoding. Supported families: `gemini-`, `gpt-`, `turbo`. |

---

## Responses

### Success Response

On a successful request, the API returns a `200 OK` status code with a JSON object.

**Status Code**: `200 OK`

**Body**:

| Parameter    | Type   | Description                                                                 |
| :----------- | :----- | :-------------------------------------------------------------------------- |
| `token_count`| Integer| The calculated number of tokens for the input text.                         |
| `model_used` | String | The model that was used for tokenization.                                   |
| `client`     | String | The name of the client associated with the provided API key.                |

### Error Responses

| Status Code | Error Condition                     | Example Error Body                                         |
| :---------- | :---------------------------------- | :--------------------------------------------------------- |
| `400`       | The `text` field is missing.        | `{"error": "Missing 'text' in request body"}`              |
| `401`       | The `X-API-Key` header is missing.  | `{"error": "API key is missing from headers"}`             |
| `403`       | The provided API key is invalid.    | `{"error": "Invalid API key"}`                             |
| `500`       | An unsupported `model` is provided. | `{"error": "Unsupported model: llama-3"}`                  |
| `500`       | An internal server error occurs.    | `{"error": "[Detailed error message from the server]"}`    |

---

## Example Usage (`curl`)

Replace `[YOUR_CLOUD_FUNCTION_URL]` with your function's URL and `[YOUR_ASSIGNED_API_KEY]` with your assigned API key.

### 1. Default Tokenization

```bash
curl -X POST "[YOUR_CLOUD_FUNCTION_URL]" \
-H "Content-Type: application/json" \
-H "X-API-Key: [YOUR_ASSIGNED_API_KEY]" \
-d '{
  "text": "This is a test of the default tokenizer."
}'
```

### 2. Tokenization with a Gemini Model

```bash
curl -X POST "[YOUR_CLOUD_FUNCTION_URL]" \
-H "Content-Type: application/json" \
-H "X-API-Key: [YOUR_ASSIGNED_API_KEY]" \
-d '{
  "text": "How does tokenization work for Gemini models?",
  "model": "gemini-1.0-pro"
}'
```

### 3. Invalid Request (Missing API Key)

```bash
curl -X POST "[YOUR_CLOUD_FUNCTION_URL]" \
-H "Content-Type: application/json" \
-d '{
  "text": "This request will fail.",
  "model": "gpt-4"
}'
```
