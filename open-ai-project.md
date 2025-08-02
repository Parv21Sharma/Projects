#python
import os
import io
import sys
import traceback
from dotenv import load_dotenv
load_dotenv()
from contextlib import redirect_stdout, redirect_stderr
import openai
try:
    openai.api_key = os.environ["OPENAI_API_KEY"]
    print("OpenAI API key loaded from .env")
except KeyError:
    print("OPENAI_API_KEY environment variable not set.")
    print("please make sure it is in .env file")
    sys.exit(1)
print("OpenAI API key setup complete.")
def execute_and_capture_errors(script_content):
    """
    Executes Python script content and captures stdout, stderr, and exceptions.
    Args:
        script_content (str): The Python code as a string.
    Returns:
        tuple: (stdout_output, stderr_output, exception_traceback_str, success_flag)
               success_flag is True if no exception, False otherwise.
    """
    stdout_capture = io.StringIO()
    stderr_capture = io.StringIO()
    exception_traceback = None
    success = True
    with redirect_stdout(stdout_capture), redirect_stderr(stderr_capture):
        try:
            exec(script_content)
        except Exception:
            exception_traceback = traceback.format_exc()
            success = False
        except SyntaxError as e:
            exception_traceback = str(e)
            success = False
    return stdout_capture.getvalue(), stderr_capture.getvalue(), exception_traceback, success
print("execute_and_capture_errors function defined.")
def get_ai_correction_suggestion(code, error_message):
    """
    Uses OpenAI GPT to suggest corrections for the given code and error.
    """
    prompt = f"""
    The following Python code encountered an error:
    {code}
    Error message:
    {error_message}
    Please analyze the code and the error, and provide a corrected version of the code.
    Explain the error and your corrections briefly.
    Output format:
    Explanation: [Your explanation of the error and correction]
    [Corrected Code]
    """
    try:
        response = openai.ChatCompletion.create(
            model="gpt-3.5-turbo",  # or gpt-4 if you have access
            messages=[
                {
                    "role": "system",
                    "content": "You are a helpful Python code debugging assistant. You identify errors and provide corrected code with explanations."
                },
                {
                    "role": "user",
                    "content": prompt
                }
            ],
            max_tokens=500,
            temperature=0.7,
        )
        return response.choices[0].message['content'].strip()
    except Exception as e:
        return f"Error communicating with OpenAI API: {e}"
script_to_debug = """
def my_function() # Missing colon here
    print("Hello")
my_function()
"""
print(f"--- Original Script ---")
print(script_to_debug)
print("\n" + "="*30 + "\n")
stdout, stderr, error_traceback, success = execute_and_capture_errors(script_to_debug)
if success:
    print("--- Script Executed Successfully ---")
    if stdout:
        print("\nStandard Output:")
        print(stdout)
    if stderr:
        print("\nStandard Error:")
        print(stderr)
else:
    print("--- Script Execution Failed ---")
    print("\nError Captured:")
    print(error_traceback)
    print("\n" + "="*30 + "\n")
    print("--- Requesting AI Correction ---")
    ai_suggestion = get_ai_correction_suggestion(script_to_debug, error_traceback)
    print("\nAI Suggestion:")
    print(ai_suggestion)

print("execute_and_capture_errors function defined.")


# output

--- Original Script ---

def my_function() # Missing colon here
    print("Hello")
my_function()


==============================

--- Script Execution Failed ---

Error Captured:
Traceback (most recent call last):
  File "C:\Users\Hp\AppData\Local\Temp\ipykernel_17756\1863789561.py", line 19, in execute_and_capture_errors
    exec(script_content)
  File "<string>", line 2
    def my_function() # Missing colon here
                      ^
SyntaxError: invalid syntax


==============================

--- Requesting AI Correction ---

AI Suggestion:
Explanation: 
The error occurred because the function definition is missing a colon at the end. In Python, a colon is required after the function signature to indicate the start of the function's body.

Corrected Code:
```python
def my_function():
    print("Hello")

my_function()
```
