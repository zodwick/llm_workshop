# Building a Groq Chat Application with Streamlit

## Overview
This guide explains how to build a chat application using Groq's API and Streamlit. The application supports multiple language models, features real-time streaming responses, and provides a user-friendly interface.

## Prerequisites
```bash
pip install streamlit groq python-dotenv
```

## Implementation

### 1. Basic Setup and Imports
```python
import os
import streamlit as st
from typing import Generator
from groq import Groq
from dotenv import load_dotenv

load_dotenv()
```

### 2. Configure Groq Client
```python
client = Groq(
    api_key=os.environ.get("GROQ_API_KEY"),
)
```

### 3. UI Components

#### Header and Logo
```python
st.write(
    f'<span style="font-size: 78px; line-height: 1">{"🏎️"}</span>',
    unsafe_allow_html=True,
)
st.subheader("Groq Chat Streamlit App", divider="rainbow", anchor=False)
```

#### Session State Initialization
```python
# Initialize chat history and selected model
if "messages" not in st.session_state:
    st.session_state.messages = []

if "selected_model" not in st.session_state:
    st.session_state.selected_model = None
```

#### Model Configuration
```python
models = {
    "gemma-7b-it": {
        "name": "Gemma-7b-it", 
        "tokens": 8192, 
        "developer": "Google"
    },
    "llama2-70b-4096": {
        "name": "LLaMA2-70b-chat", 
        "tokens": 4096, 
        "developer": "Meta"
    },
    "mixtral-8x7b-32768": {
        "name": "Mixtral-8x7b-Instruct-v0.1", 
        "tokens": 32768, 
        "developer": "Mistral"
    },
    # Add more models as needed
}
```

### 4. Layout Implementation

#### Model Selection and Token Configuration
```python
col1, col2 = st.columns(2)

with col1:
    model_option = st.selectbox(
        "Choose a model:",
        options=list(models.keys()),
        format_func=lambda x: models[x]["name"],
        index=4  # Default to mixtral
    )

# Reset chat history on model change
if st.session_state.selected_model != model_option:
    st.session_state.messages = []
    st.session_state.selected_model = model_option

max_tokens_range = models[model_option]["tokens"]

with col2:
    max_tokens = st.slider(
        "Max Tokens:",
        min_value=512,
        max_value=max_tokens_range,
        value=min(32768, max_tokens_range),
        step=512,
        help=f"Adjust the maximum number of tokens (words) for the model's response. Max for selected model: {max_tokens_range}"
    )
```

### 5. Chat Interface Implementation

#### Display Chat History
```python
for message in st.session_state.messages:
    avatar = '🤖' if message["role"] == "assistant" else '👨‍💻'
    with st.chat_message(message["role"], avatar=avatar):
        st.markdown(message["content"])
```

#### Response Generator
```python
def generate_chat_responses(chat_completion) -> Generator[str, None, None]:
    """Yield chat response content from the Groq API response."""
    for chunk in chat_completion:
        if chunk.choices[0].delta.content:
            yield chunk.choices[0].delta.content
```

#### Chat Input and Response Handling
```python
if prompt := st.chat_input("Enter your prompt here..."):
    # Add user message
    st.session_state.messages.append({"role": "user", "content": prompt})
    with st.chat_message("user", avatar='👨‍💻'):
        st.markdown(prompt)

    # Get response from Groq
    try:
        chat_completion = client.chat.completions.create(
            model=model_option,
            messages=[
                {
                    "role": m["role"],
                    "content": m["content"]
                }
                for m in st.session_state.messages
            ],
            max_tokens=max_tokens,
            stream=True
        )

        # Display streaming response
        with st.chat_message("assistant", avatar="🤖"):
            chat_responses_generator = generate_chat_responses(chat_completion)
            full_response = st.write_stream(chat_responses_generator)
    except Exception as e:
        st.error(e, icon="🚨")

    # Save response to history
    if isinstance(full_response, str):
        st.session_state.messages.append(
            {"role": "assistant", "content": full_response})
    else:
        combined_response = "\n".join(str(item) for item in full_response)
        st.session_state.messages.append(
            {"role": "assistant", "content": combined_response})
```

## Running the Application

1. Create a `.env` file with your Groq API key:
```env
GROQ_API_KEY=your_api_key_here
```

2. Run the Streamlit app:
```bash
streamlit run app.py
```

## Key Features

- **Multiple Model Support**: Choose from various LLMs including Gemma, LLaMA, and Mixtral
- **Streaming Responses**: Real-time display of AI responses
- **Configurable Parameters**: Adjust token limits based on model capabilities
- **Persistent Chat History**: Maintains conversation context within the session
- **User-Friendly Interface**: Clean layout with avatars and clear message distinction
- **Error Handling**: Graceful error display with visual indicators

## Technical Notes

- Uses Streamlit's session state for persistence
- Implements streaming responses for better user experience
- Dynamically adjusts token limits based on model selection
- Handles both string and non-string responses
- Clears chat history when switching models
- Loads API key securely from environment variables

## Security Considerations

- Store API keys in environment variables
- Use secure HTTPS connections
- Sanitize markdown content display
- Implement rate limiting if needed
- Consider adding user authentication for production use

## Customization Options

1. Add more models to the `models` dictionary
2. Customize the UI with different avatars or colors
3. Add conversation export functionality
4. Implement model comparison features
5. Add system prompts or conversation templates

## Troubleshooting

- Ensure all required packages are installed
- Verify API key is correctly set in `.env` file
- Check network connectivity for API calls
- Monitor token usage and adjust limits if needed
- Review Streamlit logs for debugging