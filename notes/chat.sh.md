./chat - A minimal curl-based chatbot with ability to script and run code (tutorial && code)

I've been thinking a lot about "Life Copilots" && automating automations and recently discovered Bash Scripting while looking for ways to run code on my devices without root

To my surprise, Bash has been around since the 80s and works on basically anything that can run or emulate a shell including Linux but also Windows, macOS, Android, iOS, robots, drones, smart devices, and everything else new and old!

Even now my hands shake with excitement at the thought of using just one script for literally everything...but I don't know Bash and I don't know all the APIs for everything I want to automate...

...so I've started working on `./chat`, a prototype chatbot client in Bash with no dependencies besides `curl` (and an AI model) in \~200 lines of code

\*\*🚨 Please Note: \*\*This is very experimental and extremely risky...use a sandbox, container, or virtual machine to experiment

# Features

* No dependencies except `curl` and an AI model
* Chat histories and logs
* Write and execute shell code
* Pipe and chain `./chat` with itself or any other command

# What can it do?

* Chat inside the terminal
* Create, run, and automate other Bash scripts
* Install packages and automate your shell
* Autonomously write and execute Python, NodeJS, etc
* Make and compile files
* Improve it's own source code
* Technically you should be able to create agents that act on their own using a simple `plan -> execute -> review` agent loop

# Setup

