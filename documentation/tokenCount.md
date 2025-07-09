# API Documentation: Token Counter

## Overview

This document provides instructions for using the Token Counter API. This API calculates the number of tokens in a given text string based on the specified generative model (e.g., Google's Gemini, OpenAI's GPT series). If no model is specified, it defaults to a standard `tiktoken` tokenizer.

This service is designed as an HTTP-triggered cloud function.

---

## Endpoint

**URL**: `[YOUR_CLOUD_FUNCTION_URL]`  
**Method**: `POST`

---

## Request Body

The request body must be a JSON object containing the text to be tokenized.

### Parameters

| Parameter | Type   | Required | Description                                                                                                                                                           |
| :-------- | :----- | :------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `text`    | String | **Yes**  | The text string for which you want to count the tokens.                                                                                                               |
| `model`   | String | No       | The name of the model to use for tokenization. If omitted, the API defaults to `tiktoken` with the `cl100k_base` encoding. Supported families: `gemini-`, `gpt-`, `turbo`. |

### Example Request Body

```json
{
  "text": "Hello, world! This is a test.",
  "model": "gemini-1.5-flash"
}
```

---

## Responses

### Success Response

On a successful request, the API returns a `200 OK` status code with a JSON object containing the token count.

**Status Code**: `200 OK`

**Body**:

| Parameter    | Type   | Description                                                                 |
| :----------- | :----- | :-------------------------------------------------------------------------- |
| `token_count`| Integer| The calculated number of tokens for the input text.                         |
| `model_used` | String | The model that was used for tokenization (e.g., `gemini-1.5-flash`, `gpt-4`). |

**Example Success Body**:

```json
{
  "token_count": 8,
  "model_used": "gemini-1.5-flash"
}
```

### Error Responses

If the request is invalid or an internal error occurs, the API will return an appropriate error code and a JSON object with an error message.

| Status Code | Error Condition                     | Example Error Body                                         |
| :---------- | :---------------------------------- | :--------------------------------------------------------- |
| `400`       | The `text` field is missing.        | `{"error": "Missing 'text' in request body"}`              |
| `500`       | An unsupported `model` is provided. | `{"error": "Unsupported model: llama-3"}`                  |
| `500`       | An internal server error occurs.    | `{"error": "[Detailed error message from the server]"}`    |

---

## Example Usage (`curl`)

Below are `curl` examples demonstrating how to call the API. Replace `[YOUR_CLOUD_FUNCTION_URL]` with the actual URL of your deployed function.

### 1. Default Tokenization (No Model Specified)

This will use the default `tiktoken` library for token counting.

```bash
curl -X POST "[YOUR_CLOUD_FUNCTION_URL]" \
-H "Content-Type: application/json" \
-d '{
  "text": "This is a test of the default tokenizer."
}'
```

### 2. Tokenization with a Gemini Model

This example specifies a Gemini model.

```bash
curl -X POST "[YOUR_CLOUD_FUNCTION_URL]" \
-H "Content-Type: application/json" \
-d '{
  "text": "How does tokenization work for Gemini models?",
  "model": "gemini-1.0-pro"
}'
```

### 3. Tokenization with an OpenAI Model

This example specifies a GPT model, which will be handled by `tiktoken`.

```bash
curl -X POST "[YOUR_CLOUD_FUNCTION_URL]" \
-H "Content-Type: application/json" \
-d '{
  "text": "How does tokenization work for OpenAI models?",
  "model": "gpt-4-turbo"
}'
```

### 4. Invalid Request (Missing `text`)

This example shows an invalid request that is missing the required `text` field.

```bash
curl -X POST "[YOUR_CLOUD_FUNCTION_URL]" \
-H "Content-Type: application/json" \
-d '{
  "model": "gpt-4"
}'
```
