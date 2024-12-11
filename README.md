# copilot-more

`copilot-more` maximizes the value of your GitHub Copilot subscription by exposing models like gpt-4o and Claude-3.5-Sonnet for use in agentic coding tools such as Cline, or any tool that supports bring-your-own-model setups. Unlike costly pay-as-you-go APIs, this approach lets you leverage these powerful models affordably. (Yes, $10 per month maximum.)

The exposed models aren't limited to coding tasks—you can connect any AI client and customize parameters like temperature, context window length, and more.

# Ethical Use
- Respect the GitHub Copilot terms of service.
- Minimize the use of the models for non-coding purposes.
- Be mindful of the risk of being banned by GitHub Copilot for misuse.


## 🏃‍♂️ How to Run

1. Get the refresh token

   A refresh token is used to get the access token. This token should never be shared with anyone :). You can get the refresh token by following the steps below:

    - Run the following command and note down the returned `device_code`, `user_code` and the next command you need to run to get the access_token.:

    ```bash
    # 01ab8ac9400c4e429b23 is the client_id for the VS Code in Github, it's a fixed value, no need to change.
    curl https://github.com/login/device/code -X POST -d 'client_id=01ab8ac9400c4e429b23&scope=user:email' | (grep -o 'device_code=[^&]*\|user_code=[^&]*' | sed 's/=/: /'; echo "Next command:"; echo "curl https://github.com/login/oauth/access_token -X POST -d 'client_id=01ab8ac9400c4e429b23&scope=user:email&device_code=$(grep -o 'device_code=[^&]*' <<< \"$(\!)\" | cut -d= -f2)&grant_type=urn:ietf:params:oauth:grant-type:device_code'| | grep -o 'access_token=[^&]*' | cut -d= -f2 | sed 's/^/$env:REFRESH_TOKEN = "/' | sed 's/$/"/'
")
    ```

    - Open https://github.com/login/device/ and enter the `user_code`.

    - Run the "next command"


2. Install and run copilot_more

    ```bash
    git clone https://github.com/jjleng/copilot-more.git
    # Get into copilot-more root folder
    cd copilot-more
    
    # install dependencies
    pip install poetry
    poetry install
    
    # run the server.
       # 1. Set environment variable, use the output from the "next command".
         # In Windows Powershell
            $env:REFRESH_TOKEN =gho_xxxxx
         # In Linux
            REFRESH_TOKEN=gho_xxxxx
       # 2. Set up the server, you can use any port number you want.
          poetry run uvicorn copilot_more.server:app --port 15432
    ```


3. AI IDE Configuration
   * API Provider: OpenAI Compatible
   * Base URL http://localhost:15432
   * API Key: Anything
   * Model ID: claude-3.5-sonnet


## Limitation

The GH Copilot models sit behind an API server that is not fully compatible with the OpenAI API. You cannot pass in a message like this:

```json
    {
      "role": "user",
      "content": [
        {
          "type": "text",
          "text": "<task>\nreview the code\n</task>"
        },
        {
          "type": "text",
          "text": "<task>\nreview the code carefully\n</task>"
        }
      ]
    }
```
copilot-more takes care of this limitation by converting the message to a format that the GH Copilot API understands. However, without the `type`, we cannot leverage the models' vision capabilities, so that you cannot do screenshot analysis.