1. Copy the script below into a file called chat: `touch chat`
2. Make it executable: `chmod +x` `chat`
3. Run `./chat` for first time to add your API and model
   1. Get an API key from https://openrouter.ai/models
      1. Want to run your own models instead? Start here and see below for connecting your own model:
   2. Pick a model, see here for free ones: https://openrouter.ai/models?max\_price=0
      1. Copy+paste company/model, eg: `google/gemini-pro-1.5-exp`
   3. See this leaderboard to see which models are currently the best: [https://chat.lmsys.org/?leaderboard](https://chat.lmsys.org/?leaderboard)
   4. Ask me for suggestions below with your use case
   5. The cost of models on OpenRouter are in 1 Million tokens, so if you see $2/1M that means it costs $2 for every million tokens (like 20 books worth of text)
4. Chat: `./chat "Your prompt here"`
5. Clear history and logs: `./chat --clear`
6. Temporarily change model: `./chat "Your prompt" --model "org/model-name"`

**If #3 doesn't work, create a chat.config file that looks like this:**

`OPENROUTER_API_KEY='sk-or-v1-xxxxxxxxxxxxx'`  
`OPENROUTER_MODEL='openai/gpt-4o-2024-08-06'`

# Basic Usage

    # Simple chat
    ./chat "Hello! What can you do?"
    
    # Create and run a script
    ./chat "Write a bash script that lists all .txt files in the current directory"

# Advanced Usage

🍒 C**herry-picked results:** The examples below were handpicked to demonstrate capabilities, it's not guaranteed to produce working code 100% yet

    # Generate a Python script and then explain it
    ./chat "Write a Python script that calculates the Fibonacci sequence" | ./chat "Explain the Python code"
    
    # Use different models in a pipeline
    ./chat "Write a short story about a robot" --model "anthropic/claude-3-opus-20240229" | 
    ./chat "Summarize it in one sentence" --model "openai/gpt-3.5-turbo"
    
    # Generate and analyze code
    ./chat "Write a simple Java class for a bank account" | 
    ./chat "Analyze the Java code for potential improvements"
    
    # Multi-step task: Generate, translate, and summarize
    ./chat "Write a short story about AI" | 
    ./chat "Translate it to French" | 
    ./chat "Summarize it in English"

# Integration with Unix tools

    # Save output to a file
    ./chat "Write a short report on climate change" > climate_report.txt
    
    # Count words in generated content
    ./chat "Write a haiku about the singularity" | tee >(wc -w)
    
    # Process file contents
    cat complicated_code.py | ./chat "Explain this Python code and suggest optimizations"
    
    # Data analysis
    cat large_dataset.csv | ./chat "Analyze this CSV data and provide insights"

# What about offline or custom models?

This script was intentionally kept simple so you can adapt it to work with any server and model, including offline and self hosted ones

Almost all the leading Large Langauge Models are compatible with the OpenAI API, which is what this script is built around. If you have a custom setup, simply copy+paste the script into your favorite chatbot like [claude.ai](http://claude.ai), [gemini.google.com](http://gemini.google.com), or [chatgpt.com](http://chatgpt.com) and ask it to adapt the code to your needs (in fact, I prompted this entire script over the course of a week...I didn't actually write it myself)

Setting up your own models will require another tutorial, but you can get started here:

* r/Oobabooga
* r/LMStudio
* r/LocalLLaMA
* r/LLMDevs

# How does it work?

Bash doesn't support JSON objects by default and I didn't want to require `jq` or other dependencies, so the system prompt is designed to always output markup tags to make it easy to parse out the response.

* Anything inside <bot>...</bot> is extracted and emitted into the terminal
* Anything inside <bash>...</bash> is actually interpreted by Bash line-by-line

# The Script

**Copy+paste the script below or clone from Github:** [https://github.com/ozfromscratch/chat](https://github.com/ozfromscratch/chat)

\*\*🚨 BE CAREFUL: \*\*This script acts on your behalf with all your permissions, it can even elevate your permissions

    #!/bin/bash
    
    SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
    CONFIG_FILE="$SCRIPT_DIR/chat.config"
    HISTORY_FILE="$SCRIPT_DIR/chat.history"
    LOG_FILE="$SCRIPT_DIR/chat.logs"
    
    # System prompt as a variable for easy editing
    SYSTEM_PROMPT="You are in a Linux Terminal. ALWAYS respond using <bot>response goes here</bot> markup. You MUST ALWAYS include executable bash scripts within <bash>bash script here</bash> tags. NEVER use backticks for code, ALWAYS use <bash> tags as these will be parsed out and executed. When asked to create files etc, assume the current directory. Always escape quotes since you're in a terminal. REMEMBER: YOU MUST ALWAYS OUTPUT THE ACTUAL CODE IN <bash> TAGS, NOT JUST DESCRIBE IT. The content may be | piped...DO NOT output any extra content like 'Done!' or 'Ok' if there is code, just output the tags with the code so it can run. NEVER use &gt; or &lt; ALWAYS output the actual characters, escape with slash if needed"
    
    # Function to load or prompt for configuration
    load_or_prompt_config() {
        if [ -f "$CONFIG_FILE" ]; then
            source "$CONFIG_FILE"
        fi
    
        if [ -z "$OPENROUTER_API_KEY" ]; then
            read -p "Enter your OpenRouter API key: " OPENROUTER_API_KEY >&2
            echo "OPENROUTER_API_KEY='$OPENROUTER_API_KEY'" >> "$CONFIG_FILE"
        fi
    
        if [ -z "$OPENROUTER_MODEL" ]; then
            read -p "Enter the OpenRouter model (e.g., openai/gpt-3.5-turbo): " OPENROUTER_MODEL >&2
            echo "OPENROUTER_MODEL='$OPENROUTER_MODEL'" >> "$CONFIG_FILE"
        fi
    }
    
    # Function to send message to OpenRouter API
    send_message() {
        local messages="$1"
        local model="$2"
        local response=$(curl -s "https://openrouter.ai/api/v1/chat/completions" \
            -H "Content-Type: application/json" \
            -H "Authorization: Bearer $OPENROUTER_API_KEY" \
            -d '{
                "model": "'"$model"'",
                "messages": '"$messages"'
            }')
        
        echo "$response"
    }
    
    # Function to update history
    update_history() {
        local role="$1"
        local content="$2"
        echo "<$role>$content</$role>" >> "$HISTORY_FILE"
    }
    
    # Function to update logs
    update_logs() {
        local content="$1"
        echo "$content" >> "$LOG_FILE"
    }
    
    # Function to clear history and logs
    clear_history_and_logs() {
        if [ -f "$HISTORY_FILE" ]; then
            rm "$HISTORY_FILE"
            echo "Chat history cleared." >&2
        else
            echo "No chat history found." >&2
        fi
        if [ -f "$LOG_FILE" ]; then
            rm "$LOG_FILE"
            echo "Chat logs cleared." >&2
        else
            echo "No chat logs found." >&2
        fi
    }
    
    # Function to execute bash scripts found in the response
    execute_bash_scripts() {
        local response="$1"
        local is_piped="$2"
        
        if [ "$is_piped" = true ]; then
            # If piped, only output the content of bash tags
            echo "$response" | sed -n '/<bash>/,/<\/bash>/p' | sed 's/<bash>//g; s/<\/bash>//g'
        else
            echo "$response"  # Output the entire response
            
            # Extract and execute all bash scripts
            local in_bash=false
            local bash_script=""
    
            while IFS= read -r line; do
                if [[ $line == *"<bash>"* ]]; then
                    in_bash=true
                    bash_script=""
                elif [[ $line == *"</bash>"* ]]; then
                    in_bash=false
                    # Unescape quotes before execution for all bash commands
                    bash_script=$(echo "$bash_script" | sed 's/\\"/"/g')
                    eval "$bash_script"
                elif $in_bash; then
                    bash_script+="$line"$'\n'
                fi
            done <<< "$response"
        fi
    }
    
    # Main chat function
    chat() {
        local user_input="$1"
        local model="$2"
        local is_piped="$3"
        
        # Load history if exists
        if [ -f "$HISTORY_FILE" ]; then
            history=$(cat "$HISTORY_FILE")
        else
            history=""
        fi
    
        # Prepare the messages for the API
        local messages="["
        messages+="{\"role\":\"system\",\"content\":\"<system>$SYSTEM_PROMPT</system>\"},"
        while IFS= read -r line || [ -n "$line" ]; do
            if [[ $line =~ \<user\>(.*)\</user\> ]]; then
                messages+="{\"role\":\"user\",\"content\":\"<user>${BASH_REMATCH[1]}</user>\"},"
            elif [[ $line =~ \<bot\>(.*)\</bot\> ]]; then
                messages+="{\"role\":\"assistant\",\"content\":\"<bot>${BASH_REMATCH[1]}</bot>\"},"
            fi
        done <<< "$history"
        messages+="{\"role\":\"user\",\"content\":\"<user>$user_input</user>\"}]"
    
        # Send message and get response
        local full_response=$(send_message "$messages" "$model")
        
        # Log the raw server response
        update_logs "$full_response"
    
        # Extract the bot's response from the full server response
        local bot_response=$(echo "$full_response" | sed -n 's/.*<bot>\(.*\)<\/bot>.*/\1/p' | sed 's/\\n/\n/g')
        
        # Execute any bash scripts found in the response and output other content
        execute_bash_scripts "$bot_response" "$is_piped"
    
        # Update history with user input and assistant's response
        update_history "user" "$user_input"
        update_history "bot" "$bot_response"
    }
    
    # Load or prompt for configuration
    load_or_prompt_config
    
    # Initialize variables
    clear_flag=false
    model="$OPENROUTER_MODEL"
    user_input=""
    
    # Parse command line arguments
    while [[ $# -gt 0 ]]; do
        case $1 in
            --clear)
                clear_flag=true
                shift
                ;;
            --model)
                model="$2"
                shift 2
                ;;
            *)
                user_input+=" $1"
                shift
                ;;
        esac
    done
    
    # Check for --clear flag
    if $clear_flag; then
        clear_history_and_logs
        exit 0
    fi
    
    # Run chat function with input from command line or pipe
    if [ -p /dev/stdin ]; then
        # Input is coming from a pipe
        piped_input=$(cat)
        if [ -n "$user_input" ]; then
            chat "Context: $piped_input\n\nTask: $user_input" "$model" false
        else
            chat "$piped_input" "$model" false
        fi
    else
        # Trim leading and trailing spaces from user input
        user_input="${user_input#"${user_input%%[![:space:]]*}"}"
        user_input="${user_input%"${user_input##*[![:space:]]}"}"
    fi
    
    if [ -z "$user_input" ]; then
        echo "Usage: $0 [--clear] [--model MODEL] \"Your message here\"" >&2
        echo "       echo \"Your message here\" | $0 [--clear] [--model MODEL]" >&2
        exit 1
    fi
    
    # Check if the script is being piped
    if [ -p /dev/stdout ]; then
        is_piped=true
    else
        is_piped=false
    fi
    
    chat "$user_input" "$model" "$is_piped"
