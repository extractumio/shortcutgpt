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
-- Use case settings
property jqPath : "/opt/homebrew/bin/jq" -- Replace with your actual path to jq tool
property apiKey : "sk-..." -- Replace with your actual API key
property systemPrompt : "As a helpful text processor, your task is to rectify any typos, syntax errors, and grammar issues, while also refining certain phrases to ensure they sound more native and fluent. It's essential to maintain the original topic, terminology, meaning of the text, as well as any formatting elements such as emojis and bullet points. Your goal is to enhance the text to make it read as if it were composed by a native US speaker."
property model : "gpt-4"

-- System settings
property scriptTimeout : 10 -- script timeout in seconds
property openaiEndpoint : "https://api.openai.com/v1/chat/completions"


-- Define a function to append content to a file
on appendTextToFile(contentToAppend, filePath)
	try
		set fileRef to open for access filePath with write permission
		write contentToAppend & linefeed to fileRef starting at eof
		close access fileRef
		--display dialog "Content has been successfully appended to the file."
	on error errMsg
		close access fileRef
		display dialog "Error: " & errMsg
	end try
end appendTextToFile

-- Converts the selected original text into a valid JSON fragment.
on convertToJSON(inputText)
	set jsonString to quoted form of inputText
	set jsonString to my replaceText(jsonString, "\\", "\\\\\\\\")
	set jsonString to my replaceText(jsonString, "\"", "\\\"")
	set jsonString to my replaceText(jsonString, "/", "\\/")
	set jsonString to my replaceText(jsonString, character id 8, "\\b")
	set jsonString to my replaceText(jsonString, character id 12, "\\f")
	set jsonString to my replaceText(jsonString, linefeed, "\\n")
	set jsonString to my replaceText(jsonString, return, "\\r")
	set jsonString to my replaceText(jsonString, tab, "\\t")
	return text 2 thru -2 of jsonString -- removes the extra quotes added by quoted form of
end convertToJSON

-- Text replacer.
on replaceText(originalText, findText, replaceText)
	set AppleScript's text item delimiters to findText
	set theTextItems to text items of originalText
	set AppleScript's text item delimiters to replaceText
	set originalText to theTextItems as string
	set AppleScript's text item delimiters to ""
	return originalText
end replaceText

-- Selected text transformation handler.
on run {input, parameters}
	if input is {} then
		set userQuery to "London is the capital of great britain.
This is the line wit som typos."
	else
		-- Coerce input to string if it is provided
		set userQuery to (convertToJSON(input as string))
	end if
	
	
	-- Construct JSON payload
	set requestData to "{ \"model\": \"" & model & "\", \"messages\": [ { \"role\": \"system\", \"content\": \"" & systemPrompt & "\" }, { \"role\": \"user\", \"content\": \"" & userQuery & "\" } ] }"
	
	-- Prepare curl command
	set curlCommand to "curl -s " & openaiEndpoint & " -H \"Content-Type: application/json\" -H \"Authorization: Bearer " & apiKey & "\" -d " & quoted form of requestData & " | " & jqPath & " -r '.choices[0].message.content'"
	
	appendTextToFile("Returned from curl:
" & curlCommand, "/Users/greg/Desktop/debug_automator.txt")
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
