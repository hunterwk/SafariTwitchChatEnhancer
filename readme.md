This is a Applescript designed for people who enjoy using Safari as their main browser but miss the chat experience of BetterTwitchTv and FrankerFaceZ.

Requirements:
Safari
Google Chrome

The script searches through all open tabs in all open Safari windows for any with a title containing "twitch".  If more than one are present, a dialog box appears which allows you to select which tab contains the stream you wish to watch and then proceeds to open a pop out chat in Google Chrome.

The raw code is below:
```
tell application "Safari"
	activate
	set winlist to every window
	set winmatchlist to {}
	set tabmatchlist to {}
	set tabnamematchlist to {}
	set twitch to "twitch"
	repeat with win in winlist
		set ok to true
		try
			set tablist to every tab of win
		on error errmsg
			set ok to false
		end try
		if ok then
			repeat with t in tablist
				if twitch is in (name of t as string) then
					set end of winmatchlist to win
					set end of tabmatchlist to t
					set end of tabnamematchlist to (id of win as string) & "." & (index of t as string) & ".  " & (name of t as string)
					
				else if twitch is in (URL of t as string) then
					set end of winmatchlist to win
					set end of tabmatchlist to t
					set end of tabnamematchlist to (id of win as string) & "." & (index of t as string) & ".  " & (name of t as string)
					
				end if
			end repeat
		end if
	end repeat
	if (count of tabmatchlist) = 1 then
		set w to item 1 of winmatchlist
		set t to item 1 of tabmatchlist
		set current tab of w to t
		set index of w to 1
	else if (count of tabmatchlist) = 0 then
		display dialog "No matches"
	else
		set whichtab to choose from list of tabnamematchlist with prompt "The following tabs match, please select one:"
		set AppleScript's text item delimiters to "."
		if whichtab is not equal to false then
			set tmp to text items of (whichtab as string)
			set w to (item 1 of tmp) as integer
			set t to (item 2 of tmp) as integer
			set current tab of window id w to tab t of window id w
			set index of window id w to 1
		end if
	end if
	set linkURL to URL of current tab of window 0
end tell


on trimText(theText, theCharactersToTrim, theTrimDirection)
	set theTrimLength to length of theCharactersToTrim
	if theTrimDirection is in {"beginning", "both"} then
		repeat while theText begins with theCharactersToTrim
			try
				set theText to characters (theTrimLength + 1) thru -1 of theText as string
			on error
				return ""
			end try
		end repeat
	end if
	if theTrimDirection is in {"end", "both"} then
		repeat while theText ends with theCharactersToTrim
			try
				set theText to characters 1 thru -(theTrimLength + 1) of theText as string
			on error
				return ""
			end try
		end repeat
	end if
	return theText
end trimText

set removeT to "https://www.twitch.tv/"
set linkURL2 to trimText(linkURL, "https://www.twitch.tv/", "beginning")

set linkURL3 to do shell script "echo " & quoted form of linkURL2 & " | tr -dc '[:alnum:]' | tr '[:upper:]' '[:lower:]'"

set theURL to "https://www.twitch.tv/popout/" & linkURL3 & "/chat?popout="

tell application "Google Chrome"
	activate
	set the URL of active tab of (make new window) to theURL
	set bounds of front window to {0, 0, 500, 1400}
end tell
```