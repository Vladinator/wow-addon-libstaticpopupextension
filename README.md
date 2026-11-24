LibStaticPopupExtension
==========================
This library shows popup dialogs that look and behave like the game's own `StaticPopup` dialogs, but without calling the game's native methods such as `StaticPopup_Show` or touching `StaticPopupDialogs`. This is to eliminate the risk of taint complications while still providing the behavior of popup dialogs. The API is meant as a drop-in replacement which means it should be very easy to implement in your own projects.

# Getting started

The library needs [LibStub](https://www.curseforge.com/wow/addons/libstub). Load the library file after it:

```
libs\LibStub\LibStub.lua
libs\LibStaticPopupExtension\LibStaticPopupExtension.lua
```

Then get the library like any other:

```lua
local SPE = LibStub("LibStaticPopupExtension-1.0", true)
```

All of the functions are called with a dot, like `SPE.Show(...)`. If the library isn't available you can fall back to the game's `StaticPopup_Show`.

# Showing a dialog

`local frame = SPE.Show(dialog[, text_arg1[, text_arg2[, data]]])`

`dialog` is a table in the same shape as an entry in `StaticPopupDialogs`, with a `which` field that identifies it. You can pass the table every time you show the dialog, the library remembers it by `which`. After that you can also pass just the `which` string.

```lua
local DIALOG = {
    which = "MYADDON_CONFIRM",
    text = "Do you want to reset %s?",
    button1 = YES,
    button2 = NO,
    hideOnEscape = true,
    OnAccept = function(frame, data)
        print("Reset", data)
    end,
}

SPE.Show(DIALOG, "your settings", nil, "some data")
```

The returned frame is the dialog, or `nil` if nothing is known by that `which`. Showing a dialog that is already visible cancels the visible one (with the reason `"override"`) and shows it again, unless `noCancelOnReuse` is set, then the visible one is just closed.

## text_arg1 and text_arg2

The text is formatted like the game does it, the first argument replaces the first `%s` and the second argument the second one. A missing argument becomes an empty string. If `text` is a function it is called every time the dialog is shown to get the text. Both arguments are also available in `frame.Text.text_arg1` and `frame.Text.text_arg2`.

## data

Anything you want to be given back to your handlers, it is stored in `frame.data`.

# Other functions

`SPE.Hide(which[, data])` closes every dialog with that `which`, or only the ones with the same `data` if you pass it.

`local name, frame = SPE.Visible(which[, data])` the name and the frame of the visible dialog, or nothing.

`local isOurs = SPE.IsFrame(frame)` tells if a frame is one of the dialogs of the library, for when you have a frame and don't know which system it came from.

# dialog

All of the fields are optional, apart from `which` and `text`. Any other field of the game's dialogs isn't supported and is ignored.

| Field | |
|---|---|
| `which` | The identifier of the dialog. |
| `text` | A string, or a function that returns one. It can have `%s` for the arguments. |
| `button1`, `button2` | The text of the buttons. A button without text is hidden. |
| `hasEditBox` | Shows an edit box. It gets the focus and its text is selected when the dialog is shown. |
| `editBoxWidth` | The width of the edit box, 130 by default. Over 260 the dialog grows by the difference, like the game's, so `350` is the usual wide edit box. |
| `maxLetters`, `countInvisibleLetters` | Limit the text of the edit box, `0` removes the limit. |
| `hideOnEscape` | Escape closes the dialog and calls `OnCancel` with `"clicked"`. Like the game, one press closes every dialog of the library that has this set, the newest first. |
| `noCancelOnEscape` | Escape closes the dialog without calling `OnCancel`. |
| `enterClicksFirstButton` | Enter clicks the first button that can be clicked. |
| `wideText` | A wider text (360, normally 290) for long texts. |
| `timeout` | Seconds until the dialog cancels itself with `"timeout"`, `0` or `nil` for never. |
| `sound` | A sound kit id played when the dialog is shown, on top of the sound the game always plays. |
| `hideSound` | A sound kit id played when `button1` is clicked with Enter (`enterClicksFirstButton`), on top of the sound the game always plays when closing. |
| `noCancelOnReuse` | See [above](#showing-a-dialog). |

## handlers

The handlers are called with the dialog frame and the same arguments as the game gives them.

| Handler | |
|---|---|
| `OnShow(frame, data)` | The dialog is shown and has its text and edit box set up. |
| `OnHide(frame, data)` | The dialog is hidden, whatever the reason. Clean up here. |
| `OnAccept(frame, data, data2)` | `button1` is clicked. Return `true` to keep the dialog open. |
| `OnCancel(frame, data, reason)` | `button2` is clicked, Escape is pressed, the dialog times out or it is shown again. The reason is `"clicked"`, `"timeout"` or `"override"`. Return `true` to keep the dialog open when a button was clicked. |
| `EditBoxOnEnterPressed(editBox, data)` | Enter is pressed in the edit box. Nothing else happens on Enter unless you set `enterClicksFirstButton`. |
| `EditBoxOnEscapePressed(editBox, data)` | Escape is pressed in the edit box. Nothing else happens on Escape, so close the dialog from the handler if you want it to. |
| `EditBoxOnTextChanged(editBox, data)` | The text of the edit box changed. |

# The frame

The frame has the same accessors as the game's dialog frames, so code that works with those keeps working:

| | |
|---|---|
| `frame.which`, `frame.data`, `frame.data2` | What it was shown with. |
| `frame.OnAcceptCallback` | Free for you to use, it is set to `nil` every time the dialog is shown. |
| `frame.Text`, `frame:GetTextFontString()` | The text. |
| `frame.EditBox`, `frame:GetEditBox()` | The edit box. |
| `frame.Button1`, `frame.Button2`, `frame:GetButton(index)`, `frame:GetButton1()`, `frame:GetButton2()` | The buttons. |

The frames are reused after they are closed, so don't keep hold of one longer than the dialog is visible.

# Stacking

The dialogs of the library always go below the popups of the game that are shown. The library hooks the functions of the game that show and hide popups (`StaticPopup_SetUpPosition`, `StaticPopup_OnHide`, `StaticPopupSpecial_Show` and `StaticPopupSpecial_Hide`) and moves your dialogs one frame later, so it never runs inside the game's own code. The game leaves a gap when one of its popups closes, so your dialogs are placed below the lowest popup of the game that is shown, wherever that is. The dialogs of the library stack below each other the same way.

# Typing

The library types itself: `LibStaticPopupExtension-1.0_Dialog` is the dialog and `LibStaticPopupExtension-1.0_Frame` is the frame.
