# ShortCutGPT
by Gregory Z., info@extractum.io

An automated ChatGPT text processing that can be run on the selected text with a shortcut right on your Mac.
For example, you can easily rewrite any selected text in any application with just a single keystroke.

## Install the Automation:
1. Open Automator (Application->Automator)
2. Select "Quck Action"
3. At the top, for "Workflow receives current", select "text".
4. For "in", select "any application" or specify an application if you want.
5. Check the "Output replaces selected text" checkbox
6. Copy the the following source code into the Clipboard, Select "Run AppleScript", Paste the Clipboard content there

```
property scriptTimeout : 10 -- script timeout in seconds
property jqPath : "/opt/homebrew/bin/jq" -- Replace with your actual path to jq tool
property openaiEndpoint : "https://api.openai.com/v1/chat/completions"
property apiKey : "sk-..." -- Replace with your actual API key
property model : "gpt-4"
property systemPrompt : "As a helpful text processor, your task is to rectify any typos, syntax errors, and grammar issues, while also refining certain phrases to ensure they sound more native and fluent. It's essential to maintain the original topic, terminology, meaning of the text, as well as any formatting elements such as emojis and bullet points. Your goal is to enhance the text to make it read as if it were composed by a native US speaker."

on run {input, parameters}
	if input is {} then
		set userQuery to "London is the capital of great britain.
This is the line wit som typos."
	else
		-- Coerce input to string if it is provided
		set userQuery to (input as string)
	end if
	
	-- Escape new lines in userQuery for JSON
	set oldDelimiters to AppleScript's text item delimiters
	set AppleScript's text item delimiters to {"
"}
	set textItemList to every text item of userQuery
	set AppleScript's text item delimiters to {"\\n"}
	set userQuery to textItemList as string
	set AppleScript's text item delimiters to oldDelimiters
	
	-- Construct JSON payload
	set requestData to "{ \"model\": \"" & model & "\", \"messages\": [ { \"role\": \"system\", \"content\": \"" & systemPrompt & "\" }, { \"role\": \"user\", \"content\": \"" & userQuery & "\" } ] }"
	
	-- Prepare curl command
	set curlCommand to "curl -s " & openaiEndpoint & " -H \"Content-Type: application/json\" -H \"Authorization: Bearer " & apiKey & "\" -d " & quoted form of requestData & " | " & jqPath & " -r '.choices[0].message.content'"
	
	--return curlCommand
	
	-- Execute curl command and parse with jq
	with timeout of scriptTimeout seconds
		set parsedResponse to do shell script curlCommand
	end timeout
	
	-- Split the strings with any delimiter and join them back with the correct one otherwise AppleScript will turn all \n to \r
	set AppleScript's text item delimiters to return
	set theLines to text items of parsedResponse
	set AppleScript's text item delimiters to "
"
	set parsedResponse to theLines as string
	set theText to parsedResponse as string
	return parsedResponse
end run
```

*[!] Note: update the OpenAI KEY, model name and the path to jq tool.* 

7. File->Save and save it as "shortcutgpt"
8. Open System Settings->Keyboard->Shortcut
9. Expand lists and find "shortcutgpt" under the "text" section
10. Double-click on the "none" and press a combination of keys at once, then Click "Done" button
