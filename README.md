<details><summary><b>Intro</b></summary>  
This is API for [[CudaText]] in Python.

* Module: cudatext. Constants, functions, class Editor, objects of class Editor.
* Module: cudatext_cmd. Constants for Editor.cmd().
* Module: cudatext_keys. Constants for on_key.
* Module: cudax_lib. High-level functions for plugins, e.g. read/write configs.
* Module: cudax_nodejs. Helper functions for plugins which use Node.js.

</details>

---
<details><summary><b>Event plugins</b></summary>  
<details><summary>&lt;descr&gt;</summary>  
To make plugin react to some program event, add method to class Command. For example, method "Command.on_save" will be called by program event "on_save" - if plugin is subscribed to that event. Event methods have param "ed_self": this is Editor object in which event occured. This object is the same as object "ed" in most cases ("ed" always refers to focused editor), but in some cases event occurs in inactive editor, e.g. command "Save all tabs" calls event "on_save" for passive tabs.

Plugin can subscribe to events in several ways:

* Static subscribing in "install.inf". This is used, when plugin needs to always react to some events, with any combination of its options.

* Static subscribing via config file "settings/plugins.ini". This is used when plugin doesn't need some events initially, but after changing some options, it needs more events. File "plugins.ini" section [events] can contain lines like "cuda_modulename=event1,event2,...". File "plugins.ini" is read on plugins initialization, so after writing there, CudaText should be restarted.

* Run-time subscribing/unsubscribing via API. See description of action PROC_SET_EVENTS in app_proc().

Event handlers (Command.on_nnnnn methods) return-value is ignored is many cases, but sometimes return value True or False has special meaning. Usually the False return value blocks propagation of event to other plugins, so don't return False if not necessary.

</details>
<details><summary>* <b>Events - General</b> &nbsp; &nbsp; [Event plugins > Events - General]</summary>  

* on_open(self, ed_self): Called after file is opened from disk.
* on_open_pre(self, ed_self, filename): Called before file opening. Method can return False to disable opening, other value is ignored.
* on_open_none(self, ed_self): Called after file is opened, and none lexer was detected/set for it.
* on_close_pre(self, ed_self): Called before closing tab, before checking if tab modified or not. Method can return False to disable closing.
* on_close(self, ed_self): Called before closing tab, after modified file was saved, and editor is still active.
* on_save(self, ed_self): Called after saving file.
* on_save_pre(self, ed_self): Called before saving file. Method can return False to disable saving, other value is ignored.
* on_save_naming(self, ed_self): Called before saving untitled tab, to get suggested filename without path and extension. Method must return valid filename string or None.
* on_start(self, ed_self): Called once on program start. ed_self is None.
* on_exit(self, ed_self): Called on exiting application. This event is lazy, ie it's called only for already loaded plugins. ed_self is None.
* on_app_activate(self, ed_self): Called when application window gets focus. ed_self is None.
* on_app_deactivate(self, ed_self): Called when application window looses focus. ed_self is None.

</details>
<details><summary>* <b>Events - Tabs</b> &nbsp; &nbsp; [Event plugins > Events - Tabs]</summary>  

* on_tab_change(self, ed_self): Called after active tab is changed.
* on_tab_move(self, ed_self): Called after closing a tab (another tab is already activated), or moving a tab (by drag-n-drop, or UI command).

</details>
<details><summary>* <b>Events - Editor</b> &nbsp; &nbsp; [Event plugins > Events - Editor]</summary>  

* on_change(self, ed_self): Called after editor text is changed.
* on_change_slow(self, ed_self): Called after editor text is changed, and few seconds (option) passed. Used in CudaLint plugin, it needs to react to change after a delay.
* on_caret(self, ed_self): Called after editor caret position and/or selection is changed.
* on_insert(self, ed_self, text): Called before inserting a text. Method can return False to disable insertion, other return value is ignored.
* on_key(self, ed_self, key, state): Called when user presses a key in editor. Param "key" is int key code; values are listed in the module cudatext_keys. Param "state" is string of chars: "a" if Alt pressed, "c" if Ctrl pressed, "s" if Shift pressed, "m" if Meta (Windows-key) pressed. Method can return False to disable key processing, other return value is ignored.
* on_key_up(self, ed_self, key, state): Called when user depresses a key in editor. Params meaning is the same as in "on_key". Currently called only for Ctrl/Shift/Alt keys (to not slow down).
* on_focus(self, ed_self): Called after any editor gets focus.
* on_lexer(self, ed_self): Called after editor's lexer is changed.
* on_lexer_parsed(self, ed_self): Called after lexer has finished its parsing. To not slow down usual work, event is called only if parsing time was >=600 msec.
* on_paste(self, ed_self, keep_caret, select_then): Called before some Clipboard Paste command runs. Parameters are options of various paste commands. Method can return False to disable default operation.
* on_scroll(self, ed_self): Called on scrolling in editor.
* on_mouse_stop(self, ed_self, x, y): Called when mouse cursor stops (for a short delay) over editor. Params "x", "y" are mouse editor-related coords.
* on_hotspot(self, ed_self, entered, hotspot_index): Called when mouse cursor moves in/out of hotspot. See ed.hotspots() API.
* on_state(self, ed_self, state): Called after some app state is changed. Param "state" is one of APPSTATE_nnnn constants. ed_self is None.
* on_state_ed(self, ed_self, state): Called after some editor state is changed. Param "state" is one of EDSTATE_nnnn constants.
* on_snippet(self, ed_self, snippet_id, snippet_text): Called when user chooses snippet to insert, in ed.complete_alt() call.

</details>
<details><summary>* <b>Events - Editor clicks</b> &nbsp; &nbsp; [Event plugins > Events - Editor clicks]</summary>  

* on_click(self, ed_self, state): Called after mouse click on text area. Param "state": same meaning as in on_key.
* on_click_dbl(self, ed_self, state): Called after mouse double-click on text area. Param "state": same meaning as in on_key. Method can return False to disable default processing.
* on_click_gutter(self, ed_self, state, nline, nband): Called on mouse click on gutter area. Param "state": same as in on_key. Param "nline": index of editor line. Param "nband": index of gutter band. Method can return False to disable default processing.
* on_click_gap(self, ed_self, state, nline, ntag, size_x, size_y, pos_x, pos_y): Called after mouse click on inter-line gap: see Editor.gap(). Param "state": same meaning as in on_key. Other params: properties of clicked gap.
* on_click_link(self, ed_self, state, link): Called after link/URL in the editor is activated by click or double-click (event is not called on URL opening from Command Palette or from editor context menu). Param "state": same meaning as in on_key. Param "link": link string. Method can return False to disable default program action.

</details>
<details><summary>* <b>Events - Smart commands</b> &nbsp; &nbsp; [Event plugins > Events - Smart commands]</summary>  

* on_complete(self, ed_self): Called by "auto-completion menu" command (default hotkey: Ctrl+Space). Method should read editor text, find all needed data, and call Editor.complete() API to show the actual auto-completion listbox to user. Method must return True if it handled the command, otherwise None.
* on_func_hint(self, ed_self): Called by "show function-hint" command (default hotkey: Ctrl+Shift+Space). Method should return function-hint string (comma-separated parameters, brackets are optional). Empty string or None mean that no hint was found.
* on_goto_def(self, ed_self): Called by "go to definition" command (e.g. by mouse shortcut or by menu item in the editor context menu). Method must return True if it handled the command, otherwise None.
* on_goto_enter(self, ed_self, text): Called after user entered text in the Go To dialog. Method can return False to disable default processing.

</details>
<details><summary>* <b>Events - Panels</b> &nbsp; &nbsp; [Event plugins > Events - Panels]</summary>  

* on_message(self, ed_self, id, text): Called on showing a text in the statusbar. Param "id" means statusbar column index, currently only value 0 is used ("text message" statusbar cell). Method can return False to disable usual showing of message. Param "ed_self" is None here.
* on_console_nav(self, ed_self, text): Called on double-clicking line, or calling context menu item "Navigate", in Python Console panel. Param "text" is line with caret. Param "ed_self" is None here.
* on_output_nav(self, ed_self, text, tag): Called on clicking line, or pressing Enter, in the Output or Validate panel. Param "text" is clicked line, param "tag" is int value associated with line. Event is called only if app cannot parse output line by itself using given regex, or regex is not set. Param "ed_self" is None here.

</details>
<details><summary>* <b>Events - Macros</b> &nbsp; &nbsp; [Event plugins > Events - Macros]</summary>  

* on_macro(self, ed_self, text): Called when command "macros: stop recording" runs. In text the "\n"-separated list of macro items is passed. These items were run after command "macros: start recording" and before command "macros: stop recording".
** if item is "number" then it's simple command.
** if item is "number,string" then it's command with str parameter (usually it's command cCommand_TextInsert).
** if item is "py:string_module,string_method,string_param" then it's call of Python plugin (usually string_param is empty).

Note: To run each on_macro item (with number) later, call ed.cmd(): number is command code, string after comma is command text.

</details>
<details><summary>* <b>Events priority</b> &nbsp; &nbsp; [Event plugins > Events priority]</summary>  

By default all events in all plugins have priority=0. So for any event, if 2+ plugins handle this event, they're called in the order on module names (e.g. cuda_aa, then cuda_dd, then cuda_mmm).

You may want to handle some event first.
To change priority for your plugin event, write in install.inf event like this: "on_key+", "on_key++"... with any number of "+" up to 4. Each "+" makes higher priority. So first called "on_key" with maximal number of "+", last called "on_key" without "+".

</details>

</details>

---
<details><summary><b>Callback param</b></summary>  

Callback parameter is supported in several API functions: dlg_proc, timer_proc, menu_proc, button_proc (maybe more later).

Parameter can be in these forms:

* callable, i.e. name of a function
* string "module=mmm;cmd=nnn;" - to call method nnn (in class Command) in plugin mmm, where mmm is usually sub-dir in the "cudatext/py", but can be any module name
* string "module=mmm;cmd=nnn;info=iii;" - the same, and callback will get param "info" with your value
* string "mmm.nnn" - the same, to call method, short form
* string "module=mmm;func=nnn;" - to call function nnn in root of module mmm
* string "module=mmm;func=nnn;info=iii;" - the same, and callback will get param "info" with your value

For example:

* If you need to call function "f" from plugin cuda_my, from file "py/cuda_my/__init__.py", callback must be "module=cuda_my;func=f;"
* If you need to call it from file "py/cuda_my/lib/mod.py", callback must be "module=cuda_my.lib.mod;func=f;".

Value after "info=" can be of any type, e.g. "info=20;" will pass int 20 to callback, "info='my';" will pass string 'my'.

</details>

---
<details><summary><b>Global funcs</b></summary>  
<details><summary>&lt;descr&gt;</summary>  

</details>
<details><summary>* <b>version</b> &nbsp; &nbsp; [Global funcs > version]</summary>  

app_exe_version()

Returns version of program. String, 4 dot-separated numbers.

app_api_version()

Returns version of API. String, 3 dot-separated numbers.

API

Constant, int. It is the last number from app_api_version() complex result.

Example of version check:

<syntaxhighlight lang="python">
from cudatext import *
if API < 350:
msg_box('Plugin needs newer program version', MB_OK+MB_ICONWARNING)
</syntaxhighlight>

</details>
<details><summary>* <b>app_path</b> &nbsp; &nbsp; [Global funcs > app_path]</summary>  

app_path(id)

Returns file-system path. Possible values of "id":

* APP_DIR_EXE: Folder of program executable.
* APP_DIR_SETTINGS: Folder "settings" (it should contain user.json file).
* APP_DIR_SETTINGS_DEF: Folder "settings_default".
* APP_DIR_DATA: Folder "data".
* APP_DIR_PY: Folder "py" with Python files.
* APP_DIR_INSTALLED_ADDON: Folder of last installed addon (for plugins it is folder in "py", for data-files it is folder in "data", for lexers it is folder "lexlib"). This dir is updated only if addon installed via file_open() or from app (Open dialog or command line).
* APP_FILE_SESSION: Filename of current session. Default filename is "history session.json" without path. Missing path means that folder "settings" will be used.
* APP_FILE_RECENTS: Str: "\n"-separated filenames of recent files.

</details>
<details><summary>* <b>app_proc</b> &nbsp; &nbsp; [Global funcs > app_proc]</summary>  
<details><summary>&lt;descr&gt;</summary>  
app_proc(id, text)

Performs application-wide action.

Parameter "text" is usually string, but can be of other types (bool, int, float, tuple/list of simple types).

===================

</details>
<details><summary>* * <b>Misc properties</b> &nbsp; &nbsp; [Global funcs > app_proc > Misc properties]</summary>  

* PROC_GET_LANG: Returns string id of active app translation. E.g. "ru_RU" if translation file is "ru_RU.ini", or "translation template" if such file is used. Returns empty string if built-in English translation is used.
* PROC_GET_GROUPING: Returns grouping mode in program, one of GROUPS_nnnn int contants.
* PROC_SET_GROUPING: Sets grouping mode in program, one of GROUPS_nnnn int contants.
* PROC_PROGRESSBAR: Changes state of the progress-bar (hidden by default), which is located on the status-bar right corner. Int value<0: progressbar hides, value in 0..100: progressbar shows with the given value.
* PROC_GET_TAB_IMAGELIST: Returns int handle of imagelist, for file-tabs. Use imagelist_proc() to work with it. Use PROP_TAB_ICON property to get/set file-tab icon index (index in this imagelist).
* PROC_GET_CODETREE: Returns int handle of code-tree UI control. Use tree_proc() to work with it.
* PROC_SET_FOLDER: Sets path of folder which will be used as initial folder in program's Open/Save-as dialogs.
* PROC_WINDOW_TOPMOST_GET: Returns bool flag: main window has "always on top" style.
* PROC_WINDOW_TOPMOST_SET: Sets bool flag: main window has "always on top" style.

* PROC_CONFIG_READ: Reads param "text" as contents of CudaText JSON config file.
* PROC_CONFIG_NEWDOC_EOL_GET: Returns str: line endings of newly created documents (app option "newdoc_ends").
* PROC_CONFIG_NEWDOC_EOL_SET: Sets str: line endings of newly created documents.
* PROC_CONFIG_NEWDOC_ENC_GET: Returns str: encoding of newly created documents (app option "newdoc_encoding").
* PROC_CONFIG_NEWDOC_ENC_SET: Sets str: encoding of newly created documents.

* PROC_CONFIG_SCALE_GET: Returns UI scales, as 2-tuple (int_scale_UI, int_scale_font). Both values are in percents, defaults are 100.
* PROC_CONFIG_SCALE_SET: Sets UI scales, value must be 2-tuple (int_scale_UI, int_scale_font).

===================

</details>
<details><summary>* * <b>Search properties</b> &nbsp; &nbsp; [Global funcs > app_proc > Search properties]</summary>  

Actions work with props of Find/Replace dialog, and props of low-level search engine. When user runs some search command in the dialog, dialog applies its props to the search engine. So most of the time, props of dialog and search engine are sync'ed. But they aren't sync'ed if user just typed new text in the dialog. They also aren't sync'ed if user runs some search command via Command Palette or hotkey (e.g. "find current word, previous").

* PROC_GET_FINDER_PROP: Returns properties of Find/Replace dialog and search engine. Returns dict with keys:
** "find": string "what to find" of search engine
** "rep": string "replace with" of search engine
** "find_d": string "what to find" of dialog field, None if dialog not inited
** "rep_d": string "replace with" of dialog field, None if dialog not inited
** "find_h": drop-down history list of "what to find" dialog field, None if dialog not inited
** "rep_h": drop-down history list of "replace with" dialog field, None if dialog not inited
** "op_case": option "case sensitive" of search engine
** "op_word": option "whole words" of search engine
** "op_regex": option "reg.ex." of search engine
** "op_cfm": option "confirm on replace" of search engine
** "op_insel": option "in selection" of search engine
** "op_wrap": option "wrapped search" of search engine
** "op_back": option "backward search" (low level) of search engine
** "op_fromcaret": option "search from caret" (low level) of search engine
** "op_tokens": option "allowed syntax elements" of search engine (int)
** "op_case_d": option "case sensitive" of dialog, None if dialog not inited
** "op_word_d": option "whole words" of dialog, None if dialog not inited
** "op_regex_d": option "reg.ex." of dialog, None if dialog not inited
** "op_cfm_d": option "confirm on replace" of dialog, None if dialog not inited
** "op_insel_d": option "in selection" of dialog, None if dialog not inited
** "op_wrap_d": option "wrapped search" of dialog, None if dialog not inited
** "op_mulline_d": option "multi-line input fields" of dialog, None if dialog not inited
** "op_tokens_d": option "allowed syntax elements" of dialog (int), None if dialog not inited

* PROC_SET_FINDER_PROP: Sets properties of Find/Replace dialog and search engine. Param "text" must be dict, with all or some keys listed in PROC_GET_FINDER_PROP. Values types: str (for text fields), bool (for flags), int (for choices).

===================

</details>
<details><summary>* * <b>System</b> &nbsp; &nbsp; [Global funcs > app_proc > System]</summary>  

* PROC_ENUM_FONTS: Returns list of font names, currently installed in OS. Note: only some names are common in all OSes (like Arial, Courier, Courier New, Terminal, maybe more).
* PROC_GET_SYSTEM_PPI: Returns int value of screen pixels-per-inch. Usual value is 96. When OS UI is scaled, it's bigger, e.g. for scale 150% it is round(96*1.5).
* PROC_GET_GUI_HEIGHT: Returns height (pixels) of GUI element for dlg_custom(). Possible values of 'text': 'button', 'label', 'linklabel', 'combo', 'combo_ro', 'edit', 'spinedit', 'check', 'radio', 'checkbutton', 'filter_listbox', 'filter_listview'. Special value 'scrollbar' gets size of OS scrollbar. Returns None for other values.
* PROC_GET_MOUSE_POS: Returns mouse cursor position in screen-related coordinates, as (x, y) tuple.
* PROC_SEND_MESSAGE: For Windows. Sends message to a window by its handle. Param "text" must be 4-tuple of int (window_handle, msg_code, param1, param2).
* PROC_GET_UNIQUE_TAG: Gets integer value, which is unique for the current CudaText process (it starts with 100, increasing with each call). Use this value for Editor.markers() and Editor.attr() tag values.

===================

</details>
<details><summary>* * <b>Clipboard</b> &nbsp; &nbsp; [Global funcs > app_proc > Clipboard]</summary>  

* PROC_GET_CLIP: Returns clipboard text.
* PROC_SET_CLIP: Copies text to clipboard (usual).
* PROC_SET_CLIP_ALT: Copies text to alternate clipboard on Linux, called "primary selection".
** CudaText Copy commands put text to usual clipboard + primary selection.
** CudaText Paste commands get text from usual clipboard.
** CudaText Paste with option "mouse_mid_click_paste":true gets text from primary selection.

===================

</details>
<details><summary>* * <b>Plugin calls</b> &nbsp; &nbsp; [Global funcs > app_proc > Plugin calls]</summary>  

* PROC_SET_EVENTS: Subscribes plugin to events. First it removes all subscribed events for specified module name (even those from install.inf file) and then adds specified events (if event list is not empty). Param "text" must be 4 values ";"-separated: "module_name;event_list;lexer_list;filter".
** module_name is plugin's main module name, usually starting with "cuda_".
** event_list is comma-separated event names, e.g. "on_open,on_save", or empty string.
** lexer_list is comma-separated lexer names, e.g. "C,C++", or empty  string.
** filter is additional parameter, used by some events, if that filter is not empty:
*** filter for "on_key": comma-separated list of int key codes to handle in event, e.g. "9,32" means that event is only called for pressed Tab and Space.
*** filter for "on_open", "on_open_pre": comma-separated list of lower-case file extensions, without leading dot, to handle in event.

* PROC_GET_LAST_PLUGIN: Returns info about last plugin which run from program. It is str "module_name,method_name", values are empty if no plugins run yet.

* PROC_SET_SUBCOMMANDS: Adds command items (items which can be called from Command Palette). These items will run specified module name and method name, but with different parameters to that method. For example, plugin External Tools uses this API to add command items for configured tools. Param "text" must be ";"-separated: "module_name;method_name;param_list".
** "param_list" is "\n"-separated items, each item is param_caption+"\t"+param_value.
** "param_caption" is caption for Command Palette.
** "param_value" is parameter for Python method. It can be any primitive type: int, string, bool, None, or even some expression.

* PROC_GET_COMMANDS: Returns list of all commands, as list of dict. Dict keys are:
** "type": str: Possible values:
*** "cmd": usual built-in command
*** "lexer": command to activate lexer
*** "plugin": plugin command
*** "openedfile": command to switch active tab
*** "recentfile": command to reopen recent file
** "cmd": int: Command code, to use in ed.cmd() call.
** "name": str: Pretty name, which is visible in the Commands dialog.
** "key1": str: Hotkey-1.
** "key2": str: Hotkey-2.
** "key1_init": str: Hotkey-1 from built-in config.
** "key2_init": str: Hotkey-2 from built-in config.
** (for plugins) "p_caption": str: Menu-item caption from install.inf.
** (for plugins) "p_module": str: Python module.
** (for plugins) "p_method": str: Python method in Command class.
** (for plugins) "p_method_params": str: Optional params for Python method.
** (for plugins) "p_lexers": str: Comma-separated lexers list.
** (for plugins) "p_from_api": bool: Shows that command was generated from API by some plugin.
** (for plugins) "p_in_menu": str: Value of parameter "menu" from install.inf.

===================

</details>
<details><summary>* * <b>Hotkeys</b> &nbsp; &nbsp; [Global funcs > app_proc > Hotkeys]</summary>  

* PROC_GET_ESCAPE: Returns Esc-key flag (bool). This flag is set when user presses Esc key, each Esc press is counted since app start. Note: to allow app to handle key press in long loop, call msg_status('text', True).
* PROC_SET_ESCAPE: Sets Esc-key flag. Param "text" must be bool (or "0"/"1").
* PROC_GET_HOTKEY: Returns hotkeys strings for given command_id. Examples of result: "F1", "Ctrl+B * B", "Ctrl+B * B * C|Ctrl+B * D" (two hotkeys for one command). Returns empty str for unknown command_id.
* PROC_SET_HOTKEY: Sets hotkeys for given command_id from strings. Text must be "command_id|hotkey1|hotkey2", where hotkey1/hotkey2 examples: "F1", "Ctrl+B * B", "Ctrl+B * B * C". Symbol "*" can be without near spaces. Returns bool: command_id exists, changed.
* PROC_GET_KEYSTATE: Returns state of pressed keyboard keys and mouse buttons. String result has chars:
** "c" for Ctrl
** "a" for Alt
** "s" for Shift
** "m" for Meta (Windows key)
** "L" for left mouse button
** "R" for right mouse button
** "M" for middle mouse button
* PROC_HOTKEY_STR_TO_INT: Converts given string hotkey to int. Returns 0 for incorrect string. Example: converts "alt+shift+z" to 41050.
* PROC_HOTKEY_INT_TO_STR: Converts given int hotkey to string. Returns empty str for unknown code. Example: converts 41050 to "Shift+Alt+Z".

Notes:

* Value of command_id can be: str(int_command_code) or "module,method" or "module,method,param".

===================

</details>
<details><summary>* * <b>Python</b> &nbsp; &nbsp; [Global funcs > app_proc > Python]</summary>  
* PROC_EXEC_PYTHON: Runs Python string in the context of Console. It is not the same as standard exec() call, it uses the same globals/locals as CudaText Console.
* PROC_EXEC_PLUGIN: Runs Python plugin's method. Text must be comma-separated: "module_name,method_name,params", where "params" is optional part, it is parameter(s) for the method.

===================

</details>
<details><summary>* * <b>Themes</b> &nbsp; &nbsp; [Global funcs > app_proc > Themes]</summary>  

Notes:
* Theme names are short strings, lowercase, e.g. "cobalt", "sub". Empty string means built-in theme.
* Setting default theme (empty str) resets both UI + syntax themes.
* To enumerate themes, you need to list themes folder.

Actions:

* PROC_THEME_UI_GET: Returns name of UI-theme.
* PROC_THEME_UI_SET: Sets name of UI-theme.
* PROC_THEME_SYNTAX_GET: Returns name of syntax-theme.
* PROC_THEME_SYNTAX_SET: Sets name of syntax-theme.

* PROC_THEME_UI_DICT_GET: Returns contents of current UI-theme, as dict.
* PROC_THEME_SYNTAX_DICT_GET: Returns contents of current syntax-theme, as dict. For description of keys of result, see LEXER_GET_STYLES.

===================

</details>
<details><summary>* * <b>Sessions</b> &nbsp; &nbsp; [Global funcs > app_proc > Sessions]</summary>  

Session is a set of opened files + untitled tabs, with some properties of editors in these tabs, with group-indexes of tabs, with layout of groups. Session format is JSON.

* PROC_SAVE_SESSION: Saves session to file with given name. Returns bool: session was saved.
* PROC_LOAD_SESSION: Loads session from file with given name (closes all tabs first). Returns bool: tabs closed w/o pressing Cancel, session was loaded.
* PROC_SET_SESSION: Tells to app filename of session. Session with this filename will be saved on exit, loaded on start, shown in app title in {} brackets. Don't set here empty string. Default filename is "history session.json" without path. Missing path means that folder "settings" will be used.

===================

</details>
<details><summary>* * <b>Side panel</b> &nbsp; &nbsp; [Global funcs > app_proc > Side panel]</summary>  

* PROC_SIDEPANEL_ADD_DIALOG: Adds tab, with embedded dialog, to sidebar. Returns bool: params correct, tab was added. Text must be 3-tuple (tab_caption, id_dlg, icon_filename):
** Caption of tab (must not contain ",")
** Handle of dialog, from dlg_proc()
** Name of icon file (must not contain ",")

* PROC_SIDEPANEL_REMOVE: Removes tab. Text is caption of tab. (Note: actually it hides tab, dialog for this tab is still in memory). Returns bool: tab was found/removed.
* PROC_SIDEPANEL_ACTIVATE: Activates tab in sidebar. Returns bool: tab was found/activated. Text can be:
** str: tab_caption.
** 2-tuple (tab_caption, bool_set_focus). Default for bool_set_focus is False.
* PROC_SIDEPANEL_ENUM: Enumerates tabs. Returns str, "\n"-separated captions of tabs.
* PROC_SIDEPANEL_GET_CONTROL: Returns int handle of dialog, inserted into tab. Text is caption of tab. Returns None if cannot find tab.
* PROC_SIDEPANEL_GET: Returns caption of last activated tab.

Notes:

* Name of icon file: name of png/bmp file. Icon size should be equal to size of current sidebar icon set, it's default is 20x20 (to detect size of icon set, you can read and parse CudaText option). If filename is without path, CudaText sub-dir "data/sideicons" is used (so only a standard icon name can be without path).

===================

</details>
<details><summary>* * <b> Bottom panel</b> &nbsp; &nbsp; [Global funcs > app_proc >  Bottom panel]</summary>  

* PROC_BOTTOMPANEL_ADD_DIALOG: Adds tab. Same as for PROC_SIDEPANEL_ADD_DIALOG.
* PROC_BOTTOMPANEL_REMOVE: Removes tab. Same as for PROC_SIDEPANEL_REMOVE.
* PROC_BOTTOMPANEL_ACTIVATE: Activates tab. Same as for PROC_SIDEPANEL_ACTIVATE.
* PROC_BOTTOMPANEL_ENUM: Enumerates tabs. Same as for PROC_SIDEPANEL_ENUM.
* PROC_BOTTOMPANEL_GET_CONTROL: Returns int handle of control. Same as for PROC_SIDEPANEL_GET_CONTROL.
* PROC_BOTTOMPANEL_GET: Returns caption of last activated tab. Same as for PROC_SIDEPANEL_GET.

===================

</details>
<details><summary>* * <b>Splitters</b> &nbsp; &nbsp; [Global funcs > app_proc > Splitters]</summary>  

Splitter id:

* SPLITTER_SIDE: splitter near side panel.
* SPLITTER_BOTTOM: splitter above bottom panel.
* SPLITTER_G1 .. SPLITTER_G5: splitters between groups.

Actions:

* PROC_SPLITTER_GET: Returns info about splitter, as 4-tuple: (bool_vertical, bool_visible, int_pos, int_parent_panel_size). Param "text" is int splitter id.
* PROC_SPLITTER_SET: Sets splitter pos. Param "text" is 2-tuple (int_splitter_id, int_splitter_pos).

Positions of group splitters (G1, G2, G3, G4, G5) are determined by grouping view, in one view splitter may be horizontal with one parent panel, in other view - vertical with another parent panel. Detailed:

<pre>
2VERT     t G1 t

2HORZ     t
G1
t

3VERT     t G1 t G2 t

3HORZ     t
G1
t
G2
t

1P2VERT    t G3 t
G2
t

1P2HORZ    t
G3
t G2 t

4VERT     t G1 t G2 t G3 t

4HORZ     t
G1
t
G2
t
G3
t

4GRID     t G1 t
G3
t G2 t

6VERT     t G1 t G2 t G3 t G4 t G5 t

6HORZ     t
G1
t
G2
t
G3
t
G4
t
G5
t

6GRID     t G1 t G2 t
G3
t G1 t G2 t
</pre>

===================

</details>
<details><summary>* * <b>Show/Hide/Undock UI elements</b> &nbsp; &nbsp; [Global funcs > app_proc > Show/Hide/Undock UI elements]</summary>  

Actions can get/set state of UI elements (pass "0"/"1" or False/True):

* PROC_SHOW_STATUSBAR_GET
* PROC_SHOW_STATUSBAR_SET
* PROC_SHOW_TOOLBAR_GET
* PROC_SHOW_TOOLBAR_SET
* PROC_SHOW_SIDEBAR_GET
* PROC_SHOW_SIDEBAR_SET
* PROC_SHOW_SIDEPANEL_GET
* PROC_SHOW_SIDEPANEL_SET
* PROC_SHOW_BOTTOMPANEL_GET
* PROC_SHOW_BOTTOMPANEL_SET
* PROC_SHOW_TABS_GET
* PROC_SHOW_TABS_SET
* PROC_SHOW_TREEFILTER_GET
* PROC_SHOW_TREEFILTER_SET
* PROC_SHOW_FLOATGROUP1_GET
* PROC_SHOW_FLOATGROUP1_SET
* PROC_SHOW_FLOATGROUP2_GET
* PROC_SHOW_FLOATGROUP2_SET
* PROC_SHOW_FLOATGROUP3_GET
* PROC_SHOW_FLOATGROUP3_SET

* PROC_FLOAT_SIDE_GET
* PROC_FLOAT_SIDE_SET
* PROC_FLOAT_BOTTOM_GET
* PROC_FLOAT_BOTTOM_SET

===================

</details>

</details>
<details><summary>* <b>app_log</b> &nbsp; &nbsp; [Global funcs > app_log]</summary>  
<details><summary>&lt;descr&gt;</summary>  
app_log(id, text, tag=0, panel="")

Performs action with standard panels in the bottom panel: Output, Validate, Console.

Possible values of "panel":

* value LOG_PANEL_OUTPUT: Output panel
* value LOG_PANEL_VALIDATE: Validate panel

Possible values of "id" for Output/Validate panels:

* LOG_CLEAR: Clears log panel. Param "text" ignored.
* LOG_ADD: Adds line to log panel. Param "tag" is used here: int value associated with line, it's passed in on_output_nav.
* LOG_SET_REGEX: Sets parsing regex for log panel. Param "text". Regex must have some groups in round brackets, indexes of these groups must be passed in separate API calls. All lines in log panel, which can be parsed by this regex, will allow navigation to source code by click or double-click.
* LOG_SET_LINE_ID: Sets index of regex group for line-number. Param "text" is one-char string from "1" to "8", and "0" means "not used".
* LOG_SET_COL_ID: Sets index of regex group for column-number. Param "text".
* LOG_SET_NAME_ID: Sets index of regex group for file-name. Param "text".
* LOG_SET_FILENAME: Sets default file name, which will be used when regex cannot find file name in a string. Param "text".
* LOG_SET_ZEROBASE: Sets flag: line and column numbers are 0-based, not 1-based. Param "text" is one-char string "0" or "1".
* LOG_GET_LINES_LIST: Returns items in panel's listbox, as list of 2-tuples (line, tag).
* LOG_GET_LINEINDEX: Returns index of selected line in panel's listbox.
* LOG_SET_LINEINDEX: Sets index of selected line in panel's listbox.

Possible values of "id" for "Console" panel:

* LOG_CONSOLE_CLEAR: Clears UI controls in console.
** If text empty or has "m" then memo-control (above) is cleared.
** If text empty or has "e" then edit-control (below) is cleared.
** If text empty or has "h" then combobox history list is cleared.
* LOG_CONSOLE_ADD: Adds line to console (its combobox and memo).
* LOG_CONSOLE_GET_COMBO_LINES: Returns list of lines in combobox-control.
* LOG_CONSOLE_GET_MEMO_LINES: Returns list of lines in memo-control.

===================

</details>

</details>
<details><summary>* <b>app_idle</b> &nbsp; &nbsp; [Global funcs > app_idle]</summary>  

app_idle(wait=False)

Performs application's message-processing. If wait=True, also waits for new UI event.

</details>
<details><summary>* <b>emmet</b> &nbsp; &nbsp; [Global funcs > emmet]</summary>  

emmet(id, text, p1="", p2="")

Performs action with embedded Emmet engine. Possible values of "id":

* EMMET_EXPAND: Expand abbreviation. Returns 2-tuple (str_result, bool_multi_caret) or None.
** Param "text": Emmet abbreviation.
** Param "p1": Emmet syntax. Possible values: "html", "xml", "xsl", "css", "svg", "sass", "less".

* EMMET_WRAP: Wrap text with abbreviation. Returns str result or None.
** Param "text": Emmet abbreviation.
** Param "p1": Emmet syntax.
** Param "p2": Text to wrap. Can be multi-line with LF line breaks.

* EMMET_GET_POS: Find beginning of abbreviation in given string. Returns offset in string (0-based), or None.
** Param "text": Text string (can include unneeded text after caret position, can include HTML tags before abbreviation).
** Param "p1": Caret offset (0-based) to search abbreviation to the left of it.

</details>
<details><summary>* <b>msg_box</b> &nbsp; &nbsp; [Global funcs > msg_box]</summary>  

msg_box(text, flags)

Shows modal message-box.

Param "text" is message text. Multi-line text with "\n" is supported.

Param "flags" is sum of button-value (OK, OK/Cancel, Yes/No etc):
* MB_OK
* MB_OKCANCEL
* MB_ABORTRETRYIGNORE
* MB_YESNOCANCEL
* MB_YESNO
* MB_RETRYCANCEL

and icon-value:
* MB_ICONERROR
* MB_ICONQUESTION
* MB_ICONWARNING
* MB_ICONINFO

Returns int code of pressed button (returns ID_CANCEL if dialog cancelled):

* ID_OK
* ID_CANCEL
* ID_ABORT
* ID_RETRY
* ID_IGNORE
* ID_YES
* ID_NO

</details>
<details><summary>* <b>msg_box_ex</b> &nbsp; &nbsp; [Global funcs > msg_box_ex]</summary>  

msg_box_ex(caption, text, buttons, icon, focused=0)

Shows modal message-box with any number of buttons with any captions.

* Param "caption": str: Caption of dialog.
* Param "text": str: Message text. Multi-line text with "\n" is supported.
* Param "buttons": list of str: Non-empty list of button captions. Multi-line captions are not supported on Windows.
* Param "icon": int: Icon in dialog, one of constants:
** MB_ICONERROR
** MB_ICONQUESTION
** MB_ICONWARNING
** MB_ICONINFO
* Param "focused": int: Index of focused button.

Returns int index of pressed button (0-based), or None of cancelled.

</details>
<details><summary>* <b>msg_status</b> &nbsp; &nbsp; [Global funcs > msg_status]</summary>  

msg_status(text, process_messages=False)

Shows given text in statusbar.

Param "process_messages": if True, function also does UI messages processing. It takes some time, it is needed to refresh status of Esc-key pressed. After call msg_status(..., True) you can get state of Esc-key pressed via PROC_GET_ESCAPE, otherwise plugin gets an old state, until UI messages are processed.

</details>
<details><summary>* <b>msg_status_alt</b> &nbsp; &nbsp; [Global funcs > msg_status_alt]</summary>  

msg_status_alt(text, seconds)

Shows given text in the floating tooptip on the top. Multi-line text (with "\n") is supported.

Param "seconds": show pause in seconds, 1..30. Value<=0 hides the tooltip.

</details>
<details><summary>* <b>dlg_input</b> &nbsp; &nbsp; [Global funcs > dlg_input]</summary>  

dlg_input(label, text)

Shows modal dialog to input one string. Returns entered string or None if cancelled.

* Param "label": prompt text above input field.
* Param "text": initial value of input field.

</details>
<details><summary>* <b>dlg_input_ex</b> &nbsp; &nbsp; [Global funcs > dlg_input_ex]</summary>  

dlg_input_ex(number, caption,
label1   , text1="", label2="", text2="", label3="", text3="",
label4="", text4="", label5="", text5="", label6="", text6="",
label7="", text7="", label8="", text8="", label9="", text9="",
label10="", text10="")

Shows modal dialog to enter 1 to 10 string fields at once. Returns list of strings or None if cancelled.

* Param "number": count of input fields.
* Params "label": prompt text above input field.
* Params "text": initial value of input field.

</details>
<details><summary>* <b>dlg_file</b> &nbsp; &nbsp; [Global funcs > dlg_file]</summary>  

dlg_file(is_open, init_filename, init_dir, filters, caption="")

Shows "Open file" or "Save file as" modal dialog. Returns filename (or list of filenames) or None if cancelled.

* Param "is_open": True for "Open" dialog, False for "Save as" dialog.
* Param "init_filename": Initial filename for "Save as" dialog. Can be empty.
* Param "init_dir": Initial folder. Can be empty.
* Param "filters": How to filter filenames from file system. Can be empty. Must be in format used by Lazarus IDE, example for 2 filters: "Source codes|*.pas;*.inc|Text files|*.txt"
* Param "caption": If not empty, dialog caption. If empty, default dialog caption is used: "Open file", "Save file".

Additional options:

* To allow multi-selection in the "Open" dialog, pass init_filename="*". If single filename was selected, result is str. If several filenames were selected, result is list of str.
* To disable checking "entered filename exists" in the "Open" dialog, start init_filename with "!".

</details>
<details><summary>* <b>dlg_dir</b> &nbsp; &nbsp; [Global funcs > dlg_dir]</summary>  

dlg_dir(init_dir, caption="Select folder")

Shows dialog to select folder path. Returns path (str), or None if cancelled.

* Param "init_dir": Initial folder.
* Param "caption": Dialog caption.

</details>
<details><summary>* <b>dlg_menu</b> &nbsp; &nbsp; [Global funcs > dlg_menu]</summary>  

dlg_menu(id, items, focused=0, caption="", clip=0, w=0, h=0)

Shows menu-like dialog. Returns index of selected item (0-based), or None if cancelled.

Possible values of "id":

* MENU_LIST: Dialog with listbox and filter.
* MENU_LIST_ALT: Dialog with listbox and filter, but each item has double height, and instead of right-aligning, 2nd part of an item shows below.

Additional flags to add to "id" value:

* MENU_NO_FUZZY: Disable fuzzy search in the items list.
* MENU_NO_FULLFILTER: Disable filtering by second part of items (after "\t"), filter only by first part.
* MENU_CENTERED: Show menu centered on screen (otherwise it's aligned to current editor).
* MENU_EDITORFONT: Use editor's font (monospaced) for list items, instead of variable-width UI font.

Parameters:

* Param "items": Menu items, can be list of str, tuple of str, or string with "\n"-separated parts. Each item can be simple string or part1+"\t"+part2, where part2 shows right-aligned or below.
* Param "focused": Index of initially selected item.
* Param "caption": If not empty string, it is dialog caption.
* Param "clip": How to shorten too long lines, one of values: CLIP_NONE (default), CLIP_LEFT, CLIP_MIDDLE, CLIP_RIGHT.
* Param "w": If value>0, width of dialog.
* Param "h": If value>0, height of dialog.

</details>
<details><summary>* <b>dlg_color</b> &nbsp; &nbsp; [Global funcs > dlg_color]</summary>  

dlg_color(value)

Shows select-color dialog with given initial color (int).

Returns int color, or None if cancelled.

</details>
<details><summary>* <b>dlg_hotkey</b> &nbsp; &nbsp; [Global funcs > dlg_hotkey]</summary>  

dlg_hotkey(title="")

Shows dialog to press single hotkey.

Returns str of hotkey (e.g. "F1", "Ctrl+Alt+B") or None if cancelled.

</details>
<details><summary>* <b>dlg_hotkeys</b> &nbsp; &nbsp; [Global funcs > dlg_hotkeys]</summary>  

dlg_hotkeys(command, lexer="")

Shows dialog to configure hotkeys of internal command or plugin. Returns bool: OK pressed and hotkeys saved (to keys.json).

Param "command" can be:

* str(int_command): for internal command codes (module cudatext_cmd).
* "module_name,method_name" or "module_name,method_name,method_param": for command plugin.

Param "lexer" is optional lexer name. If not empty, dialog enables checkbox "For current lexer" and, if checkbox checked, saves hotkey to lexer-specific config "keys lexer NNNN.json".

</details>
<details><summary>* <b>dlg_custom</b> &nbsp; &nbsp; [Global funcs > dlg_custom]</summary>  
<details><summary>&lt;descr&gt;</summary>  
dlg_custom(title, size_x, size_y, text, focused=-1, get_dict=False)

Shows dialog with controls of many types.

===================

</details>
<details><summary>* * <b>Types</b> &nbsp; &nbsp; [Global funcs > dlg_custom > Types]</summary>  

Types of UI controls:

* "button": simple button
* "label": simple read-only text
* "check": checkbox, checked/unchecked/grayed
* "radio": radio-button, only one of radio-buttons can be checked
* "edit": single-line input
* "edit_pwd": single-line input for password, text is masked
* "combo": combobox, editable + drop-down, value is text
* "combo_ro": combobox, drop-down only, value is index of drop-down
* "listbox": list of items with one item selected
* "checkbutton": looks like button, but don't close dialog, checked/unchecked
* "memo": multi-line input
* "checkgroup": group of check-boxes
* "radiogroup": group of radio-buttons
* "checklistbox": listbox with checkboxes
* "spinedit": input for numbers, has min-value, max-value, increment
* "listview": list with columns, with column headers, value is index
* "checklistview": listview with checkboxes, value is check-flags
* "linklabel": label which activates URL on click
* "panel": rectangle with centered caption only (client area is entire rect)
* "group": rectangle with OS-themed border and caption on border (client area is decreased by this border)
* "colorpanel": panel, with N-pixels colored border, with colored background
* "image": picture, which shows picture-file
* "trackbar": horiz/vert bar with handler, has position
* "progressbar": horiz/vert bar, only shows position
* "progressbar_ex": like progressbar, with new styles
* "filter_listbox": input, which filters content of another "listbox" control
* "filter_listview": input, which filter content of another "listview" control
* "bevel": control which shows only border (at one side, or all 4 sides), w/o value/caption
* "tabs": TabControl: set of tabs, w/o pages attached to them
* "pages": PageControl: set of tabs, with pages attached to them (only one of pages is visible)
* "splitter": divider bar, which can be dragged by mouse. It finds "linked control", ie control with the same "parent" and "align", nearest to splitter. On mouse drag, splitter resizes this linked control. Recommended to place another control with "align": ALIGN_CLIENT, so splitter will resize two controls at once.

Notes:

* Controls "combo", "combo_ro", "listbox": value is 0-based index of selected item, or -1 if none is selected.
* Control "button_ex": to change advanced props, you must get handle via DLG_CTL_HANDLE, and pass it to button_proc().
* Control "treeview" don't have "items"/"value": to work with it, you must get handle of control via DLG_CTL_HANDLE, and pass it to tree_proc().
* Control "listbox_ex" don't have "items"/"value": to work with it, you must get handle of control via DLG_CTL_HANDLE, and pass it to listbox_proc().
* Control "toolbar" don't have "items"/"value": to work with it, you must get handle via DLG_CTL_HANDLE, and pass it to toolbar_proc().
* Control "statusbar" don't have "items"/"value": to work with it, you must get handle via DLG_CTL_HANDLE, and pass it to statusbar_proc().
* Controls "editor"/"editor_edit"/"editor_combo" don't have "items"/"value": to work with it, you must get handle via DLG_CTL_HANDLE, and pass it to Editor() to make editor object.
* Control "paintbox" is empty area, plugin can paint on it. Get canvas_id via DLG_CTL_HANDLE, and use it in canvas_proc().
* Control property "name" is required for "filter_nnnnn" controls: must set name of listbox/listview, and name of its filter - to the same name with prefix "f_" (e.g. listbox name "mylist" with filter name "f_mylist").

===================

</details>
<details><summary>* * <b> Properties </b> &nbsp; &nbsp; [Global funcs > dlg_custom >  Properties ]</summary>  

Parameter "text" is "\n"-separated items, one item per control. Each item is chr(1)-separated props in the form "key=value". Possible props:

* "type": type of control; must be specified first
* "cap": caption
* "act": active state, bool. For many controls (edit, check, radio, combo_ro, checkbutton, listbox'es, listview's, tabs), it means that control's value change fires events (for dlg_proc) or closes form (for dlg_custom).
* "x", "y": position, left/top
* "w", "h": size, width/height
* "w_min", "w_max": Constraints for width, min/max value.
* "h_min", "h_max": Constraints for height, min/max value.
* "pos": position, str in the form "left,top,right,bottom". Some one-line controls ignore bottom and do auto size. If specified "x/y/w/h" together with "pos", then last mentioned prop has effect.
* "en": enabled state, bool
* "vis": visible state, bool
* "hint": hint string for mouse-over. Can be multiline, "\r"-separated.
* "texthint": text, shown with italic+pale font when value of control is empty. For "edit", "memo", "editor", "editor_edit", "editor_combo". Cannot be multi-line.
* "color": background color
* "autosize": control is auto-sized (by width and/or height, it's control-specific)
* "font_name": font name
* "font_size": font size
* "font_color": font color
* "font_style": font styles. String, each char can be: "b": bold, "i": italic, "u": underline, "s": strikeout.
* "name": optional name of control. It may be not unique for all controls.
* "ex0"..."ex9": advanced control-specific props. Described below.
* "props": deprecated; advanced control-specific props. Described below.
* "val": value of control. Described below.
* "items": list of items. Described below.

===================

</details>
<details><summary>* * <b>Prop "items"</b> &nbsp; &nbsp; [Global funcs > dlg_custom > Prop "items"]</summary>  

Possible values of "items":

* combo, combo_ro, listbox, checkgroup, radiogroup, checklistbox, tabs: "\t"-separated lines
* listview, checklistview: "\t"-separated items.
** first item is column headers: title1+"="+size1 + "\r" + title2+"="+size2 + "\r" +...
** size1...sizeN can be with lead char to specify alignment of column: L (default), R, C
** other items are data: cell1+"\r"+cell2+"\r"+... (count of cells may be less than count of columns)
* image: full path of picture file (png/gif/jpg/bmp)

Action DLG_CTL_PROP_GET also returns key "items" (in the same format) for these controls: "listbox", "checklistbox", "listview", "checklistview".

===================

</details>
<details><summary>* * <b>Prop "columns"</b> &nbsp; &nbsp; [Global funcs > dlg_custom > Prop "columns"]</summary>  

* For "listview", "checklistview": "\t"-separated items, each item is "\r"-separated props of a column:
** caption (must be without "\t", "\r", "\n")
** width, as str
** (optional) minimal width, as str. 0 means "not used". Not supported on Windows.
** (optional) maximal width, as str. 0 means "not used". Not supported on Windows.
** (optional) alignment - one of "L", "R", "C"
** (optional) autosize - "0", "1"
** (optional) visible - "0", "1"

* For "radiogroup", it is number of vertical columns of radiobuttons.

Action DLG_CTL_PROP_GET also returns key "columns" in the same format.

Action DLG_CTL_PROP_SET, for "listview", allows both "items" (it sets columns) and "columns", and "columns" is applied last.

===================

</details>
<details><summary>* * <b>Prop "val"</b> &nbsp; &nbsp; [Global funcs > dlg_custom > Prop "val"]</summary>  

* check: "0", "1" or "?" (grayed state)
* radio, checkbutton: "0", "1"
* edit, edit_pwd, spinedit, combo, filter_*: text
* memo: "\t"-separated lines, in which "\t" chars must be replaced to chr(3)
* combo_ro, listbox, radiogroup, listview: current index
* checkgroup: ","-separated checked states ("0", "1")
* checklistbox, checklistview: index + ";" + checked_states
* tabs, pages: index of active tab
* trackbar, progressbar: int position

===================

</details>
<details><summary>* * <b>Prop "props"</b> &nbsp; &nbsp; [Global funcs > dlg_custom > Prop "props"]</summary>  

Deprecated. Tuple of advanced control-specific properties. See description of prop "ex". If some control supports 4 props, "ex0" to "ex3", then "props" must be 4-tuple for it. If some control supports only "ex0", then "props" must be one value, same as "ex0".

* For dlg_custom, "props" must be str with ","-separated items, and "0"/"1" for bool items.
* For dlg_proc, "props" must be simple type or tuple of simple types, e.g. (True, False, 2).

===================

</details>
<details><summary>* * <b>Prop "ex"</b> &nbsp; &nbsp; [Global funcs > dlg_custom > Prop "ex"]</summary>  

Props "ex0"..."ex9" are control-specific. They have different simple type (str, bool, int...).

* "button":
** "ex0": bool: default for Enter key
* "edit", "memo":
** "ex0": bool: read-only
** "ex1": bool: font is monospaced
** "ex2": bool: show border
* "spinedit":
** "ex0": int: min value
** "ex1": int: max value
** "ex2": int: increment
* "label":
** "ex0": bool: right aligned
* "linklabel":
** "ex0": str: URL. Should not have ",". Clicking on http:/mailto: URLs should work, result of clicking on other kinds depends on OS.
* "listview":
** "ex0": bool: show grid lines
* "radiogroup":
** "ex0": int: 0: arrange items horizontally then vertically (default); 1: opposite
* "tabs", "pages":
** "ex0": int: 0: tab position on top, 1: on bottom, 2: on left, 3: on right
* "colorpanel":
** "ex0": int: border width (from 0)
** "ex1": int: color of fill
** "ex2": int: color of font
** "ex3": int: color of border
* "filter_listview":
** "ex0": bool: filter works for all columns
* "image":
** "ex0": bool: center picture
** "ex1": bool: stretch picture
** "ex2": bool: allow stretch in
** "ex3": bool: allow stretch out
** "ex4": bool: keep origin x, when big picture clipped
** "ex5": bool: keep origin y, when big picture clipped
* "trackbar":
** "ex0": int: orientation (0: horz, 1: vert)
** "ex1": int: min value
** "ex2": int: max value
** "ex3": int: line size
** "ex4": int: page size
** "ex5": bool: reversed
** "ex6": int: tick marks position (0: bottom-right, 1: top-left, 2: both)
** "ex7": int: tick style (0: none, 1: auto, 2: manual)
* "progressbar":
** "ex0": int: orientation (0: horz, 1: vert, 2: right-to-left, 3: top-down)
** "ex1": int: min value
** "ex2": int: max value
** "ex3": bool: smooth bar
** "ex4": int: step
** "ex5": int: style (0: normal, 1: marquee)
** "ex6": bool: show text (only for some OSes)
* "progressbar_ex":
** "ex0": int: style (0: text only, 1: horz bar, 2: vert bar, 3: pie, 4: needle, 5: half-pie)
** "ex1": int: min value
** "ex2": int: max value
** "ex3": bool: show text
** "ex4": int: color of background
** "ex5": int: color of foreground
** "ex6": int: color of border
* "bevel":
** "ex0": int: shape (0: sunken panel, 1: 4 separate lines - use it as border for group of controls, 2: top line, 3: bottom line, 4: left line, 5: right line, 6: no lines, empty space)
* "splitter":
** "ex0": bool: paint border
** "ex1": bool: on mouse drag, use instant repainting (else splitter paints after mouse released)
** "ex2": bool: auto jump to edge, when size of linked control becomes < min size
** "ex3": int: mininal size of linked control (controlled by splitter)

===================

</details>
<details><summary>* * <b>Result</b> &nbsp; &nbsp; [Global funcs > dlg_custom > Result]</summary>  

Dialog is closed by clicking any button or by changing of any control which has "act=1".

* If cancelled, returns None
* If get_dict=True, returns dict: {0: str_value_0, 1: str_value_1, ..., 'clicked': N, 'focused': N}
* If get_dict=False, returns 2-tuple: (clicked_index, state_text)
** "clicked_index": index of control which closed dialog (0-based).
** "state_text": "\n"-separated values of controls. Same count of items as in text, plus optional additional lines in the form "key=value". Line "focused=N" with index of last focused control (0-based) or -1.

===================

</details>

</details>
<details><summary>* <b>dlg_proc</b> &nbsp; &nbsp; [Global funcs > dlg_proc]</summary>  
<details><summary>&lt;descr&gt;</summary>  
dlg_proc(id_dialog, id_action, prop="", index=-1, index2=-1, name="")

Advanced work with dialogs (forms). More advanced than dlg_custom(), forms can show in modal/nonmodal way, controls can change their value during form showing.

Often the function works with some control. To pass some control here, use one of 2 methods:

* Set param "name" to control's name. You give name to a control when you create this control. Param "name" is used if it's not empty.
* Set param "index" to control's index. You get index of newly created control from dlg_proc call. You can find the index from name, using DLG_CTL_FIND action. Param "index" is used if it's >=0 and param "name" is empty.

===================

</details>
<details><summary>* * <b>Types</b> &nbsp; &nbsp; [Global funcs > dlg_proc > Types]</summary>  

Possible types of UI controls are described in dlg_custom, see [[#Types]]. Additional types only for dlg_proc:

* "button_ex": button, application-themed, works via button_proc()
* "editor": full-featured multi-line editor, accessible via Editor API
* "editor_edit": one-line variant of "editor"
* "editor_combo": one-line variant of "editor", with drop-down arrow
* "listbox_ex": listbox, application-themed, works via listbox_proc()
* "paintbox": control which must be painted by plugin via canvas_proc()
* "statusbar": statusbar: panel with one horizontal row of cells, works via statusbar_proc()
* "toolbar": toolbar: panel which holds buttons with icons, works via toolbar_proc()
* "treeview": tree structure with nodes and nested nodes, works via tree_proc()

===================

</details>
<details><summary>* * <b>Form properties</b> &nbsp; &nbsp; [Global funcs > dlg_proc > Form properties]</summary>  

* "cap": str: Caption of form.
* "x", "y": int: Position (screen coordinates), left/top.
* "w", "h": int: Size, width/height.
* "w_min", "w_max": int: Constraints for width, min/max value.
* "h_min", "h_max": int: Constraints for height, min/max value.
* "p": int: Handle of parent form, or 0 if no parent.
* "tag": str: Any string, set by plugin.
* "border": int: Sets border of form, DBORDER_nnn constants. Don't specify it together with deprecated "resize".
** DBORDER_NONE: No visible border, not resizable
** DBORDER_SIZE: Standard resizable border (on Windows: with Minimize/Maximize buttons)
** DBORDER_SINGLE: Single-line border, not resizable (on Windows: form has icon and Minimize button)
** DBORDER_DIALOG: Standard dialog box border, not resizable (on Windows: form has no icon nor Minimize button)
** DBORDER_TOOL: Like DBORDER_SINGLE but with a smaller caption
** DBORDER_TOOLSIZE: Like DBORDER_SIZE but with a smaller caption
* "topmost": bool: Makes form stay on top of other forms in CudaText.
* "vis": bool: Visible state.
* "color": int: Background color.
* "autosize": bool: Form resizes to minimal size, which shows all visible controls. Don't use it together with "resize".
* "keypreview": bool: If on, then key press calls on_key_down before passing key to focused control. Should be True if form needs to handle on_key_down.
* "p": int: Parent handle. Set this property to int handle of any windowed UI control or form, this control/form will be parent of form.
* "focused" (only for reading): index of currently focused control.
* (deprecated) "resize": bool: Border of form has "resizable" style.

===================

</details>
<details><summary>* * <b>Form events</b> &nbsp; &nbsp; [Global funcs > dlg_proc > Form events]</summary>  

These are also properties of forms.

* "on_resize": Called after form is resized.
* "on_close": Called after form is closed.
* "on_close_query": Called to ask plugin, is it allowed to close form (in any way: pressing Esc key, clicking X icon, pressing Alt+F4 or similar hotkey). Plugin must return bool, if True then form will be closed.

* "on_show": Called after form shows.
* "on_hide": Called after form hides.
* "on_act": Called when form is activated.
* "on_deact": Called when form is deactivated.
* "on_mouse_enter": Called when mouse cursor enters form area.
* "on_mouse_exit": Called when mouse cursor leaves form area.

* "on_key_down": Called when key is pressed in form (form should have "keypreview":True). If plugin returns False, key is blocked.
** param "id_ctl": int key code.
** param "data": key-state string: few chars, "c" for Ctrl, "a" for Alt, "s" for Shift, "m" for Meta.

* "on_key_up": Called when key is depressed (after on_key_down). If plugin returns False, depressing of key is blocked.

How to get nice key description in on_key_down, e.g. "Ctrl+Alt+Esc" from id_ctl=27 and data="ca":

<syntaxhighlight lang="python">
str_key =\
('Meta+' if 'm' in data else '')+\
('Ctrl+' if 'c' in data else '')+\
('Alt+' if 'a' in data else '')+\
('Shift+' if 's' in data else '')+\
app_proc(PROC_HOTKEY_INT_TO_STR, id_ctl)
</syntaxhighlight>

===================

</details>
<details><summary>* * <b>Control properties</b> &nbsp; &nbsp; [Global funcs > dlg_proc > Control properties]</summary>  

* "name": str: Optional name of control, to find control later by name. May be not unique for controls.
* "cap": str: Caption.
* "act": bool: Active state. For many controls (edit, check, radio, combo_ro, checkbutton, listbox'es, listview's, tabs), it means that control's value change fires events (for dlg_proc) or closes form (for dlg_custom).
* "x", "y": int: Position (coordinates relative to dialog), left/top.
* "w", "h": int: Size, width/height.
* "en": bool: Enabled state.
* "vis": bool: Visible state.
* "color": int: Color.
* "border": bool: Control has border.
* "font_name": str: Font name.
* "font_size": int: Font size.
* "font_color": int: Font color.
* "hint": str: Hint (tooltip) for mouse-over. Newlines must be "\r".
* "ex0"..."ex9": Advanced control-specific props. Described in dlg_custom.
* "props": Deprecated. Advanced control-specific props. Described in dlg_custom.
* "items": str: Usually tab-separated items. Described in dlg_custom.
* "val": str: Value of control. Described in dlg_custom.
* "tag": str: Any string, set by plugin.

* "focused": bool: Shows if control has focus.
* "tab_stop": bool: Allows tab-key to jump to this control.
* "tab_order": int: Tab-key jumps to controls using tab_orders. First activated is control with tab_order=0, next with =1, etc. If tab_orders not set, controls activated by creation order.

* "sp_l", "sp_r", "sp_t", "sp_b", "sp_a": int: Border spacing, ie padding of control's edge from anchored controls (or parent form). 5 props here: left, right, top, bottom, around. "Around" padding is added to padding of all 4 sides.

* "a_l", "a_r", "a_t", "a_b": 2-tuple (str_control_name, str_side): Anchors of control. See [[#Anchors]]. Value is 2-tuple, or None to disable anchor.
** Item-0: name of target control, or empty str to use control's parent (it is form by default).
** Item-1: side of target control, one of 3 values: "[" (left/top), "-" (center), "]" (right/bottom).

* "align": alignment of control:
** ALIGN_NONE: no alignment (props "x", "y", "w", "h" have meaning)
** ALIGN_CLIENT: stretched to entire parent area (props "x", "y", "w", "h" are ignored)
** ALIGN_LEFT: glued to the left side of parent (only prop "w" has meaning)
** ALIGN_RIGHT: glued to the right side of parent (only prop "w" has meaning)
** ALIGN_TOP: glued to the top of parent (only prop "h" has meaning)
** ALIGN_BOTTOM: glued to the bottom of parent (only prop "h" has meaning)

* "p": str: Name of control's parent control, or empty if the form is used as a parent. Coordinates x/y of a control are relative to the current parent, so control is attached to its parent. For example, you can place several controls on a "panel" control (panel can have hidden border), then change this panel's coordinates to visually move all child controls. Special case: to place control on PageControl's page with index N, specify such parent: pages_name+"."+str(N).

Special properties for control "listview":

* "imagelist_small": int: handle of ImageList with small icons. This ImageList is used in usual "report" mode of listview.
* "imagelist_large": int: handle of ImageList with large icons. This ImageList is used in "big icons" mode of listview.
* "imageindexes": str: string with '\t'-separated numbers. Each number is icon index, or -1 if icon is not used.

Special properties for controls "tabs", "pages":

* "tab_hovered": int: index of tab under mouse cursor.

===================

</details>
<details><summary>* * <b>Control events</b> &nbsp; &nbsp; [Global funcs > dlg_proc > Control events]</summary>  

Events of controls are also properties, but values of these properties must be Python functions (ie callables). Signature of these functions are shown below.

General events for all controls:

* "on_change": Called after "value" of control is changed.
* "on_click": Called after mouse click on control, for non-active controls, which don't change "value" by click. Param "data" is tuple (x, y) with control-related coordinates of click.
* "on_click_dbl": Called after mouse double-click on control. Param "data" is tuple (x, y) with control-related coordinates.
* "on_mouse_down": Called when mouse button pressed. Param "data" is dict: { "btn": int_button_code, "state": str_keyboard_state, "x": int, "y": int }. Button code: 0: left; 1: right; 2: middle. Keyboard state: value like in global event "on_key".
* "on_mouse_up": Called when mouse button depressed (released). Param "data": like in "on_mouse_down".
* "on_mouse_enter": Called when mouse cursor enters control area.
* "on_mouse_exit": Called when mouse cursor leaves control area.
* "on_focus_enter": Called when control gets focus. Not supported for controls (they don't have OS window handle): label, button, button_ex, panel, linklabel, colorpanel, bevel, image, progressbar_ex, paintbox, statusbar.
* "on_focus_exit": Called when control looses focus. Supported for the same controls as "on_focus_enter".
* "on_menu": Called before showing context menu, after right click or ContextMenu hotkey. Param "data": like in "on_mouse_down". Method can return False to disable default menu (if any).

Special events for controls "listview", "checklistview":

* "on_click_header": Called when user clicks on column header. Param "data": int column index.
* "on_select": Called after selection is changed. Param "data" is tuple: (int_item_index, bool_item_selected).

Special events for control "listbox_ex":

* "on_click_x": Called when listbox has X marks shown and X mark is clicked.
* "on_draw_item": Called if listbox is owner-drawn. Param "data" is dict: { "canvas": canvas_id, "index": int_item_index, "rect": item_rectangle_4_tuple }.

Special events for control "treeview":

* "on_fold", "on_unfold": Called before treeview node is folded/unfolded. Param "data" is int handle of treeview node.
* "on_select": Called after selection is changed.

Special events for control "editor"/"editor_edit"/"editor_combo":

* "on_change": Called after text is changed.
* "on_caret": Called after caret position and/or selection is changed.
* "on_scroll": Called after editor is scrolled.
* "on_key_down": Called when user presses a key. Param "data" is tuple (int_key_code, str_key_state). Method can return False to disable default processing.
* "on_key_up": Called when user depresses a key. Param "data" is tuple (int_key_code, str_key_state). Method can return False to disable default processing.
* "on_click_gutter": Called on mouse click on gutter area. Param "data" is dict: {"state": str_key_state, "line": int_line_index, "band": int_gutterband_index}.
* "on_click_gap": Called on mouse click on inter-line gap. Param "data" is dict: {"state": str_key_state, "line": int, "tag": int, "gap_w": int, "gap_h": int, "x": int, "y": int}.
* "on_click_link": Called on mouse click (single or double, depends on settings) over hyperlink (hyperlinks are defined via PROP_LINKS_SHOW, PROP_LINKS_REGEX). Param "data" is str.
* "on_paste": Called before doing one of Paste commands. Param "data" is dict: {"keep_caret": bool, "sel_then": bool}. Method can return False to disable default operation.

===================

</details>
<details><summary>* * <b>Control events - signatures</b> &nbsp; &nbsp; [Global funcs > dlg_proc > Control events - signatures]</summary>  

Callbacks must be in forms shown in the topic [[#Callback_param]].

Callbacks for "dlg_proc" must be declared as:

<syntaxhighlight lang="python">
# for functions
def my(id_dlg, id_ctl, data='', info=''):
pass

# for class methods
class Command:
def my(self, id_dlg, id_ctl, data='', info=''):
pass
</syntaxhighlight>

Parameters:

* "id_dlg": Int handle of the form.
* "id_ctl": Int index of control. But for event "on_key_down" it has different meaning.
* "data": Value specific to event.
* "info": Additional value passed from CudaText, if extended form of callback is specified, with "info" part.

===================

</details>
<details><summary>* * <b>Actions</b> &nbsp; &nbsp; [Global funcs > dlg_proc > Actions]</summary>  

Param "prop": it can be of any simple type (str, int, bool), also tuple/list (of any simple type), also dict (keys: str, values: simple type or tuple/list). Most used is dict. Example: prop={"cap": "...", "x": 10, "y": 10, "w": 600, "en": False}.

Param "id_dialog": int, form handle. Ignored only for DLG_CREATE action (pass 0 with it).

Possible values of "id_action":

* DLG_CREATE: Creates new empty form, returns form "handle". You must pass this handle (int) to next calls of dlg_proc().
* DLG_HIDE: Hides form.
* DLG_FREE: Hides and deletes form.
* DLG_SHOW_MODAL: Shows form in modal mode. Waits for form to hide, then returns.
* DLG_SHOW_NONMODAL: Shows form in non-modal mode. Returns immediately.
* DLG_FOCUS: Focuses form (in non-modal mode).
* DLG_SCALE: Scales form, with all controls, for the current OS high-DPI value. E.g. of OS scale is 150%, all will be scaled by 1.5.

* DLG_PROP_GET: Returns form props, as dict. See example plugin, which props are returned.
* DLG_PROP_SET: Sets form props, from dict. Param "prop" is dict. Only props mentioned in "prop" are applied, other props don't change.

* DLG_DOCK: Docks (inserts) form into another form. Param "index" is handle of another form, and 0 means main CudaText form. Param "prop" can be: "L", "R", "T", "B" for sides left/right/top/bottom (default is bottom).
* DLG_UNDOCK: Undocks form from it's current parent form.

* DLG_LOCK: Disables updating of the form. This can be used before adding many controls at once, to remove flickering of these controls.
* DLG_UNLOCK: Enables updating of the form. This must be used in pair with DLG_LOCK.

* DLG_CTL_COUNT: Returns count of controls on form.
* DLG_CTL_ADD: Adds new control to form, returns its index, or None if cannot add. Param "prop" is type of control. See description in dlg_custom.
* DLG_CTL_PROP_GET: Returns control props, as dict. Control must be specified by name or index.
* DLG_CTL_PROP_SET: Sets control props.  Control must be specified by name or index. Param "prop" is dict with props. Only props mentioned in "prop" are applied, other props don't change. To "reset" some props, you must mention them with some value.
* DLG_CTL_FOCUS: Focuses control.  Control must be specified by name or index.
* DLG_CTL_DELETE: Deletes control.  Control must be specified by name or index. Controls are stored in list, so after a control deleted, indexes of next controls shift by -1. So don't use fixed indexes if you delete some, use DLG_CTL_FIND.
* DLG_CTL_DELETE_ALL: Deletes all controls.
* DLG_CTL_FIND: Returns index of control by name, or -1 if cannot find. Param "prop" is name.
* DLG_CTL_HANDLE: Returns int "handle" of control on a form. Control must be specified by name or index. This handle is currently useful for types:
** type "button_ex": pass handle to button_proc()
** types "editor"/"editor_edit"/"editor_combo": pass handle to Editor() constructor
** type "listbox_ex": pass handle to listbox_proc()
** type "paintbox": pass handle to canvas_proc()
** type "statusbar": pass handle to statusbar_proc()
** type "treeview": pass handle to tree_proc()
** type "toolbar": pass handle to toolbar_proc()

* DLG_COORD_LOCAL_TO_SCREEN: Converts x/y coordinates from form-related, to screen-related. Param "index" is x, "index2" is y. Returns tuple (x,y).
* DLG_COORD_SCREEN_TO_LOCAL: Converts x/y coordinates from screen-related, to form-related. Param "index" is x, "index2" is y. Returns tuple (x,y).

* DLG_POS_GET_STR: Returns form position/size as string.
* DLG_POS_SET_FROM_STR: Sets form position/size from string. Param "prop" is string value.
** It is faster than setting form properties "x", "y", "w", "h", it makes single resize.
** String format is the same as in DLG_POS_GET_STR, comma-separated numbers "x,y,w,h", any number can be skipped to keep old value.

===================

</details>
<details><summary>* * <b>Anchors</b> &nbsp; &nbsp; [Global funcs > dlg_proc > Anchors]</summary>  

Anchor is attaching of control's side to another control, or to the parent form, so control is auto positioned, initially and on form resize. Lazarus IDE has such Anchor Editor dialog:

[[Image:Anchor_Editor_en.png]]

In this dialog you see, that all 4 sides of control attach to one of 3 sides of another control (or parent form).

* Anchors override absolute positions, e.g. anchor of left side overrides prop "x".
* Anchoring to invisible control is allowed.
* Anchoring circles (A to B to C to A) is not allowed, but should not give errors.

To change anchors of control, set its properties: a_l, a_r, a_t, a_b.
Initially left/top anchors are set (to the parent form).

Side value "[" aligns control to left/top side of target:

+--------+
| target |
+--------+

+--------------+
|   control    |
+--------------+

Side value "]" aligns control to right/bottom side of target:

+--------+
| target |
+--------+

+--------------+
|   control    |
+--------------+

Side value "-" centers control relative to target:

+--------+
| target |
+--------+

+--------------+
|   control    |
+--------------+

Example: to attach "colorpanel" to the right side of form, clear left anchor (set to None), and add right/bottom anchors. This also sets spacing-aroung (padding) to 6 pixels.

<syntaxhighlight lang="python">
#attach colorpanel to the right
dlg_proc(id_form, DLG_CTL_PROP_SET, index=n, prop=
{ 'a_l': None, 'a_r': ('', ']'), 'a_b': ('', ']'), 'sp_a': 6  } )
</syntaxhighlight>

===================

</details>

</details>
<details><summary>* <b>dlg_commands</b> &nbsp; &nbsp; [Global funcs > dlg_commands]</summary>  

dlg_commands(options, title="", w=0, h=0)

Show commands dialog, which is customizable version of Commands (F1) dialog in CudaText.

Param "options" should be 0 or sum of int flags:

* COMMANDS_USUAL: Show usual commands, which have int codes. Function returns "c:"+str(int_command) for them.
* COMMANDS_PLUGINS: Show plugins commands. Function returns "p:"+callback_string for them.
* COMMANDS_LEXERS: Show lexers pseudo-commands. Function returns "l:"+lexer_name for them.
* COMMANDS_FILES: Show opened files pseudo-commands. Function returns "f:"+file_name for them.
* COMMANDS_RECENTS: Show recent files pseudo-commands. Function returns "r:"+file_name for them.
* COMMANDS_CONFIG: Allow to call Configure Hotkeys dialog by F9 key.
* COMMANDS_CONFIG_LEXER: Enable checkbox "For current lexer" when dialog to configure hotkey is called.
* COMMANDS_CENTERED: Set dialog position to the center of desktop. Otherwise, position depends on CudaText option.

Params "w" and "h": if values>0, they are width and height of dialog.

Returns string if command selected, or None if cancelled.

</details>
<details><summary>* <b>file_open</b> &nbsp; &nbsp; [Global funcs > file_open]</summary>  

file_open(filename, group=-1, options="")
file_open((filename1, filename2), group=-1, options="")

Opens editor tab with given filename. If filename already opened, activates its tab. Pass empty string to open untitled tab.

First parameter can be 2-tuple (or 2-list) of string, to open two files in a single tab, in a splitted view. In this case, second filename won't be handled specially for zip file, picture file, binary file etc.

Returns bool. True, if first filename is empty. Otherwise, returns True if first filename exists, and second filename is empty or exists.
For zip file (only first filename is checked for it), returns True only if zip file contains valid CudaText add-on, and it was installed.

* Param "group": index of tab-group (0-based), default means "current group". If you pass index of currently hidden group, group won't show, you need to call editor command to show it, see [[#cmd]].
* Param "options": string:
** If it has "/preview", then file opens in a "temporary preview" tab, with italic caption. Param "group" is ignored then, used 1st group.
** If it has "/nohistory", file's saved history (caret, scroll pos, etc) won't be used.
** If it has "/nolexerdetect", lexer won't be auto-detected.
** If it has "/noevent", then "on_open_pre" event won't fire.
** If it has "/noopenedevent", then "on_open" event won't fire.
** If it has "/nononeevent", then "on_open_none" event won't fire.
** If it has "/silent", then zipped add-on will install w/o prompt and report.
** If it has "/passive", then tab will not activate, it will be passive tab.
** If it has "/nonear", then app option "ui_tab_new_near_current" will be ignored, tab will append to end.
** If it has "/nozip", then .zip files will not be handled specially.
** If it has "/nopictures", then picture files will not be handled specially.
** If it has "/view-text', then file will open in internal viewer, text (variable width) mode.
** If it has "/view-binary', then file will open in internal viewer, binary (fixed width) mode.
** If it has "/view-hex', then file will open in internal viewer, hex mode.
** If it has "/view-unicode', then file will open in internal viewer, unicode (variable width) mode.
** If it has "/nontext-view-text", then in the case of non-text file, file will open in internal viewer, text (variable width) mode.
** If it has "/nontext-view-binary", then in the case of non-text file, file will open in internal viewer, binary (fixed width) mode.
** If it has "/nontext-view-hex", then in the case of non-text file, file will open in internal viewer, hex mode.
** If it has "/nontext-view-unicode", then in the case of non-text file, file will open in internal viewer, unicode mode.
** If it has "/nontext-view-uhex", then in the case of non-text file, file will open in internal viewer, unicode+hex mode.
** If it has "/nontext-cancel", then in the case of non-text file, function will fail and return False.

Note: "ed" is always the current editor, after file_open() current editor changes, and "ed" is the new cur editor.

Example opens untitled tab, and writes multi-line text to it:
<syntaxhighlight lang="python">
file_open('')
ed.set_text_all(text)
</syntaxhighlight>

</details>
<details><summary>* <b>ed_handles</b> &nbsp; &nbsp; [Global funcs > ed_handles]</summary>  

ed_handles()

Returns range object: it contains int handles of all editor tabs. Pass each handle to Editor() to make editor object from handle.

Example code, which shows filenames of all tabs:

<syntaxhighlight lang="python">
#show all file names in console
for h in ed_handles():
e = Editor(h)
print(e.get_filename())
</syntaxhighlight>

</details>
<details><summary>* <b>ed_group</b> &nbsp; &nbsp; [Global funcs > ed_group]</summary>  

ed_group(index)

Returns Editor object for active editor in tab-group with given group-index. Returns None for incorrect index, or if no tabs in this group.

Index possible values: 0..5 (first 6 groups) and 6..8 (3 floating groups).

</details>
<details><summary>* <b>ini_read/ini_write</b> &nbsp; &nbsp; [Global funcs > ini_read/ini_write]</summary>  

ini_read(filename, section, key, value)
ini_write(filename, section, key, value)

Reads/writes single string from/to .ini file. Params:

* "filename": str: Path of file. Can be without path, this means that path of "settings" dir is used.
* "section": str: Section of ini file.
* "key": str: Key in section.
* "value": str:
** on read: default value which is returned if no such filename/section/key was found.
** on write: value to write.

On read: returns string value. On write: returns None.

</details>
<details><summary>* <b>ini_proc</b> &nbsp; &nbsp; [Global funcs > ini_proc]</summary>  

ini_proc(id, filename, section="", key="")

Performs action on .ini file. Filename can be without path, this means that path of "settings" dir is used.

Possible values of "id":

* INI_GET_SECTIONS: Returns list of sections. Params "section", "key" ignored.
* INI_GET_KEYS: Returns list of keys in given section. Param "key" ignored.
* INI_DELETE_KEY: Deletes given key in given section.
* INI_DELETE_SECTION: Deletes given section with all its keys. Param "key" ignored.

</details>
<details><summary>* <b>lexer_proc</b> &nbsp; &nbsp; [Global funcs > lexer_proc]</summary>  

lexer_proc(id, value)

Perform some lexer-related action.

Possible values of "id":

* LEXER_GET_LEXERS: Returns list of lexers. Param "value" must be bool: allow to include also hidden lexers (not visible in the lexers menu).
* LEXER_GET_PROP: For lexer name (param "value"), returns lexer general properties, as dict. Supported for "lite" lexers too (lexer name must end with ^ suffix). For incorrect lexer name, returns None. Keys of dict:
** "en": bool: lexer is visible in the lexers menu.
** "typ": list of str: list of file-types (they detect lexer when file loads; "ext" is simple extension, "ext1.ext2" is double extension, "/fullname" is name w/o path).
** "st": list of str: list of all styles.
** "st_c": list of str: list of styles of syntax comments (e.g. used by Spell Checker).
** "st_s": list of str: list of styles of syntax strings (e.g. used by Spell Checker).
** "sub": list of str: list of sub-lexers (some items can be empty if lexer setup broken).
** "kinds": list of str: list of token kind names. (This list has strings which lexer author had assigned in lexer properties, it has nothing common with "syntax elements" search.)
** "c_line": str or None: line comment (until end-of-line).
** "c_str": 2-tuple or None: stream comment (for any range).
** "c_lined": 2-tuple or None: comment for full lines.

* LEXER_GET_STYLES: For lexer name (param "value"), returns lexer styles properties, as dict. This works even if CudaText lexer themes are disabled (option "ui_lexer_themes": false). Not supported for "lite" lexers. Dict keys:
** "type": Kind of style. Enum. 0: reserved value, 1: colors and font styles, 2: colors only, 3: background color only.
** "color_font": RGB color of font.
** "color_back": RGB color of background. Can be COLOR_NONE if not used.
** "color_border": RGB color of border.
** "border_left", "border_right", "border_top", "border_bottom": Kind of border at one side. Enum. Values from 0: none, solid, dash, dot, dash dot, dash dot dot, solid2, solid3, wave, double.
** "styles": Font styles, as string. If empty, no styles. If "b" in value, bold. If "i" in value, italic. If "u" in value, underline. If "s" in value, strikeout.
** "tkind": Kind of syntax element. "c" - comment, "s" - string, "a" - any other.

* LEXER_DETECT: Detects lexer name by file name (param "value" is file name). Returns None if cannot detect. Returns string or tuple of string (for example, "C" and "C++" for "test.h"). Function tests file extension, or even filename before extension (e.g. "/path/makefile.gcc" gives "Makefile").
* LEXER_REREAD_LIB: Re-reads lexer library from disk, updates lexer menu. Make sure that plugins' dialogs don't use editor with lexer, which may crash.

* LEXER_ADD_VIRTUAL: Adds virtual lexer, which doesn't have a file and doesn't highlight text, it's only an item in lexers list. The purpose of virtual lexers is to avoid creating lexer files, but allow plugins to be activated by some lexer name / some file types. Param "value" must be tuple of string: (lexer_name, file_types, line_comment, range_comment_begin, range_comment_end). Item file_types here must have the same notation as for "lite" lexers, e.g. "*.pas;*.pp;*.lpr". Virtual lexers are added to the "lite" lexers list, and show the same suffix in lexer menu. To activate such a lexer, call Editor.set_prop(PROP_LEXER_FILE, lexer_name+suffix). Action returns bool: lexer_name didn't exist, lexer was added.

</details>
<details><summary>* <b>tree_proc</b> &nbsp; &nbsp; [Global funcs > tree_proc]</summary>  

tree_proc(id_tree, id_action, id_item=0, index=0, text="", image_index=-1, data="")

Perform action on treeview UI-control.

* Param "id_tree": int handle of treeview.
* Param "id_item": int handle of tree-item. Can be 0 for invisible root-item:
** can clear entire tree using root-item
** can enumerate root level using root-item.

Possible values of "id_action":

* TREE_ITEM_ENUM: Enumerates subitems of given item. Params: "id_item". Returns list of 2-tuples, or None.
** tuple item 0: int: handle of item.
** tuple item 1: string: caption of item.

* TREE_ITEM_ENUM_EX: Enumerates subitems of given item. Params: "id_item". Returns list of dict, or None. Dict keys are:
** "id": int: handle of item.
** "text": str: caption of item
** "data": str: data string of item.
** "img": int: icon index of item, or -1 if icon not used.
** "sub_items": bool: item has the sub-items.

* TREE_ITEM_ADD: Adds subitem as item's child. Returns int handle of subitem.
** Param "id_item": handle of item.
** Param "index": at which subitem index to insert (0-based), or -1 to append.
** Param "text": caption of item.
** Param "image_index": index in tree's icon list or -1 to not show icon.
** Param "data": optional data string attached to item.
* TREE_ITEM_DELETE: Deletes item (with all subitems). Params: "id_item".
* TREE_ITEM_SET_TEXT: Sets item's text. Params: "id_item", "text".
* TREE_ITEM_SET_ICON: Sets item's icon. Params: "id_item", "image_index".

* TREE_ITEM_SELECT: Selects item.
* TREE_ITEM_SHOW: Makes item visible, ie scrolls control to this item.
* TREE_ITEM_GET_SELECTED: Returns int handle of selected item (param "id_item" is ignored). Returns None if nothing is selected.

* TREE_ITEM_GET_PROPS: Returns properties of item, as dict. Dict keys are:
** "text": str: caption of item
** "data": str: data string of item
** "icon": int: index of icon in tree's imagelist object, or -1 for none
** "level": int: how deep this item is nested (how many parents this item has)
** "parent": int: id of parent item, or 0 if no parent
** "folded": bool: is this item folded (item itself, not parents)
** "selected": bool: is this item selected
** "sub_items": bool: item has sub-items
** "index": int: index of item, relative to its branch
** "index_abs": int: absolute index of item, relative to root

* TREE_ITEM_FOLD: Folds item w/o subitems.
* TREE_ITEM_FOLD_DEEP: Folds item with subitems. Root-item allowed too.
* TREE_ITEM_FOLD_LEVEL: Folds all items (id_item ignored) from level with specified index, 1 or bigger. (This is what CudaText commands "fold level N" do for code-tree).
* TREE_ITEM_UNFOLD: Unfolds item w/o subitems.
* TREE_ITEM_UNFOLD_DEEP: Unfolds item with subitems. Root-item allowed too.
* TREE_GET_IMAGELIST: Returns int handle of image-list object.
* TREE_PROP_SHOW_ROOT: This allows to hide lines for invisible root-item. If hidden, tree (folded) looks like listbox. Param text: "0"/"1" to hide/show.
* TREE_LOCK: Disables repainting of control.
* TREE_UNLOCK: Enables repainting of control.
* TREE_THEME: Applies current color theme to control.

* TREE_ITEM_GET_RANGE: Should be used only for code-tree. Returns range, stored in tree-item, as 4-tuple (start_x, start_y, end_x, end_y). If range is not set, returns (-1,-1,-1,-1).
* TREE_ITEM_SET_RANGE: Should be used only for code-tree. Sets range for tree-item. Param "text" must be 4-tuple of int (start_x, start_y, end_x, end_y).

</details>
<details><summary>* <b>listbox_proc</b> &nbsp; &nbsp; [Global funcs > listbox_proc]</summary>  

listbox_proc(id_listbox, id_action, index=0, text="", tag=0)

Perform action on listbox UI-control.

* Param id_listbox: int handle of listbox.
* Param index: index of item (0-base).

Possible values of "id_action":

* LISTBOX_GET_COUNT: Returns number if items.
* LISTBOX_ADD: Adds item with given text and tag. Returns new count of items. Param "index": index of new item, or -1 to append. Param "text": text of item. Param "tag": int tag.
* LISTBOX_ADD_PROP: Adds item with given text and other properties. Returns new count of items. Param "index": index of new item, or -1 to append. Param "text": text of item. Param "tag": dict with keys "tag", "modified", "datatext".
* LISTBOX_DELETE: Deletes item with given index.
* LISTBOX_DELETE_ALL: Deletes all items.
* LISTBOX_GET_ITEM: Returns text/tag of item with given index. Returns 2-tuple (text, tag) or None if index incorrect.
* LISTBOX_SET_ITEM: Sets text/tag of item with given index. Params used: "index", "text", "tag".
* LISTBOX_GET_ITEM_PROP: Returns properties of item with given index, as dict or None. Dict keys are: "text", "tag", "modified", "datatext".
* LISTBOX_SET_ITEM_PROP: Sets properties of item with given index. Param "index": int value. Param "text": new text of item. Param "tag": dict with keys "tag", "modified", "datatext".
* LISTBOX_GET_ITEM_H: Returns height of items in pixels.
* LISTBOX_SET_ITEM_H: Sets height of items in pixels. Param "index": int value
* LISTBOX_GET_SEL: Returns selected index. -1 for none.
* LISTBOX_SET_SEL: Sets selected index. Param "index": int value
* LISTBOX_GET_TOP: Returns index of top visible item.
* LISTBOX_SET_TOP: Sets index of top visible item. Param "index": int value
* LISTBOX_GET_DRAWN: Returns owner-drawn state (bool). Owner-drawn state means that listbox doesn't paint itself, but plugin must paint it via event "on_draw_item".
* LISTBOX_SET_DRAWN: Sets owner-drawn state. Param "index": 0 or 1 (off/on).
* LISTBOX_GET_SHOW_X: Returns state of X marks. Int value: 0: don't show, 1: show for all items, 2: show for mouse-over item.
* LISTBOX_SET_SHOW_X: Sets state of X marks. Param "index": int value.
* LISTBOX_GET_HOTTRACK: Returns HotTrack state (bool). HotTrack means that listbox highlights item under mouse cursor.
* LISTBOX_SET_HOTTRACK: Sets HotTrack state. Param "index": 0 or 1 (off/on).
* LISTBOX_GET_SCROLLSTYLE_HORZ: Returns style of horizontal scrollbar. One of SCROLLSTYLE_ constants.
* LISTBOX_SET_SCROLLSTYLE_HORZ: Sets style of horizontal scrollbar. Param "index": int value, one of SCROLLSTYLE_ constants.
* LISTBOX_GET_SCROLLSTYLE_VERT: Returns style of vertical scrollbar. One of SCROLLSTYLE_ constants.
* LISTBOX_SET_SCROLLSTYLE_VERT: Sets style of vertical scrollbar. Param "index": int value, one of SCROLLSTYLE_ constants.
* LISTBOX_GET_SCROLLPOS_HORZ: Returns horizontal scroll position.
* LISTBOX_SET_SCROLLPOS_HORZ: Sets horizontal scroll position. Param "index": int value.

Listbox can have columns. It works only if listbox is not owner-drawn, and column sizes were set via LISTBOX_SET_COLUMNS. You must add separator-chars inside items to split items into columns. Column sizes are passed as list of int. Each list item can be:
* value>0: Column width in pixels.
* value<0: Column width in percents of listbox width.
* value=0: Column is auto-stretched to fill the entire width. Several auto-stretched columns are allowed.

Actions for columns:

* LISTBOX_GET_COLUMN_SEP: Returns column separator char.
* LISTBOX_SET_COLUMN_SEP: Sets column separator char. Param "text" must be single-char string.
* LISTBOX_GET_COLUMNS: Returns list of column sizes.
* LISTBOX_SET_COLUMNS: Sets list of column sizes. Param "text" must be list of int.

Example adds 3 columns:
<syntaxhighlight lang="python">
listbox_proc(h_list, LISTBOX_SET_COLUMN_SEP, text='|')
listbox_proc(h_list, LISTBOX_SET_COLUMNS, text=[-50,0,-20]) # width<0 means value in %
listbox_proc(h_list, LISTBOX_ADD, index=-1, text='first|second|third')
</syntaxhighlight>

</details>
<details><summary>* <b>canvas_proc</b> &nbsp; &nbsp; [Global funcs > canvas_proc]</summary>  

canvas_proc(id_canvas, id_action,
text="", color=-1, size=-1,
x=-1, y=-1, x2=-1, y2=-1,
style=-1, p1=-1, p2=-1)

Performs action on canvas (drawing surface of some GUI control).
Id_canvas is handle of canvas of some GUI control. Special value 0 means testing empty panel, it appears at the top of app, when used.

Possible values of "id_action":

* CANVAS_SET_FONT: Sets props of font. Params:
** "text": font name
** "color"
** "size"
** "style": 0 for normal, or sum of values FONT_B (bold), FONT_I (italic), FONT_U (underline), FONT_S (strikeout)

* CANVAS_SET_PEN: Sets props of pen. Params:
** "color"
** "size"
** "style": one of PEN_STYLE_nnnn
** "p1": end caps style - one of PEN_CAPS_nnnn
** "p2": line joining style - one of PEN_JOIN_nnnn

* CANVAS_SET_BRUSH: Sets props of brush. Params:
** "color"
** "style": one of BRUSH_nnnn. Usually used: BRUSH_SOLID (filled background), BRUSH_CLEAR (transparent background).

* CANVAS_SET_ANTIALIAS: Sets anti-aliasing mode of canvas. Params: style - ANTIALIAS_NONE, _ON, _OFF.
* CANVAS_GET_TEXT_SIZE: Returns size of text on canvas, as 2-tuple (size_x, size_y). Uses font. Params: text.
* CANVAS_TEXT: Paints text at given coords. Uses font and brush. Params: text, x, y.
* CANVAS_LINE: Paints line at given coords. Uses pen. Params: x, y, x2, y2.
* CANVAS_PIXEL: Paints one pixel at given coords. Params: x, y, color.
* CANVAS_RECT: Paints rectangle. Uses pen and brush. Params: x, y, x2, y2.
* CANVAS_RECT_FRAME: Paints rectangle. Uses only pen. Params: x, y, x2, y2.
* CANVAS_RECT_FILL: Paints rectangle. Uses only brush. Params: x, y, x2, y2.
* CANVAS_RECT_ROUND: Paints rounded rectangle. Uses pen and brush. Params: x, y, x2, y2, style - radius of corners.
* CANVAS_ELLIPSE: Paints ellipse or circle. Uses pen and brush. Params: x, y, x2, y2.
* CANVAS_POLYGON: Paints polygon from any number of points (>2). Uses pen and brush. Params: text - comma separated list of (x,y) coords. Example: "10,10,200,50,10,100" - 3 points.
* CANVAS_SET_TESTPANEL: Sets height of testing panel at the top. Params: size. If it's "too small", panel hides; for big size, size is limited.

</details>
<details><summary>* <b> timer_proc </b> &nbsp; &nbsp; [Global funcs >  timer_proc ]</summary>  
<details><summary>&lt;descr&gt;</summary>  
timer_proc(id, callback, interval, tag="")

Perform action on timers. Many different timers allowed, they work at the same time, each uniq callback - makes new timer with its own interval. To stop some timer, you must specify same callback as on start.

* "callback": Callback which is called on timer tick, see below.
* "interval": Timer delay in msec, 150 or bigger. Specify it only on starting (ignored on stopping).
* "tag": Some string, if not empty, it will be parameter to callback. If empty, callback is called without additional params.

Possible values of "id":

* TIMER_START - Create and start timer, for infinite ticks. If timer for such callback is already created, then it's restated.
* TIMER_START_ONE - Create and start timer, for single tick. If timer for such callback is already created, then it's restated.
* TIMER_STOP - Stop timer (timer must be created before).
* TIMER_DELETE - Stop timer, and delete it from list of timers. Usually don't use it, use only to save memory if created lot of timers.

Result is True if params ok; False if params not ok (callback str incorrect, interval incorrect, not created callback on stopping); or None (for unknown id).

===================

</details>

</details>
<details><summary>* <b>menu_proc</b> &nbsp; &nbsp; [Global funcs > menu_proc]</summary>  
<details><summary>&lt;descr&gt;</summary>  
menu_proc(id_menu, id_action, command="", caption="", index=-1, hotkey="", tag="")

Perform action on menu items.

===================

</details>
<details><summary>* * <b>Menu id</b> &nbsp; &nbsp; [Global funcs > menu_proc > Menu id]</summary>  

Value of "id_menu" can be:

* number, str(number): all menu items can be specified by unique int value
* "top": top menu
* "top-file": top menu "File" submenu
* "top-edit": top menu "Edit" submenu
* "top-sel": top menu "Selection" submenu
* "top-sr": top menu "Search" submenu
* "top-view": top menu "View" submenu
* "top-op": top menu "Options" submenu
* "top-help": top menu "Help" submenu
* "text": editor context menu
* "tab": tab header context menu
* "toolmenu:"+name: dropdown submenu of toolbar button

===================

</details>
<details><summary>* * <b>Command for new items</b> &nbsp; &nbsp; [Global funcs > menu_proc > Command for new items]</summary>  

Value of "command" parameter for MENU_ADD can be:

* int_command or str(int_command) - int command code, from module cudatext_cmd (pass 0 if item does nothing)
* callback in one of these forms: [[#Callback_param]].
* (deprecated callback form) callback in the form "module,method" or "module,method,param" (param can be of any primitive type).

* to create standard special sub-menus, special values:
** "_recents": Recent-files submenu
** "_plugins": Plugins submenu
** "_oplugins": Settings-plugins submenu

* empty string, if item will be used as submenu (item is a submenu, if any subitems are added to it)

===================

</details>
<details><summary>* * <b>Item props as dict</b> &nbsp; &nbsp; [Global funcs > menu_proc > Item props as dict]</summary>  

Some actions get dict for menu items. Dict keys:

* "id": menu_id, int
* "cap": caption, str (for separator items it is "-")
* "cmd": command code (e.g. from module cudatext_cmd), int
* "hint": callback (used if "cmd" value <=0), str
* "hotkey": hotkey, str
* "command": combined command description: int value of "cmd" (if code>0), or str value of "hint" (otherwise)
* "tag": plugin-defined tag, str
* "en": enabled state, bool
* "vis": visible state, bool
* "checked": checked state, bool
* "radio": radio-item state, bool (it means round checkmark in checked state)

===================

</details>
<details><summary>* * <b>Actions</b> &nbsp; &nbsp; [Global funcs > menu_proc > Actions]</summary>  

Possible values of "id_action":

* MENU_CLEAR: Removes all sub-items from menu item.
* MENU_ENUM: Enumerates sub-items of the menu item. Returns list of dict, or None (for incorrect menu_id).
* MENU_GET_PROP: Returns props of menu item, as dict.

* MENU_ADD: Adds sub-item to menu item. Returns string, menu_id for newly added sub-item. Params:
** "id_menu": Item in which you add sub-item.
** "caption": Caption of item, or "-" for menu separator.
** "index": Index (0-based) at which to insert sub-item. Default: append item to end.
** "command": Values are described above.
** "hotkey": String of hotkey (e.g. "Ctrl+Shift+A"). Hotkey combos are not allowed. It overrides hotkey, which is auto assigned from command code.
** "tag": Any string stored in menu item.

* MENU_REMOVE: Deletes menu item. Note: don't remove items, which are auto-updated by CudaText, because you'll get Access Violation on showing menu containing these items. E.g. some items in the tab context menu are auto-updated.
* MENU_CREATE: Creates new popup-menu, independent from CudaText menus. It can be filled like other menus, then shown via MENU_SHOW.
* MENU_SHOW: Shows given popup-menu. Only menu created with MENU_CREATE should be specified here. Param "command" must be tuple (x,y) or string "x,y" with screen-related coordinates, if empty - menu shows at mouse cursor.

* MENU_SET_CAPTION: Changes caption of menu item. Param "command" must be str value.
* MENU_SET_VISIBLE: Changes visible state of menu item. Param "command" must be bool value.
* MENU_SET_ENABLED: Changes enabled state of menu item. Param "command" must be bool value.
* MENU_SET_HOTKEY: Changes hotkey of menu item. Param "command" must be str value, e.g. "Ctrl+Alt+F1".
* MENU_SET_CHECKED: Changes checked state of menu item. When item is checked, it shows a checkmark. Param "command" must be bool value.
* MENU_SET_RADIOITEM: Changes radio kind of menu item. When item has radio kind, its checkmark is round (in checked state). Param "command" must be bool value.
* MENU_SET_IMAGELIST: Changes image-list object of menu, which contains menu item. Param "command" must be image-list handle.
* MENU_SET_IMAGEINDEX: Changes icon index of menu item (index in image-list). Param "index" must be icon index.

===================

</details>

</details>
<details><summary>* <b>toolbar_proc</b> &nbsp; &nbsp; [Global funcs > toolbar_proc]</summary>  

toolbar_proc(id_toolbar, id_action, text="", text2="", command=0, index=-1, index2=-1)

Perform action on toolbar UI control.

Param "id_toolbar": int handle of toolbar. Can be "top" for main app toolbar. Function returns None, if id_toolbar not correct.

Param "id_action" possible values:

* TOOLBAR_GET_COUNT: Returns number of buttons in toolbar.
* TOOLBAR_GET_IMAGELIST: Returns int handle of image-list object.
* TOOLBAR_GET_BUTTON_HANDLE: Returns int handle of button to use in button_proc(), or None if index not correct. Param "index": button index.
* TOOLBAR_GET_BUTTON_HANDLES: Returns list of handles of all buttons.
* TOOLBAR_GET_INDEX_HOVERED: Returns index of button under mouse cursor, or -1 if none.
* TOOLBAR_DELETE_ALL: Deletes all buttons.
* TOOLBAR_DELETE_BUTTON: Deletes one button. Param "index": button index.
* TOOLBAR_ADD_ITEM: Adds one button, of usual kind. Returns its handle. Param "index": button index at which to insert, -1 to append.
* TOOLBAR_ADD_MENU: Adds one button, with submenu appearing by left click (not context menu). Returns its handle. Param "index": button index at which to insert, -1 to append.
* TOOLBAR_UPDATE: Updates buttons positions and sizes (for current size of image-list, current captions etc).
* TOOLBAR_GET_VERTICAL: Returns vertical state of toolbar.
* TOOLBAR_SET_VERTICAL: Sets vertical state of toolbar. Param "index": bool value.
* TOOLBAR_GET_WRAP: Returns wrappable state of toolbar.
* TOOLBAR_SET_WRAP: Sets wrappable state of toolbar (implemented for horizontal toolbar only). Param "index": bool value.
* TOOLBAR_THEME: Applies current UI theme to toolbar.

</details>
<details><summary>* <b>statusbar_proc</b> &nbsp; &nbsp; [Global funcs > statusbar_proc]</summary>  

statusbar_proc(id_statusbar, id_action, index=-1, tag=0, value="")

Perform action on statusbar UI control.

* Param "id_statusbar": handle of control. Can be "main" for main program statusbar.
* Param "index": index of cell (0-based), if action needs it.
* Param "tag": int tag of cell. Tag value, if not 0, overrides "index" param. It's needed to address a cell without knowing its index, only by tag. CudaText addresses standard cells by tag in the range 1..20.

Param "id_action" can be:

* STATUSBAR_GET_COUNT: Returns int number of cells.
* STATUSBAR_DELETE_ALL: Deletes all cells.
* STATUSBAR_DELETE_CELL: Deletes one cell (by "index" or "tag").

* STATUSBAR_ADD_CELL: Adds one cell. Param "index": -1: cell will be appended; >=0: cell will be inserted at given index. Param "tag" has special meaning: it is tag value of a new cell. Returns index of new cell, or None if cannot add (e.g. "tag" is busy).
* STATUSBAR_FIND_CELL: Returns index if cell from tag, or None. Param "value": int tag. Params "index", "tag" ignored.

* STATUSBAR_GET_IMAGELIST: Returns handle of image-list object, attached to statusbar, or 0 for none.
* STATUSBAR_SET_IMAGELIST: Sets handle if image-list object. Param "value": handle.

* STATUSBAR_GET_COLOR_BACK: Returns color of background.
* STATUSBAR_GET_COLOR_FONT: Returns color of font.
* STATUSBAR_GET_COLOR_BORDER_TOP: Returns color of border-top.
* STATUSBAR_GET_COLOR_BORDER_L: Returns color of border-left.
* STATUSBAR_GET_COLOR_BORDER_R: Returns color of border-right.
* STATUSBAR_GET_COLOR_BORDER_U: Returns color of border-up.
* STATUSBAR_GET_COLOR_BORDER_D: Returns color of border-down.

* STATUSBAR_SET_COLOR_BACK: Sets color of background. Param "value": int color.
* STATUSBAR_SET_COLOR_FONT: Sets color of font.
* STATUSBAR_SET_COLOR_BORDER_TOP: Sets color of border-top.
* STATUSBAR_SET_COLOR_BORDER_L: Sets color of border-left.
* STATUSBAR_SET_COLOR_BORDER_R: Sets color of border-right.
* STATUSBAR_SET_COLOR_BORDER_U: Sets color of border-up.
* STATUSBAR_SET_COLOR_BORDER_D: Sets color of border-down.

* STATUSBAR_GET_CELL_SIZE: Returns width of cell.
* STATUSBAR_GET_CELL_AUTOSIZE: Returns auto-size of cell, bool. Auto-size: adjust width of cell to its icon+text.
* STATUSBAR_GET_CELL_AUTOSTRETCH: Returns auto-stretch of cell, bool. Auto-stretch: stretch cell to fill entire statusbar width.
* STATUSBAR_GET_CELL_ALIGN: Returns alignment of cell. One of str values: "L" (left), "C" (center), "R" (right).
* STATUSBAR_GET_CELL_TEXT: Returns text of cell.
* STATUSBAR_GET_CELL_HINT: Returns hint of cell.
* STATUSBAR_GET_CELL_IMAGEINDEX: Returns icon index (inside image-list) of cell, -1 for none.
* STATUSBAR_GET_CELL_COLOR_FONT: Returns font color of cell, COLOR_NONE if not set.
* STATUSBAR_GET_CELL_COLOR_BACK: Returns background color of cell, COLOR_NONE if not set.
* STATUSBAR_GET_CELL_FONT_NAME: Returns special font name of cell (if empty str, not used).
* STATUSBAR_GET_CELL_FONT_SIZE: Returns special font size of cell (if <=0, not used).
* STATUSBAR_GET_CELL_TAG: Returns int tag of cell.
* STATUSBAR_GET_CELL_CALLBACK: Returns callback string of cell.

* STATUSBAR_SET_CELL_SIZE: Sets width of cell. Param "value": int.
* STATUSBAR_SET_CELL_AUTOSIZE: Sets auto-size of cell. Param "value": bool.
* STATUSBAR_SET_CELL_AUTOSTRETCH: Sets auto-stretch of cell. Param "value": bool.
* STATUSBAR_SET_CELL_ALIGN: Sets alignment of cell. Param "value": str constant: "L", "C", "R".
* STATUSBAR_SET_CELL_TEXT: Sets text of cell. Param "value": str.
* STATUSBAR_SET_CELL_HINT: Sets hint of cell. Param "value": str.
* STATUSBAR_SET_CELL_IMAGEINDEX: Sets icon index (inside image-list) of cell, -1 for none. Param "value": int.
* STATUSBAR_SET_CELL_COLOR_FONT: Sets font color of cell. Param "value": int color or COLOR_NONE.
* STATUSBAR_SET_CELL_COLOR_BACK: Sets background color of cell. Param "value": int color or COLOR_NONE.
* STATUSBAR_SET_CELL_FONT_NAME: Sets special font name of cell (if empty str, not used). Param "value": str.
* STATUSBAR_SET_CELL_FONT_SIZE: Sets special font size of cell (if <=0, not used). Param "value": int.
* STATUSBAR_SET_CELL_TAG: Sets tag of cell. Param "value": int.
* STATUSBAR_SET_CELL_CALLBACK: Sets callback string of cell.

Notes:

* If cell text not empty, alignment applies only for text, icon is on the left. If text empty, alignment applies for icon.

</details>
<details><summary>* <b>imagelist_proc</b> &nbsp; &nbsp; [Global funcs > imagelist_proc]</summary>  

imagelist_proc(id_list, id_action, value="")

Perform action on image-list object.

Param "id_list" is int handle of image-list. It is required for all actions, except IMAGELIST_CREATE, where it should be 0.

Possible values of "id_action":

* IMAGELIST_CREATE: Creates new image-list object with default icon size 16x16. Returns int handle of this image-list. Param "value" must be int handle of owner form of object.
** If it is form handle from dlg_proc, object will be deleted after deletion of this form.
** If it is 0, main application form is used as owner, and object will be persistent.

* IMAGELIST_COUNT: Returns int number of icons in image-list.
* IMAGELIST_GET_SIZE: Returns current icon size as 2-tuple (width, height).
* IMAGELIST_SET_SIZE: Sets new icon size, and clears image-list. Param "value" must be 2-tuple of int (width, height). Returns new icon size (corrected by minimal value) as 2-tuple.
* IMAGELIST_ADD: Loads image into image-list. Param "value" must be full path to png/bmp image file. Image size should be the same as size in image-list (but not required). Returns int icon index, or None if cannot load.
* IMAGELIST_DELETE: Deletes one icon. Param "value" is int icon index (0-based).
* IMAGELIST_DELETE_ALL: Deletes all icons.
* IMAGELIST_PAINT: Paints single icon on given canvas, at given coords. Param "value" must be tuple (canvas_id, x, y, icon_index). Value canvas_id can be 0 for testing paintbox in CudaText.

</details>
<details><summary>* <b>image_proc</b> &nbsp; &nbsp; [Global funcs > image_proc]</summary>  

image_proc(id_image, id_action, value="")

Perform action on image object.

Param "id_image" is int handle of image. It is required for all actions, except IMAGE_CREATE, where it should be 0.

Possible values of "id_action":

* IMAGE_CREATE: Creates new image object. Returns int handle of this image. Param "value" must be int handle of owner form of object.
** If it is form handle from dlg_proc, object will be deleted after deletion of this form.
** If it is 0, main application form is used as owner, and object will be persistent.

* IMAGE_GET_SIZE: Returns current image size as 2-tuple (width, height).
* IMAGE_LOAD: Reads picture file into image object. Param "value" must be full file path (png, jpg, bmp, gif, ico).
* IMAGE_PAINT: Paints image object on given canvas, at given coords. Param "value" must be tuple (canvas_id, x, y). Value canvas_id can be 0, for testing paintbox in CudaText.
* IMAGE_PAINT_SIZED: Paints image object on given canvas, resized to given rectangle. Param "value" must be tuple (canvas_id, x1, y1, x2, y2).

</details>
<details><summary>* <b>button_proc</b> &nbsp; &nbsp; [Global funcs > button_proc]</summary>  

button_proc(id_button, id_action, value="")

Perform action on extended button (control type "button_ex").

Param "id_button" is int handle of button.

Possible values of "id_action":

* BTN_UPDATE: Repaints button.
* BTN_GET_TEXT: Returns caption string.
* BTN_SET_TEXT: Sets caption string.
* BTN_GET_HINT: Returns hint (tooltip) string.
* BTN_SET_HINT: Sets hint string.
* BTN_GET_ENABLED: Returns enabled state, bool.
* BTN_SET_ENABLED: Sets enabled state. Param "value" must be bool.
* BTN_GET_VISIBLE: Returns visible state, bool.
* BTN_SET_VISIBLE: Sets visible state. Param "value" must be bool.
* BTN_GET_CHECKED: Returns checked state, bool.
* BTN_SET_CHECKED: Sets checked state. Param "value" must be bool.
* BTN_GET_IMAGELIST: Returns handle of image-list for button.
* BTN_SET_IMAGELIST: Sets handle of image-list. Param "value" must be int handle.
* BTN_GET_IMAGEINDEX: Returns icon index (in attached image-list).
* BTN_SET_IMAGEINDEX: Sets icon index. Param "value" must be int index (0-based), or -1 for none. To show icon, you must also set appropriate kind of button.
* BTN_GET_MENU: Returns int handle of submenu. It can be passed to menu_proc(h, MENU_SHOW).
* BTN_SET_MENU: Sets int handle of submenu. It can be handle created by menu_proc(0, MENU_CREATE).
* BTN_GET_KIND: Returns kind of button. Int value, one of BTNKIND_nnn constants.
* BTN_SET_KIND: Sets kind of button.
* BTN_GET_BOLD: Returns bold-style of button, bool.
* BTN_SET_BOLD: Sets bold-style. Param "value" must be bool.
* BTN_GET_ARROW: Returns dropdown arrow visible flag, bool.
* BTN_SET_ARROW: Sets dropdown arrow visible flag. Param "value" must be bool.
* BTN_GET_ARROW_ALIGN: Returns alignment of dropdown arrow, as str: "L", "R", "C".
* BTN_SET_ARROW_ALIGN: Sets alignment of dropdown arrow. Param "value" must be str: "L", "R", "C".
* BTN_GET_FOCUSABLE: Returns focusable state, bool.
* BTN_SET_FOCUSABLE: Sets focusable state. Param "value" must be bool.
* BTN_GET_FLAT: Returns flat state, bool.
* BTN_SET_FLAT: Sets flat state. Param "value" must be bool.
* BTN_GET_DATA1: Returns data1 string. Data1 contains:
** str(integer_command_code) to run command by number like Editor.cmd()
** or string with [[#Callback_param]]. Note: not any callback form, only string callback form
* BTN_SET_DATA1: Sets data1 string. See BTN_GET_DATA1.
* BTN_GET_DATA2: Returns data2 string. Data2 is currently not used by CudaText.
* BTN_SET_DATA2: Sets data2 string. See BTN_GET_DATA2.
* BTN_GET_WIDTH: Returns width (in pixels).
* BTN_SET_WIDTH: Sets width.
* BTN_GET_ITEMS: Returns choice items, used for kind=BTNKIND_TEXT_CHOICE, as "\n"-separated strings.
* BTN_SET_ITEMS: Sets choice items. Param "value" must be str, "\n"-separated strings.
* BTN_GET_ITEMINDEX: Returns choice index, used for kind=BTNKIND_TEXT_CHOICE.
* BTN_SET_ITEMINDEX: Sets choice index. Param "value" must be int >=0, or -1 for none.

Note: Toolbars contain several "button_ex" objects, which are anchored one to another (horizontally or vertically). You can also construct such toolbar by hands. API toolbar_proc() don't allow to specify kind of buttons, it sets kind from button properties.

</details>

</details>

---
<details><summary><b>Editor class</b></summary>  
<details><summary>&lt;descr&gt;</summary>  
Editor class has methods to work with editor. Global objects of Editor exist:

* "ed": refers to currently focused editor (in any tab).
* "ed_con_log": refers to log field (multi-line) in the "Console" panel.
* "ed_con_in": refers to input field (single line) in the "Console" panel.

</details>
<details><summary>* <b>Carets</b> &nbsp; &nbsp; [Editor class > Carets]</summary>  
<details><summary>* * <b>Editor.get_carets</b> &nbsp; &nbsp; [Editor class > Carets > Editor.get_carets]</summary>  

get_carets()

Returns list of 4-tuples, each item is info about one caret: (PosX, PosY, EndX, EndY).

* PosX is caret's column (0-base). Tab-chars give x increment 1, like others.
* PosY is caret's line (0-base).
* EndX/EndY is position of selection edge for this caret. Both -1 if no selection for caret.

Example shows text of first caret's selection:

<syntaxhighlight lang="Python">
carets = ed.get_carets()
x1, y1, x2, y2 = carets[0]
if y2>=0:
# sort (y,x) pairs
if (y1, x1)>(y2, x2):
x1, y1, x2, y2 = x2, y2, x1, y1
s = ed.get_text_substr(x1, y1, x2, y2)
msg_status('Selection: '+s)
</syntaxhighlight>

===================

</details>

</details>
<details><summary>* <b>Text read/write</b> &nbsp; &nbsp; [Editor class > Text read/write]</summary>  
<details><summary>&lt;descr&gt;</summary>  

===================

</details>
<details><summary>* * <b>Editor.get_text_all</b> &nbsp; &nbsp; [Editor class > Text read/write > Editor.get_text_all]</summary>  

get_text_all()

Returns the entire editor text, as string.
Returns "\n" as line-breaks.

===================

</details>
<details><summary>* * <b>Editor.set_text_all</b> &nbsp; &nbsp; [Editor class > Text read/write > Editor.set_text_all]</summary>  

set_text_all(text)

Sets the entire editor text, from given string.
Text can have any line-breaks (LF, CRLF, CR, mixed).

Notes:
* Function cannot work with read-only editor (see Editor.get_prop, Editor.set_prop and PROP_RO).
* Function looses Undo information. To support Undo, use another API to set text, e.g. Editor.replace().

===================

</details>
<details><summary>* * <b>Editor.get_text_line</b> &nbsp; &nbsp; [Editor class > Text read/write > Editor.get_text_line]</summary>  

get_text_line(index, max_len=0)

Returns single line (str) with given index (0-based).
Returns None if index incorrect.

If param "max_len" is non-zero, and UTF-8 length of given line is greater than "max_len", function returns empty str. This allows to skip huge lines.

===================

</details>
<details><summary>* * <b>Editor.set_text_line</b> &nbsp; &nbsp; [Editor class > Text read/write > Editor.set_text_line]</summary>  

set_text_line(index, text)

Sets single line (str) with given index (0-based).
Text must be without line-breaks.
To add new line, pass index=-1.

===================

</details>
<details><summary>* * <b>Editor.get_text_substr</b> &nbsp; &nbsp; [Editor class > Text read/write > Editor.get_text_substr]</summary>  

get_text_substr(x1, y1, x2, y2)

Returns substring from position (x1, y1) to bigger position (x2, y2).

===================

</details>
<details><summary>* * <b>Editor.delete</b> &nbsp; &nbsp; [Editor class > Text read/write > Editor.delete]</summary>  

delete(x1, y1, x2, y2)

Deletes range from position (x1, y1) to bigger position (x2, y2).

* Too big x1/x2 are allowed (after line-end)
* Too big y2 means delete to end of file

Note: don't pass tuple from get_carets()[0], this tuple has not sorted pos=(x1, y1), end=(x2, y2), you need to sort them (first sort by y, then by x).

Example replaces selection of 1st caret with text:

<syntaxhighlight lang="Python">
x0, y0, x1, y1 = ed.get_carets()[0]
if (y0, x0) >= (y1, x1): #note that y first
x0, y0, x1, y1 = x1, y1, x0, y0

ed.set_caret(x0, y0)
ed.delete(x0, y0, x1, y1)
ed.insert(x0, y0, text)
</syntaxhighlight>

===================

</details>
<details><summary>* * <b>Editor.insert</b> &nbsp; &nbsp; [Editor class > Text read/write > Editor.insert]</summary>  

insert(x, y, text)

Inserts given text at position (x, y). If y too big, appends block to end (even to final line w/out line-end).
Text can be multi-line, all CR LF are converted to currently used line-breaks.

Returns 2-tuple (x, y) of position after inserted text. It is on the same line, if text is single line. Returns None if cannot insert.

===================

</details>
<details><summary>* * <b>Editor.replace</b> &nbsp; &nbsp; [Editor class > Text read/write > Editor.replace]</summary>  

replace(x1, y1, x2, y2, text)

Replaces range from position (x1, y1) to bigger position (x2, y2), with new text.

* Too big x1/x2 are allowed (after line-end)
* Too big y2 means delete to end of file

Function does the same as delete+insert, but

* optimized for replace inside one line (when y1==y2 and no line-breaks in text)
* for multi-line it also makes grouped-undo for delete+insert

Returns 2-tuple (x, y) of position after inserted text.

===================

</details>

</details>
<details><summary>* <b>Selection</b> &nbsp; &nbsp; [Editor class > Selection]</summary>  
<details><summary>&lt;descr&gt;</summary>  

===================

</details>
<details><summary>* * <b>Editor.get_text_sel</b> &nbsp; &nbsp; [Editor class > Selection > Editor.get_text_sel]</summary>  

get_text_sel()

Returns selected text for 1st caret (empty, if no selection).

===================

</details>
<details><summary>* * <b>Editor.get_sel_mode</b> &nbsp; &nbsp; [Editor class > Selection > Editor.get_sel_mode]</summary>  

get_sel_mode()

Returns current selection mode:

* SEL_NORMAL: normal (stream) selection; it doesn't mean that selection exists.
* SEL_COLUMN: column (vertical) selection.

===================

</details>
<details><summary>* * <b>Editor.get_sel_lines</b> &nbsp; &nbsp; [Editor class > Selection > Editor.get_sel_lines]</summary>  

get_sel_lines()

Returns 2-tuple, indexes of first and last lines affected by first caret selection. Returns (-1,-1) if no selection.

===================

</details>

</details>
<details><summary>* <b>Properties</b> &nbsp; &nbsp; [Editor class > Properties]</summary>  
<details><summary>&lt;descr&gt;</summary>  

===================

</details>
<details><summary>* * <b>Editor.get_line_count</b> &nbsp; &nbsp; [Editor class > Properties > Editor.get_line_count]</summary>  

get_line_count()

Returns number of lines.

===================

</details>
<details><summary>* * <b>Editor.get_filename</b> &nbsp; &nbsp; [Editor class > Properties > Editor.get_filename]</summary>  

get_filename(options="")

Returns filename (str) of the editor.

* Returns empty string for untitled tab.
* Returns string "?" if tab contains not normal text editor. See: get_prop(PROP_KIND).

If param "options" contains char "*", the real file name is always returned, even in hex/picture viewer mode.

===================

</details>
<details><summary>* * <b>Editor.get_prop</b> &nbsp; &nbsp; [Editor class > Properties > Editor.get_prop]</summary>  

get_prop(id, value="")

Returns editor's property.

Param "value" can be: str, number, bool (for bool it can be also "0"/"1"). Possible values of "id":

* PROP_GUTTER_ALL: bool: gutter (container for all gutter columns) is visible.
* PROP_GUTTER_STATES: bool: show gutter column "line states".
* PROP_GUTTER_NUM: bool: show gutter column "line numbers".
* PROP_GUTTER_FOLD: bool: show gutter column "folding".
* PROP_GUTTER_BM: bool: show gutter column "bookmarks".
* PROP_NEWLINE: str: kind of line endings. One of values "lf", "crlf", "cr".
* PROP_WRAP: int: word-wrap mode. One of WRAP_nnn values.
* PROP_RO: bool: read-only mode.
* PROP_MARGIN: int: position of fixed margin.
* PROP_MARGIN_STRING: str: user-defined margins positions, e.g. "20 25".
* PROP_INSERT: bool: insert/overwrite mode.
* PROP_MODIFIED: bool: editor is modified.
* PROP_MODIFIED_VERSION: int: counter which is incremented on each text change.
* PROP_RULER: bool: horz ruler is shown.

* PROP_LINE_STATE: int: state of the line with given index. One of LINESTATE_nnn values.
* PROP_LINE_STATES: list of int: list of line states (LINESTATE_nnn values) for all lines.
* PROP_LINE_STATES_UPDATED: list of bool: list of "updated" flags for all lines, each flag shows that its line was added/changed since the last clearing of this "updated" flag.

* PROP_LINE_NUMBERS: int: style of line numbers. One of LINENUM_nnn values.
* PROP_LINE_TOP: int: index of line visible at the top of editor.
* PROP_LINE_BOTTOM: int: index of line visible at the bottom of editor (considers word wrap).

* PROP_SCROLL_VERT: int: approximate vertical scroll position. Index in WrapInfo list, you can read this list via get_wrapinfo().
* PROP_SCROLL_HORZ: int: approximate horizontal scroll position.
* PROP_SCROLL_VERT_SMOOTH: int: smooth vertical scroll position. In pixels.
* PROP_SCROLL_HORZ_SMOOTH: int: smooth horizontal scroll position. In pixels.
* PROP_SCROLL_VERT_INFO: int: dict with all available info about vertical scroll.
* PROP_SCROLL_HORZ_INFO: int: dict with all available info about horizontal scroll.
* PROP_SCROLLSTYLE_HORZ: int: style of horizontal scrollbar, one of SCROLLSTYLE_ constants.
* PROP_SCROLLSTYLE_VERT: int: style of vertical scrollbar, one of SCROLLSTYLE_ constants.

* PROP_SCALE_FONT: int: scale of editor's font in percents, or 0 to use global app font scale.
* PROP_COLOR: int: color property. Value must be one of COLOR_ID_nnn values. Returns None for incorrect id.
* PROP_ENC: str: encoding name. Names are listed at [[CudaText#Encodings]].
* PROP_LEXER_FILE: str: name of lexer for entire file (empty str if none is active).
* PROP_LEXER_POS: str: name of lexer at specified position. Param "value" must be 2-tuple (column, line), or string str(column)+','+str(line), with 0-based indexes.
* PROP_LEXER_CARET: str: name of lexer at the position of 1st caret.
* PROP_INDEX_GROUP: int: index of group with editor's tab, 0-based.
* PROP_INDEX_TAB: int: index of editor's tab in group, 0-based.
* PROP_UNPRINTED_SHOW: bool: unprinted chars: global enable-flag.
* PROP_UNPRINTED_SPACES: bool: unprinted chars: show spaces/tabs.
* PROP_UNPRINTED_SPACES_TRAILING: bool: unprinted chars: show spaces/tabs only at end of lines.
* PROP_UNPRINTED_ENDS: bool: unprinted chars: show line ends.
* PROP_UNPRINTED_END_DETAILS: bool: unprinted chars: show line end details.
* PROP_TAG: str: some string attached to editor. Value must be "key:defvalue" or simply "defvalue". Saved value for "key" is returned, or "defvalue" returned if value for key was not set. Empty key means key "_".
* PROP_CARET_VIEW: 3-tuple: shape of caret, normal mode. Tuple contents is (int_width, int_height, bool_empty_inside). Width/height<0 means value in percents.
* PROP_CARET_VIEW_OVR: 3-tuple: shape of caret, overwrite mode.
* PROP_CARET_VIEW_RO: 3-tuple: shape of caret, read-only mode.
* PROP_CARET_VIRTUAL: bool: caret position is allowed after line-ends.
* PROP_CARET_STOP_UNFOCUSED: bool: caret blinks only when editor is focused.
* PROP_MACRO_REC: bool: currently macro is recording.
* PROP_MARKED_RANGE: 2-tuple with line indexes of "marked range"; (-1, -1) if range not set.
* PROP_VISIBLE_LINES: int: max count of lines that fit to window (doesn't consider word wrap).
* PROP_VISIBLE_COLUMNS: int: max count of columns that fit to window.
* PROP_PICTURE: properties of picture file as 3-tuple: (picture_filename, size_x, size_y), or None if not picture loaded in tab. For picture ed.get_filename() returns "?".
* PROP_MINIMAP: bool: minimap is visible.
* PROP_MICROMAP: bool: micromap is visible.
* PROP_LINK_AT_POS: str: URL in the document, at given position. Value must be str(pos_x)+","+str(pos_y).
* PROP_LINKS_SHOW: bool: enable hyperlinks underlining.
* PROP_LINKS_REGEX: str: regular expression for hyperlinks.
* PROP_LINKS_CLICKS: int: how mouse clicks activate hyperlinks. 0: disabled, 1: single click, 2: double click.
* PROP_IN_SESSION: bool: document was loaded from "session".
* PROP_IN_HISTORY: bool: document history was loaded from history file (history embedded to "session" or main history file).
* PROP_TAB_SPACES: bool: tab-key inserts spaces (not tab-char).
* PROP_TAB_SIZE: int: size of tab-char.
* PROP_TAB_COLLECT_MARKERS: bool: tab-key collects (jumps to and deletes) markers (if markers placed).
* PROP_TAB_TITLE: str: title of tab, useful for untitled tabs, for tabs with picture files.
* PROP_TAB_COLOR: int: color of tab containing editor; COLOR_NONE if not set.
* PROP_TAB_ICON: int: index of tab icon, ie index in imagelist, which handle you can get via app_proc(PROC_GET_TAB_IMAGELIST).
* PROP_TAB_ID: int: unique tab's identifier (one number for main/secondary editors in tab), it is not changed when tab is moved.
* PROP_INDENT_SIZE: int: size of indent for Indent/Unindent commands. N>0: indent in spaces, N<0: indent in tabs, N=0: indent from PROP_TAB_SIZE/PROP_TAB_SPACES.
* PROP_INDENT_KEEP_ALIGN: bool: command Unindent keeps alignment of block edge, ie don't unindent if any line reached column 0.
* PROP_INDENT_AUTO: bool: makes next line also indented on Enter command.
* PROP_INDENT_KIND: int: kind of auto-indented line, see description of app option "indent_kind".
* PROP_FOLD_ALWAYS: bool: always show icons on "folding" gutter bar, otherwise show them only on mouse over.
* PROP_FOLD_ICONS: int: icon set for "folding" gutter bar. Possible values: 0, 1.
* PROP_FOLD_TOOLTIP_SHOW: bool: show floating tooltip when mouse hovers [...] mark for folded block.
* PROP_LAST_LINE_ON_TOP: bool: allow to scroll control, so last line shows on top.
* PROP_HILITE_CUR_COL: bool: highlight current column.
* PROP_HILITE_CUR_LINE: bool: highlight current line.
* PROP_HILITE_CUR_LINE_MINIMAL: bool: highlight current line, only minimal part of line.
* PROP_HILITE_CUR_LINE_IF_FOCUS: bool: highlight current line, only when editor focused.
* PROP_CODETREE: bool: enable standard code-tree filling.
* PROP_CODETREE_MODIFIED_VERSION: modification version of the editor for the moment of code-tree filling. See PROP_MODIFIED_VERSION.
* PROP_CODETREE_SUBLEXER: bool: enable code-tree to show nodes from sublexer(s) of current lexer.
* PROP_EDITORS_LINKED: bool: enable sharing of the same text by primary/secondary editors in file tab.
* PROP_CELL_SIZE: 2-tuple of int: size in pixels of average char cell.
* PROP_ONE_LINE: bool: editor has single-line mode. (Usual editors are multi-line.)
* PROP_MODERN_SCROLLBAR: bool: use custom-drawn themed scrollbars.
* PROP_SAVE_HISTORY: bool: allows to save history (caret, scroll pos, folding etc) on file closing.
* PROP_PREVIEW: bool: editor is inside "Preview tab" (it has italic caption).
* PROP_UNDO_GROUPED: bool: editor undo/redo is grouped.
* PROP_UNDO_LIMIT: int: max count of simple actions, which can be undone.
* PROP_UNDO_DATA: str: string representation of Undo data.
* PROP_REDO_DATA: str: string representation of Redo data.
* PROP_ZEBRA: int: if 0, "zebra" mode is not active; if >0, it is alpha-blend value (1..255) of active zebra mode.
* PROP_ZEBRA_STEP: int: step (in lines) of "zebra" mode.
* PROP_FONT: 2-tuple (str, int): normal font: name, size.
* PROP_FONT_B: 2-tuple (str, int): bold font: name, size. Empty name: not used.
* PROP_FONT_I: 2-tuple (str, int): italic font: name, size. Empty name: not used.
* PROP_FONT_BI: 2-tuple (str, int): bold+italic font: name, size. Empty name: not used.
* PROP_ACTIVATION_TIME: int: relative time (in OS timer ticks) when tab ("main" or "brother" editor in the same tab) was last focused. It's 0 for not yet focused tabs. It's None for editors created by dlg_proc().
* PROP_KIND: str: kind of UI control in the tab.
** "text": normal editor
** "bin": binary viewer
** "pic": picture

* PROP_SPLIT: 2-tuple: information about primary/secondary editors splitting as tuple (split_kind, split_pos).
** "split_kind": str: "-" (not splitted), "v" (splitted vertically), "h" (splitted horizontally).
** "split_pos": int: part of entire size for secondary editor, multiplied by 1000 (e.g. 500 is half of size).

* PROP_SAVING_FORCE_FINAL_EOL: bool: on saving file, ensure the newline at end of document.
* PROP_SAVING_TRIM_SPACES: bool: on saving file, trim trailing whitespaces.
* PROP_SAVING_TRIM_FINAL_EMPTY_LINES: bool: on saving file, trim redundant empty lines at end of document.

* PROP_HANDLE_SELF: int: handle of editor object, for which get_prop() is called. This handle is unique among all objects in program (it is memory address).
** To get editor object from handle, use ed_obj=Editor(handle_value).
** Object "ed" has virtual handle 0 for active focused editor, ie ed.h==0. And PROP_HANDLE_SELF returns the actual non-zero handle.
* PROP_HANDLE_PRIMARY: int: handle of editor object, which is primary editor in ui-tab (primary editor is always visible). It's None for editor objects created from dlg_proc().
* PROP_HANDLE_SECONDARY: int: handle of editor object, which is secondary editor in ui-tab (secondary editor is visible only when ui-tab is splitted). It's None for editor objects created from dlg_proc().
* PROP_HANDLE_PARENT: int: handle of parent of editor object, or 0 if no such parent. This handle is needed to dock forms near the editor.

* PROP_COORDS: 4-tuple of int: rectangle of entire editor area, relative to screen.
* PROP_RECT_CLIENT: 4-tuple of int: rectangle of entire editor area, relative to editor.
* PROP_RECT_TEXT: 4-tuple of int: rectangle of text-only editor area (excluding gutter/minimap/etc), relative to editor.

* PROP_COMBO_ITEMS: only for "editor_combo" control, contents of drop-down list. On getting: list of str. On setting: value must be str with '\n'-separated items.

* PROP_V_MODE: int: mode of binary viewer. Reading it for editor, returns VMODE_NONE. Reading it for viewer, returns one of: VMODE_TEXT, VMODE_BINARY, VMODE_HEX, VMODE_UNICODE, VMODE_UNICODE_HEX.
* PROP_V_POS: int: for binary viewer, scroll position.
* PROP_V_SEL_START: int: for binary viewer, selection start offset.
* PROP_V_SEL_LEN: int: for binary viewer, selection length.
* PROP_V_WIDTH: int: for binary viewer, width of text in mode VMODE_BINARY.
* PROP_V_WIDTH_HEX: int: for binary viewer, width of text in mode VMODE_HEX.
* PROP_V_WIDTH_UHEX: int: for binary viewer, width of text in mode VMODE_UNICODE_HEX.

===================

</details>
<details><summary>* * <b>Editor.set_prop</b> &nbsp; &nbsp; [Editor class > Properties > Editor.set_prop]</summary>  

set_prop(id, value)

Sets editor's property.

Param "value" can be: str, number, bool (for bool it can be also "0"/"1"), tuple of simple type. Possible values of "id":

* PROP_GUTTER_ALL: bool: gutter (container for all gutter columns) is visible.
* PROP_GUTTER_STATES: bool: show gutter column "line states".
* PROP_GUTTER_NUM: bool: show gutter column "line numbers".
* PROP_GUTTER_FOLD: bool: show gutter column "folding".
* PROP_GUTTER_BM: bool: show gutter column "bookmarks".
* PROP_WRAP: int: word-wrap mode. One of WRAP_nnn values.
* PROP_RO: bool: read-only mode.
* PROP_NEWLINE: str: kind of line endings. One of values "lf", "crlf", "cr".
* PROP_MARGIN: int: position of fixed margin.
* PROP_MARGIN_STRING: str: space-separated user-margins columns.
* PROP_INSERT: bool: insert/overwrite mode.
* PROP_MODIFIED: bool: editor is modified.
* PROP_RULER: bool: show ruler.
* PROP_COLOR: color property, value must be 2-tuple (COLOR_ID_nnnn, int_color_value).
* PROP_LINE_TOP: int: index of line visible at the editor top.
* PROP_LINE_NUMBERS: int: style of line numbers. One of LINENUM_nnn values.
* PROP_LINE_STATE: 2-tuple: state of the line with given index. 2-tuple (line_index, LINESTATE_nnn).
* PROP_LINE_STATES_UPDATED: currently supports writing only empty string value, writing it means clearing of "updated" flags for all lines.

* PROP_SCROLL_VERT: int: approximate vertical scroll position. Index in WrapInfo list, you can read this list via get_wrapinfo().
* PROP_SCROLL_HORZ: int: approximate horizontal scroll position.
* PROP_SCROLL_VERT_SMOOTH: int: smooth vertical scroll position. In pixels.
* PROP_SCROLL_HORZ_SMOOTH: int: smooth horizontal scroll position. In pixels.
* PROP_SCROLLSTYLE_HORZ: int: style of horizontal scrollbar, one of SCROLLSTYLE_ constants.
* PROP_SCROLLSTYLE_VERT: int: style of vertical scrollbar, one of SCROLLSTYLE_ constants.

* PROP_SCALE_FONT: int: scale of editor's font in percents, or 0 to use global app font scale.
* PROP_ENC: str: encoding name. Names listed at [[CudaText#Encodings]].
* PROP_ENC_RELOAD: str: like PROP_ENC, but setting this property also reloads file in new encoding (if it's not modified). Supported only for editors embedded in ui-tabs.
* PROP_LEXER_FILE: str: name of lexer.
* PROP_INDEX_GROUP: int: index of group with editor's tab, 0-based.
* PROP_INDEX_TAB: int: index of editor's tab in group, 0-based.
* PROP_UNPRINTED_SHOW: bool: unprinted chars: global enable-flag.
* PROP_UNPRINTED_SPACES: bool: unprinted chars: show spaces/tabs.
* PROP_UNPRINTED_SPACES_TRAILING: bool: unprinted chars: show spaces/tabs only at end of lines.
* PROP_UNPRINTED_ENDS: bool: unprinted chars: show line ends.
* PROP_UNPRINTED_END_DETAILS: bool: unprinted chars: show line end details.
* PROP_TAG: str: some string attached to editor. Param text must be pair "key:value" or simply "value", this sets value for specified key (internally it is dictionary). Empty key means key "_".
* PROP_CARET_VIEW: 3-tuple: shape of caret, normal mode. Tuple contents is (int_width, int_height, bool_empty_inside). Width/height<0 means value in percents.
* PROP_CARET_VIEW_OVR: 3-tuple: shape of caret, overwrite mode.
* PROP_CARET_VIEW_RO: 3-tuple: shape of caret, read-only mode.
* PROP_CARET_VIRTUAL: bool: caret position is allowed after line-ends.
* PROP_CARET_STOP_UNFOCUSED: bool: caret blinks only when editor is focused.
* PROP_MARKED_RANGE: line indexes of "marked range", value is 2-tuple (index1, index2) or (-1, -1) to remove this range.
* PROP_MINIMAP: bool: minimap is visible.
* PROP_MICROMAP: bool: micromap is visible.
* PROP_LINKS_SHOW: bool: enable hyperlinks underlining.
* PROP_LINKS_REGEX: str: regular expression for hyperlinks.
* PROP_LINKS_CLICKS: int: how mouse clicks activate hyperlinks. 0: disabled, 1: single click, 2: double click.
* PROP_TAB_SPACES: bool: tab-key inserts spaces.
* PROP_TAB_SIZE: int: size of tab-char.
* PROP_TAB_COLLECT_MARKERS: bool: tab-key collects (jumps to and deletes) markers.
* PROP_TAB_COLOR: int: color of tab containing editor; set COLOR_NONE to reset.
* PROP_TAB_TITLE: str: title of tab, useful for untitled tabs.
* PROP_TAB_ICON: int: index of tab icon, ie index in imagelist, which handle you can get via app_proc(PROC_GET_TAB_IMAGELIST).
* PROP_INDENT_SIZE: int: size of indent for Indent/Unindent commands. N>0: indent in spaces, N<0: indent in tabs, N=0: indent from PROP_TAB_SIZE/PROP_TAB_SPACES.
* PROP_INDENT_KEEP_ALIGN: bool: command Unindent keeps alignment of block edge, ie don't unindent if any line reached column 0.
* PROP_INDENT_AUTO: bool: makes next line also indented on Enter command.
* PROP_INDENT_KIND: int: kind of auto-indented line, see description of app option "indent_kind".
* PROP_FOLD_ALWAYS: bool: always show icons on "folding" gutter bar, otherwise show them only on mouse over.
* PROP_FOLD_ICONS: int: icon set for "folding" gutter bar. Possible values: 0, 1.
* PROP_FOLD_TOOLTIP_SHOW: bool: show floating tooltip when mouse hovers [...] mark for folded block.
* PROP_LAST_LINE_ON_TOP: bool: allow to scroll control, so last line shows on top.
* PROP_HILITE_CUR_COL: bool: highlight current column.
* PROP_HILITE_CUR_LINE: bool: highlight current line.
* PROP_HILITE_CUR_LINE_MINIMAL: bool: highlight current line, only minimal part of line.
* PROP_HILITE_CUR_LINE_IF_FOCUS: bool: highlight current line, only when editor focused.
* PROP_CODETREE: bool: enable standard code-tree filling.
* PROP_EDITORS_LINKED: bool: enable sharing of the same text by primary/secondary editors in file tab.
* PROP_ONE_LINE: bool: editor has single-line mode.
* PROP_MODERN_SCROLLBAR: bool: use custom-drawn themed scrollbars.
* PROP_SAVE_HISTORY: bool: allows to save history (caret, scroll pos, folding etc) on file closing.
* PROP_PREVIEW: bool: editor is inside "Preview tab" (it has italic caption). Currently only False value can be written.
* PROP_UNDO_GROUPED: bool: editor undo/redo is grouped.
* PROP_UNDO_LIMIT: int: max count of simple actions, which can be undone.
* PROP_UNDO_DATA: str: string representation of Undo data.
* PROP_REDO_DATA: str: string representation of Redo data.
* PROP_ZEBRA: int: if 0, "zebra" mode is not active; if >0, it is alpha-blend value (1..255) of active zebra mode.
* PROP_ZEBRA_STEP: int: step (in lines) of "zebra" mode.
* PROP_FONT: 2-tuple (str, int): normal font: name, size.
* PROP_FONT_B: 2-tuple (str, int): bold font: name, size. Empty name: not used.
* PROP_FONT_I: 2-tuple (str, int): italic font: name, size. Empty name: not used.
* PROP_FONT_BI: 2-tuple (str, int): bold+italic font: name, size. Empty name: not used.
* PROP_CODETREE_SUBLEXER: bool: enable code-tree to show nodes from sublexer(s) of current lexer.

* PROP_SPLIT: 2-tuple: information about primary/secondary editors splitting as tuple (split_kind, split_pos).
** "split_kind": str: "-" (not splitted), "v" (splitted vertically), "h" (splitted horizontally).
** "split_pos": int: part of entire size for secondary editor, multiplied by 1000 (e.g. 500 is half of size).

* PROP_SAVING_FORCE_FINAL_EOL: bool: on saving file, ensure the newline at end of document.
* PROP_SAVING_TRIM_SPACES: bool: on saving file, trim trailing whitespaces.
* PROP_SAVING_TRIM_FINAL_EMPTY_LINES: bool: on saving file, trim redundant empty lines at end of document.

* PROP_COMBO_ITEMS: only for "editor_combo" control, contents of drop-down list. On getting: list of str. On setting: value must be str with '\n'-separated items.

* PROP_V_MODE: int: mode of binary viewer. Setting it for viewer to one of VMODE_ values, changes viewer mode. Setting it for editor to one of VMODE_ values, switches editor to viewer in the given mode. Setting it for viewer to VMODE_NONE, switches viewer to editor.
* PROP_V_POS: int: for binary viewer, scroll position.
* PROP_V_SEL_START: int: for binary viewer, selection start offset.
* PROP_V_SEL_LEN: int: for binary viewer, selection length.
* PROP_V_WIDTH: int: for binary viewer, width of text in mode VMODE_BINARY.
* PROP_V_WIDTH_HEX: int: for binary viewer, width of text in mode VMODE_HEX.
* PROP_V_WIDTH_UHEX: int: for binary viewer, width of text in mode VMODE_UNICODE_HEX.

===================

</details>
<details><summary>* * <b>props vs options</b> &nbsp; &nbsp; [Editor class > Properties > props vs options]</summary>  

Many get_prop/set_prop ids correspond to CudaText options. List of correspoding pairs:

* PROP_CARET_VIRTUAL - caret_after_end
* PROP_GUTTER_ALL - gutter_show
* PROP_GUTTER_BM - gutter_bookmarks
* PROP_GUTTER_FOLD - gutter_fold
* PROP_HILITE_CUR_COL - show_cur_column
* PROP_HILITE_CUR_LINE - show_cur_line
* PROP_HILITE_CUR_LINE_MINIMAL - show_cur_line_minimal
* PROP_HILITE_CUR_LINE_IF_FOCUS - show_cur_line_only_focused
* PROP_INDENT_AUTO - indent_auto
* PROP_INDENT_KEEP_ALIGN - unindent_keeps_align
* PROP_INDENT_KIND - indent_kind
* PROP_INDENT_SIZE - indent_size
* PROP_LAST_LINE_ON_TOP - show_last_line_on_top
* PROP_MARGIN - margin
* PROP_MARGIN_STRING - margin_string
* PROP_MICROMAP - micromap_show
* PROP_MINIMAP - minimap_show
* PROP_RULER - ruler_show
* PROP_TAB_SIZE - tab_size
* PROP_TAB_SPACES - tab_spaces
* PROP_UNPRINTED_ENDS - unprinted_ends
* PROP_UNPRINTED_END_DETAILS - unprinted_end_details
* PROP_UNPRINTED_SHOW - unprinted_show
* PROP_UNPRINTED_SPACES - unprinted_spaces
* PROP_UNPRINTED_SPACES_TRAILING - unprinted_spaces_trailing
* PROP_WRAP - wrap_mode

===================

</details>
<details><summary>* * <b>Editor.bookmark</b> &nbsp; &nbsp; [Editor class > Properties > Editor.bookmark]</summary>  

bookmark(id, nline, nkind=1, ncolor=-1, text="", auto_del=True, show=True, tag=0)

Controls bookmarks. Possible values of "id":

* BOOKMARK_GET_ALL: Returns list of bookmarks, as list of dict. Param "nline" ignored. Dict keys are:
** "line": int
** "kind": int
** "tag": int
** "hint": str
** "auto_delx": enum, 0: don't auto-delete; 1: auto-delete; 2: auto-delete if global option is set
** "auto_del" (deprecated): bool, True if "auto_delx" is 1
** "show_in_list": bool
* BOOKMARK_GET_LIST: Returns list of line indexes, which have bookmarks. Param "nline" ignored.
* BOOKMARK_GET_PROP: Returns properties of bookmark at line index, given by param "nline". Returns dict, like for BOOKMARK_GET_ALL, or None.

* BOOKMARK_SET: Sets bookmark.
** Param "nline": int: line index of bookmark.
** Param "nkind": int: kind of bookmark: 1 means usual bookmark with usual color and icon. Other kind values mean custom bookmark, which must be setup via BOOKMARK_SETUP.
** Param "text": str: tooltip, shown when mouse moves over bookmark gutter icon.
** Param "auto_del": bool: auto delete bookmark on deleting its text line.
** Param "show": bool: show bookmark in "Go to bookmark" dialog.
** Param "tag": int: some value attached to bookmark.

* BOOKMARK_CLEAR: Removes bookmark from line, specified by "nline" param.
* BOOKMARK_CLEAR_ALL: Removes all bookmarks ("nline" ignored).
* BOOKMARK_DELETE_BY_TAG: Removes all bookmarks, which have tag, given by "tag" param.

* BOOKMARK_SETUP: Configures bookmarks with kind, given by "nkind" param.
** Param "ncolor": Color of bookmarked line.
*** Can be COLOR_NONE to not use background color.
*** Can be COLOR_DEFAULT to use current themed color.
** Param "text": Path to icon file for gutter, 16x16 .bmp or .png file. Empty str: don't show icon.

Additional internal list Bookmarks2 exists, which holds list of background bookmarks. They have lower paint priority than usual bookmarks, so background bookmark is painted only if usual bookmark not exists. Actions BOOKMARK2_* are used for Bookmarks2 list, actions have the same meaning. Background bookmarks don't have gutter icon and mouse-over tooltip. They share the single setup with usual bookmarks.

* BOOKMARK2_GET_ALL
* BOOKMARK2_GET_PROP
* BOOKMARK2_SET
* BOOKMARK2_CLEAR
* BOOKMARK2_CLEAR_ALL
* BOOKMARK2_DELETE_BY_TAG

Notes:

* Param "nkind" must be in the range 1..63.
* Param "nkind" values 2..9 have setup by default: they have blue icons "1" to "8".

===================

</details>
<details><summary>* * <b>Editor.decor</b> &nbsp; &nbsp; [Editor class > Properties > Editor.decor]</summary>  

decor(id, line=-1, tag=0, text="", color=0, bold=False, italic=False, image=-1, auto_del=True)

Controls decorations on gutter. It is used to show some visual marks, which look like bookmark icons, but independent of bookmarks. Possible values of "id":

* DECOR_GET_ALL: Returns list of all decorations, as list of dict, or None.
* DECOR_GET: Returns decoration for single line, as dict, or None. Params used: "line".
* DECOR_SET: Sets decoration for single line, overwriting existing item for that line. Returns bool: line index correct, item added. Params used:
** "line": int, line index.
** "tag": int, some value attached to item.
** "text": str, text to show; decoration will be text.
** "color": int, color of text decoration.
** "bold": bool, use bold font for text decoration.
** "italic": bool, use italic font for text decoration.
** "image": int, icon index from decoration image-list; decoration will be icon, if text is empty.
** "auto_del": bool, auto-delete decoration, when its line is deleted.

* DECOR_DELETE_BY_LINE: Deletes single decoration for given line. Params used: "line".
* DECOR_DELETE_BY_TAG: Deletes all decorations for given tag. Param used: "tag".
* DECOR_DELETE_ALL: Deletes all decorations.
* DECOR_GET_IMAGELIST: Returns int handle of decoration image-list (default size is 16x16).

===================

</details>
<details><summary>* * <b>Editor.folding</b> &nbsp; &nbsp; [Editor class > Properties > Editor.folding]</summary>  

folding(id, index=-1, item_x=-1, item_y=-1, item_y2=-1, item_staple=False, item_hint="")

Performs action on folding ranges. Possible values of "id":

* FOLDING_GET_LIST: Returns list of folding ranges. Params used: none except "id". Returns list of tuples (y, y2, x, staple, folded), or None.
** "y": int: line of range start.
** "y2": int: line of range end. If y==y2, then range is simple and don't have gutter-mark and staple.
** "x": int: x-offset of range start (char index in start line).
** "staple": bool: range has block staple.
** "folded": bool: range is currently folded.

* FOLDING_GET_LIST_FILTERED: Returns list of folding ranges, which contain given range: line indexes from param "item_y" to param "item_y2" (inclusive). If item_y<0, then no filtering is done. You can perform this filtering from the result of FOLDING_GET_LIST, but it's slower.

* FOLDING_FOLD: Folds range with given index (index in list, from FOLDING_GET_LIST). Params used: "index".
* FOLDING_FOLD_ALL: Folds all ranges.
* FOLDING_FOLD_LEVEL: Unfolds all ranges, then folds ranges starting with given nesting level. Params used: "index": nesting level, 0..9.

* FOLDING_UNFOLD: Unfolds range with given index (index in list, from FOLDING_GET_LIST). Params used: "index".
* FOLDING_UNFOLD_ALL: Unfolds all ranges.
* FOLDING_UNFOLD_LINE: Unfolds line with given index (index in lines list). Params used: "index".

* FOLDING_ADD: Adds folding range. Life time of this range is until lexer analisys runs (it clears ranges and adds ranges from lexer), which runs after any text change. Params used:
** "item_x": char index (offset in line) of range start.
** "item_y": line index of range start.
** "item_y2": line index of range end.
** "item_staple" (optional): bool, range has block staple.
** "item_hint" (optional): str, hint which is shown when range is folded.
** "index": if it's valid range index (0-based), range inserts at this index, otherwise range appends.

* FOLDING_DELETE: Deletes folding range with given index (index in list, from FOLDING_GET_LIST). Params used: "index".
* FOLDING_DELETE_ALL: Deletes all folding ranges. Params used: none except "id".
* FOLDING_FIND: Finds index of range (index in list, from FOLDING_GET_LIST). Returns None if not found. Params used: "item_y" is line index at which range begins.
* FOLDING_CHECK_RANGE_INSIDE: For 2 ranges, which indexes given by "index" and "item_x", detects: 1st range is inside 2nd range. Returns bool. Returns False if indexes incorrect.
* FOLDING_CHECK_RANGES_SAME: For 2 ranges, which indexes given by "index" and "item_x", detects: ranges are "equal" (x, y, y2 in ranges may differ). Returns bool. Returns False if indexes incorrect.

===================

</details>
<details><summary>* * <b>Editor.get_sublexer_ranges</b> &nbsp; &nbsp; [Editor class > Properties > Editor.get_sublexer_ranges]</summary>  

get_sublexer_ranges()

Returns list of ranges which belong to nested lexers (sublexers of current lexer for entire file, e.g. "JavaScript" inside "PHP", "CSS" inside "HTML"). Returns list of 5-tuples, or None if no ranges. Each tuple is:

* str: sublexer name
* int: start column (0-based)
* int: start line (0-based)
* int: end column
* int: end line

===================

</details>
<details><summary>* * <b>Editor.get_token</b> &nbsp; &nbsp; [Editor class > Properties > Editor.get_token]</summary>  

get_token(id, index1=0, index2=0)

Returns info about lexer "tokens". "Token" is minimal text fragment for lexer, e.g. one indentifier, one operator, one whole comment, one whole string. Each token has its color from lexer properties or color theme. Function returns None if no lexer is active in editor.

Possible values of "id":

* TOKEN_GET_KIND: For text position given by "index1" (column) and "index2" (line), returns kind of lexer token:
** "c": syntax comment
** "s": syntax string
** "a": any other kind

* TOKEN_LIST: Returns all tokens, as list of dict. Params "index1", "index2" are ignored. Returns None if no lexer active. Dict keys are:
** "x1": start column (0-based)
** "y1": start line (0-based)
** "x2": end column
** "y2": end line
** "str": text of token (can be multi-line with "\n")
** "style": lexer style of token (lexer dependent)
** "ki": index of token kind (0-based), in the list returned by lexer_proc()
** "ks": kind of syntax element: "c" - comment, "s" - string, "a" - any other

* TOKEN_LIST_SUB: Like TOKEN_LIST, but returns tokens only for given lines range, from "index1" to "index2" inclusive. Index in "index2" can be too big, this makes list until end of file.

===================

</details>
<details><summary>* * <b>Editor.get_wrapinfo</b> &nbsp; &nbsp; [Editor class > Properties > Editor.get_wrapinfo]</summary>  

get_wrapinfo()

Gets info about wrapped lines. It allows to calculate, at which positions long lines are wrapped (when wrap mode is on). It allows to see, which text parts are folded.

Returns list of dict, or None. Dict items has keys:

* "line": int: Original line index, for this part.
* "char": int: Char index (1-based, for UTF-16 stream of chars), at which this part starts. It is >1, for next parts of long wrapped line.
* "len": int: Length of this part. It equals to entire line length, if line is not wrapped.
* "indent": int: Screen indent in spaces, for this part, it's used in rendering when option "Show wrapped parts indented" is on.
* "final": int: State of this part. 0: final part of the entire line; 1: partially folded part; 2: first or middle part of the entire line.
* "initial": bool: True if part is initial for the entire (wrapped) line, False if it is continuation (ie, wrapped) part.

===================

</details>
<details><summary>* * <b>Editor.markers</b> &nbsp; &nbsp; [Editor class > Properties > Editor.markers]</summary>  

markers(id, x=0, y=0, tag=0, len_x=0, len_y=0, line_len=0)

Controls markers (used e.g. in Snippets plugin). Possible values of "id":

* MARKERS_GET: Returns list of markers. Each list item is [x, y, len_x, len_y, tag]. Returns None if no markers.

* MARKERS_GET_DICT: Returns list of markers. Each list item is dict with the following keys:
** "x", "y": Position of markers.
** "len_x", "len_y": Length of selection, which is applied when this marker is "collected". See description of MARKERS_ADD.
** "tag": Tag of marker.
** "line_len": Line-length of marker. If <0, line is painted to the left. If >0, line is painted to the right.
** "micromap": Marker is shown only on micromap.

* MARKERS_ADD: Adds marker. Also returns number of markers. Params:
** "x", "y": Position of marker (like caret position).
** "tag": Some int value attached to marker. Value>0 is needed if you want to place multi-carets when command "goto last marker (and delete)" runs. All markers with the same tag>0 will get multi-carets (after markers will be deleted).
** "len_x", "len_y": Selection length. It's applied when marker is "collected" by command "goto last marker".
*** if len_y==0: len_x is length of selection (single line),
*** if len_y>0: len_y is y-delta of selection-end, and len_x is absolute x-pos of selection-end.
** "line_len": Line length, in chars. If 0, not used. If <0, line is painted to the left. If >0, line is painted to the right.

* MARKERS_DELETE_ALL: Deletes all markers.
* MARKERS_DELETE_LAST: Deletes last added marker.
* MARKERS_DELETE_BY_TAG: Deletes all markers with the given tag. Param "tag": tag value.
* MARKERS_DELETE_BY_INDEX: Deletes marker by its index in the markers list. Param "tag": index (0-based).

===================

</details>
<details><summary>* * <b>Editor.attr</b> &nbsp; &nbsp; [Editor class > Properties > Editor.attr]</summary>  

attr(id, tag=0, x=0, y=0, len=1,
color_font=COLOR_NONE, color_bg=COLOR_NONE, color_border=COLOR_NONE,
font_bold=0, font_italic=0, font_strikeout=0,
border_left=0, border_right=0, border_down=0, border_up=0,
show_on_map=False, map_only=0 )

Controls additional color attributes. Possible values of "id":

* MARKERS_ADD: Adds fragment with specified properties. Also returns number of fragments. Parameters:
** "tag": Some int value attached to fragment (different plugins should add fragments with different tags).
** "x", "y": Position of fragment start (text position like caret).
** "len": Length of fragment in chars.
*** If len<0, and "show_on_map" is set, it will be multi-line micromap fragment with specified height=-len.
** "color_nnnn": Color values (RGB) for font, background, borders.
** "font_nnnn": Font attributes: 0 - off, 1 - on.
** "border_nnnn": Border types for edges, int value 0..6: none, solid, dash, solid 2pixel, dotted, rounded, wave.
** "show_on_map": How to show this fragment on micromap:
*** int value>=0: Show on micromap column with given tag. Don't pass value=0, because micromap column with tag=0 is reserved.
*** int value=-1: Don't show on micromap.
*** int value=-2: Show on micromap as full-width fragment (over full micromap width). Fragment color will be XOR'ed with colors of all micromap columns.
*** True: Show on micromap column 1.
*** False: Don't show on micromap.
** "map_only": int: 0: Show on text area only; 1: Show on micromap only; 2: Show both on text area and micromap.

* MARKERS_ADD_MANY: Adds one or multiple fragments at once. Also returns number of fragments. Parameters:
** "x", "y", "len": The same as for MARKERS_ADD, but can be also list/tuple of int. Lists/tuples must contain the equal number of elements (otherwise the minimal number of elements will be used).
** other parameters: The same as for MARKERS_ADD.

* MARKERS_GET: Returns list of fragments, or None. Each item is list of int, corresponding to Editor.attr() params (bool values will be 0 or 1).
* MARKERS_GET_DICT: Returns list of fragments. Each item is dict, with keys named like parameters of Editor.attr.

* MARKERS_DELETE_ALL: Deletes all fragments.
* MARKERS_DELETE_LAST: Deletes last added fragment.
* MARKERS_DELETE_BY_TAG: Deletes all fragments with the given tag. Param "tag": tag value.
* MARKERS_DELETE_BY_INDEX: Deletes fragment by its index in the list. Param "tag": index (0-based).

Notes:

* Fragment list is sorted by (x, y) pair.
* Duplicate fragments, ie with the same (x, y) pair, are allowed. Last added fragments go to the later indexes, and all fragments are painted in their order in the fragment list, so last added fragments are painted over previous ones.
* Param "color_bg" can be COLOR_NONE: it uses usual back-color.
* Params "color_font", "color_border" can be COLOR_NONE: it uses usual text-color.

===================

</details>

</details>

</details>

---
<details><summary><b>TreeHelpers</b></summary>  

CudaText contains built-in support for TreeHelpers. They build code-tree for additional lexers. TreeHelpers must be plugins, which have special install.inf file (Ini format).

Section "info":
* key "subdir" must begin with "cuda_tree_"

Section(s) "treehelper" followed by any string (e.g. "treehelper1"):
* key "lexers" is comma-separated lexers list
* key "method" is name of getter method in __init__.py

File __init__.py must contain getter method(s), referenced by install.inf. Getter has signature (with any name):

def get_headers(filename, lines)

* param "lines" is list of str, editor contents in CudaText.
* param "filename" is full file path, it is to support included files or something.

Getter must return list of tuples: (pos, level, caption, icon).

* field "pos": tuple of int (x1, y1, x2, y2): range for tree node.
* field "level": 1-based level of node, node of level K+1 is nested into (nearest upper) node of level K.
* field "caption": caption of node.
* field "icon": 0-based index of icon from standard image-list, or -1 if icon not used.
** 0: folder
** 1: parts1
** 2: parts2
** 3: parts3
** 4: box
** 5: func
** 6: arrow1
** 7: arrow2

Getter can also return value not of "list" type, in this case CudaText will not update the code-tree.

See examples of TreeHelpers: for Markdown, for MediaWiki. They are add-ons in the CudaText AddonManager.

</details>

---
<details><summary><b>Linters</b></summary>  

Linters are 2nd-level plugins for CudaLint plugin. CudaLint was ported from SublimeLinter 3, and its linters should be ported from SublimeLinter linters.
Here is example:

* Original linter: https://github.com/SublimeLinter/SublimeLinter-javac
* Ported linter: https://github.com/cudatext-addons/cuda_lint_javac

To see how to port, compare files "linter.py". Other files must be added like in other CudaLint linters.

</details>

---
<details><summary><b>Tech info</b></summary>  
<details><summary>&lt;descr&gt;</summary>  

</details>
<details><summary>* <b>Format of text for cmd_MouseClick</b> &nbsp; &nbsp; [Tech info > Format of text for cmd_MouseClick]</summary>  

Text is "X,Y" where X/Y are position of caret relative to top-caret (other carets removed when command runs). If Y is 0, X is addition to caret's x.
If Y not 0, Y is addition to caret's y, and X is absolute caret's x.

</details>
<details><summary>* <b>Finder options as string</b> &nbsp; &nbsp; [Tech info > Finder options as string]</summary>  

Some APIs allow to specify finder options as a string. Such string can have the following chars:

* "b": Backward search
* "c": Case sensitive
* "r": Regular expression
* "w": Whole words
* "f": Search from caret
* "s": Search in selection
* "o": Confirm replaces
* "a": Wrap search at edge of text
* "T" followed by a digit: Allowed syntax elements. Digit can be:
** "0": any
** "1": only comments
** "2": only strings
** "3": only comments/string
** "4": except comments
** "5": except strings
** "6": except comments/strings

</details>
<details><summary>* <b>Format of text for cmd_FinderAction</b> &nbsp; &nbsp; [Tech info > Format of text for cmd_FinderAction]</summary>  

Text is chr(1) separated items:

* item 0: Finder action, one of:
** "findfirst": Find first
** "findnext": Find next
** "findprev": Find previous
** "rep": Replace next, and find next
** "repstop": Replace next, and don't find
** "repall": Replace all
** "findcnt": Count all
** "findsel": Find all, make selections
** "findmark": Find all, place markers
* item 1: Text to find
* item 2: Text to replace with
* item 3: Options string, see [[#Finder_options_as_string]]

</details>

</details>

---
<details><summary><b>API of 3rd-party plugins</b></summary>  
<details><summary>* <b>Options Editor</b> &nbsp; &nbsp; [API of 3rd-party plugins > Options Editor]</summary>  
Options Editor plugin (preinstalled in CudaText) has API which allows to call its dialog with custom options. Example shows dialog with 4 custom options, with custom title. Options which user changed in dialog will be saved to user.json or lexer-specific config.

<syntaxhighlight lang="Python">
import os
import cudax_lib as appx
import cuda_options_editor as op_ed

fn_config = 'cuda_my_plugin.json' # without path!
meta_info = [
{
"opt": "my_opt_1",
"cmt": ["Comment line 1",
"Comment line 2",
"Comment line 3"],
"def": True,        # Default value
"frm": "bool",      # Value type from
#   bool float int str - simple types
#   font - font name
#   font-e - font name or empty
#   hotk - hotkey
#   file - file path
#   strs  - list of str
#   int2s - dict int to str
#   str2s - dict str to str
"chp": "MySection"  # Section (can be empty)
},
{   "opt": "my_opt_2",
"cmt": ["Comment"],
"def": "v1",
"frm": "strs",
"lst": ["v1", "v2"]
},
{   "opt": "my_opt_3",
"cmt": ["Comment"],
"def": 11,
"frm": "int2s",
"dct": [[11, "value for 11"], [22, "value for 22"]]
},
{   "opt": "my_opt_4",
"cmt": ["Comment"],
"def": "a",
"frm": "int2s",
"dct": [["a", "value for a"], ["b", "value for b"]]
}
]

class Command:
def get_opt(path, val):
return appx.get_opt(path, val, user_json=fn_config)

def config(self):
subset = '' # section in user.json, if user.json is used
title = 'My Plugin Options'
how = {'hide_lex_fil': True, 'stor_json': fn_config}
op_ed.OptEdD(
path_keys_info = meta_info,
subset = subset,
how = how
).show(title)
do_load_plugin_options() # some plugin function to reread options
</syntaxhighlight>

This will show:

[[File:cudatext_options_editor.png]]

Options Editor stores values in JSON with trailing comma, so to read these options, plugin must use cudax_lib.get_opt() instead of json.load().

</details>

</details>

---
<details><summary><b>Questions</b></summary>  
<details><summary>&lt;descr&gt;</summary>  

</details>
<details><summary>* <b>How plugin can fill code-tree</b> &nbsp; &nbsp; [Questions > How plugin can fill code-tree]</summary>  

* Handle events on_open, on_focus, on_change_slow (better don't use on_change, it slows down the app)
* Get handle of code-tree: h_tree = app_proc(PROC_GET_CODETREE, "")
* Fill tree via tree_proc(h_tree, ...)
** Disable standard tree filling via ed.set_prop(PROP_CODETREE, False)
** Clear tree via tree_proc(h_tree, TREE_ITEM_DELETE, id_item=0)
** On adding tree items, set range for them via tree_proc(h_tree, TREE_ITEM_SET_RANGE...)

</details>
<details><summary>* <b>How plugin can show tooltips on mouse-over</b> &nbsp; &nbsp; [Questions > How plugin can show tooltips on mouse-over]</summary>  

* Create dialog for tooltip, via dlg_proc()

* Handle events on_open, on_focus, on_change_slow (better don't use on_change, it will slow down the app)
** Find text regions which need tooltip
** Delete old hotspots via ed.hotspot(HOTSPOT_DELETE_BY_TAG...)
** Add hotspots via ed.hotspots(HOTSPOT_ADD...)

* Handle event on_hotspot
** Find which text region is under cursor
** Fill dialog via dlg_proc()
** Set dialog prop "p" to the handle of editor, ed_self.h. Don't use ed.h, because it is virtual handle 0.
** Set dialog pos (props "x", "y") to editor-related pixel coords. It is complex. See example: plugin HTML Tooltips.
** Show dialog via dlg_proc(..., DLG_SHOW_NONMODAL)
* On exiting hotspot, hide dialog

</details>

</details>

---
<details><summary><b>History</b></summary>  

1.0.167
* add: timer_proc.

1.0.168
* add: app_proc: PROC_SAVE_SESSION/ PROC_LOAD_SESSION get bool (session was saved/loaded)
* add: dlg_custom: "label" has prop to right-align
* add: ed.get_prop/set_prop: value can be int/bool too

1.0.169
* add: ed.insert does append, if y too big
* add: ed.delete can use too big y2

1.0.170
* add: app_proc: PROC_SET_CLIP_ALT

1.0.171
* add: app_proc: PROC_SIDEPANEL_ADD takes additional value icon_filename
* deprecated: app_log: LOG_PANEL_ADD
* deprecated: app_log: LOG_PANEL_DELETE
* deprecated: app_log: LOG_PANEL_FOCUS

1.0.172
* add: menu_proc()
* deprecated: app_proc actions: PROC_MENU_*
* change: PROP_ENC now uses short names, see [[CudaText#Encodings]]

1.0.173
* add: dlg_commands()
* add: toolbar_proc()
* deprecated: app_proc actions: PROC_TOOLBAR_*

1.0.174
* add: dlg_custom: "name=" (not required)
* add: dlg_custom: "font="
* add: dlg_custom: "type=filter_listbox", "type=filter_listview". To setup these filter controls, you must set "name=" for filter and its listbox/listview
* add: app_proc: PROC_ENUM_FONTS

1.0.175
* add: dlg_proc()
* add: dlg_custom: added props x= y= w= h= vis= color= font_name= font_size= font_color=
* add: timer_proc: can also use callbacks "module=nnn;cmd=nnn;" and "module=nnn;func=nnn;"
* add: timer_proc: added param "tag"
* add: menu_proc: actions MENU_CREATE, MENU_SHOW

1.0.176
* reworked dlg_proc, many form props+events added
* add: dlg_proc/dlg_custom: control type "treeview"
* add: dlg_proc/dlg_custom: control type "listbox_ex"
* add: dlg_proc/dlg_custom: control type "trackbar"
* add: dlg_proc/dlg_custom: control type "progressbar"
* add: dlg_proc/dlg_custom: control type "progressbar_ex"
* add: dlg_proc/dlg_custom: control type "bevel"
* add: file_open: added param "args"
* add: toolbar_proc: TOOLBAR_GET_CHECKED, TOOLBAR_SET_CHECKED
* delete: event on_dlg
* delete: dlg_proc/dlg_custom: prop "font" (use font_name, font_size, font_color)
* delete: deprecated APIs LOG_PANEL_ADD LOG_PANEL_DELETE LOG_PANEL_FOCUS
* delete: deprecated APIs PROC_TOOLBAR_*

1.0.177
* add: ed.replace
* add: ed.replace_lines
* add: dlg_custom: parameter "get_dict" to get new dict result (not tuple)
* add: dlg_proc: action DLG_SCALE
* add: app_proc: action PROC_GET_SYSTEM_PPI

1.0.178
* add: dlg_proc: control props for anchors/spacing: "a_*", "sp_*"
* add: dlg_proc: param "name" to specify controls by name
* add: dlg_proc: form prop "color"
* add: dlg_proc: form prop "autosize"
* add: dlg_proc: actions DLG_DOCK, DLG_UNDOCK
* add: dlg_custom/dlg_proc: control type "paintbox"

1.0.179
* add: dlg_proc/timer_proc/menu_proc: callbacks can be "callable", ie function names
* add: dlg_proc/timer_proc: callbacks can be with "info=...;" at end
* add: menu_proc: can use new-style callbacks like in dlg_proc
* deprecated: menu_proc old-style callbacks "module,method[,param]"

1.0.180
* add: app_proc: PROC_SHOW_SIDEBAR_GET, PROC_SHOW_SIDEBAR_SET
* add: app_proc: can pass not only string value
* add: ed.get_prop: PROP_COORDS
* add: dlg_proc/dlg_custom: type "panel", type "group"
* add: dlg_proc: control prop "p" (parent)

1.0.181
* add: dlg_proc/dlg_custom: type "pages"
* add: dlg_proc: control "paintbox" has sub-event "on_click" with info="x,y"

1.0.182
* add: dlg_proc: type "toolbar"
* add: app_proc: PROC_PROGRESSBAR
* add: app_proc: PROC_SPLITTER_GET, PROC_SPLITTER_SET
* deprecated: app_proc: PROC_GET_SPLIT, PROC_SET_SPLIT
* add: app_log: LOG_CONSOLE_GET_COMBO_LINES
* add: app_log: LOG_CONSOLE_GET_MEMO_LINES
* add: app_log: LOG_GET_LINES_LIST
* deprecated: app_log: LOG_CONSOLE_GET, LOG_CONSOLE_GET_LOG, LOG_GET_LINES

1.0.183
* big changes in dlg_proc:
** deleted: parameter 'id_event'
** deleted: props 'callback', 'events'
** form events is not called for controls (forms have own events, controls have own events)
** add: events for forms: 'on_resize', 'on_close', 'on_close_query', 'on_key_down', 'on_key_up'
** add: events for controls: 'on_change', 'on_click', 'on_click_dbl', 'on_select', 'on_menu'
** add: events for 'treeview' control: 'on_fold', 'on_unfold'

1.0.184
* deprecated: app_proc: PROC_SIDEPANEL_ADD, PROC_BOTTOMPANEL_ADD
* deprecated: on_panel event
* add: app_proc: PROC_SIDEPANEL_ADD_DIALOG, PROC_BOTTOMPANEL_ADD_DIALOG
* add: tree_proc: TREE_ITEM_FOLD_LEVEL
* add: tree_proc: TREE_THEME
* add: listbox_proc: LISTBOX_THEME
* add: toolbar_proc: TOOLBAR_THEME
* add: toolbar_proc: used new callback form
* add: menu_proc: MENU_SHOW with command="" to show at cursor
* add: menu_proc: added param "hotkey" for MENU_ADD, "hotkey" returned from MENU_ENUM
* deleted deprecated: PROC_GET_SPLIT, PROC_SET_SPLIT
* deleted deprecated: LOG_GET_LINES
* deleted deprecated: LOG_CONSOLE_GET, LOG_CONSOLE_GET_LOG

1.0.185 (app 1.11.0)
* add: menu_proc: for MENU_ADD added param "tag"
* add: menu_proc: MENU_SHOW can use 2-tuple (x,y)
* add: menu_proc: MENU_ENUM gives additional dict key "command"
* deprecated: menu_proc command values: "recents", "enc", "langs", "lexers", "plugins", "themes-ui", "themes-syntax"

1.0.186 (app 1.12.0)
* add: tree_proc: TREE_ICON_GET_SIZES, TREE_ICON_SET_SIZES
* deleted deprecated: app_proc: PROC_MENU_* actions
* deleted deprecated: app_proc: PROC_SIDEPANEL_ADD, PROC_BOTTOMPANEL_ADD

1.0.187 (app 1.12.2)
* add: lexer_proc: LEXER_GET_PROP
* deprecated: lexer_proc: LEXER_GET_EXT, LEXER_GET_ENABLED, LEXER_GET_COMMENT, LEXER_GET_COMMENT_STREAM, LEXER_GET_COMMENT_LINED, LEXER_GET_LINKS, LEXER_GET_STYLES, LEXER_GET_STYLES_COMMENTS, LEXER_GET_STYLES_STRINGS
* deleted: lexer_proc: actions didnt work, lexer files were not saved: LEXER_SET_*, LEXER_DELETE, LEXER_IMPORT
* add: dlg_proc: control "listview" on_select returns "data" param with 2-tuple

1.0.188 (app 1.13.0)
* add: imagelist_proc()
* add: tree_proc: TREE_GET_IMAGELIST
* add: toolbar_proc: TOOLBAR_GET_IMAGELIST
* add: lexer_proc: LEXER_GET_LEXERS
* deprecated: tree_proc: TREE_ICON_ADD, TREE_ICON_DELETE, TREE_ICON_GET_SIZES, TREE_ICON_SET_SIZES
* deprecated: toolbar_proc: TOOLBAR_GET_ICON_SIZES, TOOLBAR_SET_ICON_SIZES, TOOLBAR_ADD_ICON
* deprecated: lexer_proc: LEXER_GET_LIST

1.0.189 (app 1.13.1)
* add: tree_proc: TREE_ITEM_SHOW

1.0.190 (app 1.14.0)
* add: app_proc: PROC_GET_TAB_IMAGELIST
* add: ed.get_prop/ed.set_prop: PROP_TAB_ICON
* add: imagelist_proc: IMAGELIST_PAINT

1.0.191 (app 1.14.2)
* add: dlg_proc: form property "border"
* add: dlg_proc: control type "button_ex"
* add: button_proc() to work with "button_ex"

1.0.192 (app 1.14.8)
* add: dlg_proc: control type "splitter"
* add: dlg_proc: control prop "align"
* add: dlg_proc: control "listbox_ex" has event "on_draw_item"
* add: listbox_proc: actions LISTBOX_GET_ITEM_H, LISTBOX_SET_ITEM_H
* add: listbox_proc: actions LISTBOX_GET_DRAWN, LISTBOX_SET_DRAWN
* add: event on_paste
* deleted deprecated: event on_panel
* deleted deprecated: menu_proc spec strings: "recents", "enc", "langs", "lexers", "plugins", "themes-ui", "themes-syntax"

1.0.193 (app 1.15.0)
* add: image_proc()
* deleted: canvas_proc actions: CANVAS_IMAGE, CANVAS_IMAGE_SIZED (use image_proc instead)
* add: dlg_menu can have argument of type list/tuple
* add: dlg_menu can have argument "caption"

1.0.194 (app 1.15.4)
* add: app_proc: PROC_SIDEPANEL_ACTIVATE has 2nd param
* add: menu_proc: MENU_GET_PROP
* add: menu_proc: MENU_SET_CAPTION
* add: menu_proc: MENU_SET_VISIBLE
* add: menu_proc: MENU_SET_ENABLED
* add: menu_proc: MENU_SET_CHECKED
* add: menu_proc: MENU_SET_RADIOITEM
* add: menu_proc: MENU_SET_HOTKEY
* add: tree_proc: TREE_ITEM_GET_PROPS
* deprecated: tree_proc: TREE_ITEM_GET_PROP, TREE_ITEM_GET_PARENT

1.0.195 (app 1.15.5)
* add: dlg_proc: control prop "border"
* add: dlg_proc: control "linklabel" uses "on_click" (after opening URL if it's valid)

1.0.196 (app 1.16.2)
* add: ed.dim()

1.0.197 (app 1.17.2)
* add: dlg_proc: type "editor"
* add: object ed_goto
* add: objects ed_con_log, ed_con_in
* add: events: on_goto_enter, on_goto_change, on_goto_caret, on_goto_key, on_goto_key_up
* add: ed.get_prop/set_prop: PROP_ONE_LINE
* add: ed.get_prop/set_prop: PROP_LINE_NUMBERS

1.0.198 (app 1.18.0)
* add: constants WRAP_nnn
* add: ed.set_prop works for PROP_LINE_STATE

1.0.199 (app 1.19.2)
* add: toolbar_proc: TOOLBAR_GET_VERTICAL, TOOLBAR_SET_VERTICAL
* add: toolbar_proc: TOOLBAR_GET_WRAP, TOOLBAR_SET_WRAP
* deleted deprecated: TOOLBAR_GET_ICON_SIZES, TOOLBAR_SET_ICON_SIZES, TOOLBAR_ADD_ICON
* deleted deprecated: TREE_ITEM_GET_PROP, TREE_ITEM_GET_PARENT
* deleted deprecated: TREE_ICON_ADD, TREE_ICON_DELETE, TREE_ICON_GET_SIZES, TREE_ICON_SET_SIZES
* deleted deprecated: LEXER_GET_LIST, LEXER_GET_EXT, LEXER_GET_ENABLED, LEXER_GET_COMMENT, LEXER_GET_COMMENT_STREAM, LEXER_GET_COMMENT_LINED, LEXER_GET_LINKS, LEXER_GET_STYLES, LEXER_GET_STYLES_COMMENTS, LEXER_GET_STYLES_STRINGS

1.0.200 (app 1.20.2)
* add: event on_insert

1.0.201 (app 1.21.0)
* add: several commands in cudatext_cmd.py: simple word jump, goto abs line begin/end

1.0.202 (app 1.23.0)
* add: file_open: can open preview-tab
* add: file_open: can open file ignoring history
* change: renamed file_open param "args" to "options"

1.0.203 (app 1.23.4)
* add: event on_scroll
* add: event on_group
* add: file_open: options can have "/noevent"

1.0.204 (app 1.23.5)
* add: ed.get_wrapinfo()
* add: ed.get_prop/ed.set_prop: PROP_SCROLL_VERT, PROP_SCROLL_HORZ
* add: ed.export_html()
* deleted: ed.set_prop: PROP_EXPORT_HTML

1.0.205 (app 1.24.1)
* add: dlg_proc: ctl "edit" supports "on_change" (if "act":True)

1.0.206 (app 1.24.2)
* add: dlg_proc: added ctl property "focused"
* add: dlg_proc: added form events "on_act", "on_deact"

1.0.207 (app 1.25.1)
* add: app_proc: PROC_GET_COMMANDS

1.0.208 (app 1.26.2)
* add: lexer_proc supports lite lexers too (their names have " ^" suffix)

1.0.208 (app 1.27.2)
* add: menu_proc: added MENU_SET_IMAGELIST, MENU_SET_IMAGEINDEX
* add: file_open: for zip files, returns True if zip installation completed

1.0.210 (app 1.29.0)
* add: file_open: for binary viewer, pass options='/binary'
* add: ed.get_prop/set_prop: PROP_KIND, PROP_V_MODE, PROP_V_POS, PROP_V_SEL_START, PROP_V_SEL_LEN

1.0.211 (app 1.32.0)
* add: statusbar_proc()
* add: dlg_proc: added type "statusbar"
* add: toolbar_proc: TOOLBAR_UPDATE

1.0.212 (app 1.32.2)
* add: toolbar_proc: TOOLBAR_GET_COUNT
* add: toolbar_proc: TOOLBAR_GET_BUTTON_HANDLE
* add: button_proc: BTN_GET_TEXT, BTN_SET_TEXT
* add: button_proc: BTN_GET_ENABLED, BTN_SET_ENABLED
* add: button_proc: BTN_GET_VISIBLE, BTN_SET_VISIBLE
* add: button_proc: BTN_GET_HINT, BTN_SET_HINT
* add: button_proc: BTN_GET_DATA1, BTN_SET_DATA1, BTN_GET_DATA2, BTN_SET_DATA2
* change: toolbar_proc: TOOLBAR_ENUM returns int value of "kind", before it was str
* deprecated: TOOLBAR_GET_CHECKED, TOOLBAR_SET_CHECKED - now use TOOLBAR_GET_BUTTON_HANDLE + button_proc()
* deprecated: PROC_GET_COMMAND, PROC_GET_COMMAND_INITIAL, PROC_GET_COMMAND_PLUGIN - now use PROC_GET_COMMANDS instead

1.0.213 (app 1.32.3)
* add: button_proc: BTN_GET_MENU, BTN_SET_MENU
* add: button_proc: BTN_GET_ARROW, BTN_SET_ARROW
* add: button_proc: BTN_GET_FOCUSABLE, BTN_SET_FOCUSABLE
* add: button_proc: BTN_GET_FLAT, BTN_SET_FLAT
* add: toolbar_proc: TOOLBAR_ADD_ITEM, TOOLBAR_ADD_MENU
* deprecated: toolbar_proc: TOOLBAR_ADD_BUTTON - now use TOOLBAR_ADD_ITEM/TOOLBAR_ADD_MENU + button_proc()
* deprecated: toolbar_proc: TOOLBAR_SET_BUTTON - now use TOOLBAR_GET_BUTTON_HANDLE + button_proc()
* deprecated: toolbar_proc: TOOLBAR_ENUM - now use TOOLBAR_GET_COUNT + TOOLBAR_GET_BUTTON_HANDLE + button_proc()

1.0.214 (app 1.32.4)
* add: button_proc supports callable param "value"

1.0.215 (app 1.34.1)
* add: dlg_proc: DLG_CTL_PROP_GET returns key "items" for controls "listview", "checklistview", "listbox", "checklistbox"

1.0.216 (app 1.34.2)
* add: statusbar_proc: STATUSBAR_GET_CELL_AUTOSIZE, STATUSBAR_SET_CELL_AUTOSIZE
* add: statusbar_proc: STATUSBAR_GET_CELL_AUTOSTRETCH, STATUSBAR_SET_CELL_AUTOSTRETCH
* add: statusbar_proc: 10 actions to get/set colors, STATUSBAR_GET/SET_COLOR_*
* deleted: STATUSBAR_GET_COLOR_BORDER, STATUSBAR_SET_COLOR_BORDER
* deleted: STATUSBAR_AUTOSIZE_CELL - now use STATUSBAR_SET_CELL_AUTOSTRETCH

1.0.217 (app 1.34.4)
* add: button_proc: const BTNKIND_TEXT_CHOICE
* add: button_proc: BTN_GET_WIDTH, BTN_SET_WIDTH
* add: button_proc: BTN_GET_ITEMS, BTN_SET_ITEMS
* add: button_proc: BTN_GET_ITEMINDEX, BTN_SET_ITEMINDEX
* add: button_proc: BTN_GET_ARROW_ALIGN, BTN_SET_ARROW_ALIGN
* add: button_proc: BTN_UPDATE

1.0.218 (app 1.38.0)
* add: Editor.hotspots()
* add: event on_hotspot
* add: app_proc: PROC_GET_MOUSE_POS
* add: dlg_proc: DLG_PROP_SET supports prop "p" (parent of form)
* add: Editor.convert: CONVERT_SCREEN_TO_LOCAL, CONVERT_LOCAL_TO_SCREEN
* add: Editor.convert: CONVERT_PIXELS_TO_CARET, CONVERT_CARET_TO_PIXELS
* add: Editor.markers: MARKERS_DELETE_BY_TAG
* add: Editor.get_prop: PROP_CELL_SIZE

1.0.219 (app 1.38.1)
* add: app_proc: PROC_THEME_UI_DATA_GET
* add: app_proc: PROC_THEME_SYNTAX_DATA_GET

1.0.220 (app 1.38.2)
* add: on_state event aslo called with new constants APPSTATE_nnnn

1.0.221 (app 1.38.3)
* add: Editor.get_prop/set_prop: PROP_INDENT_SIZE, PROP_INDENT_KEEP_ALIGN, PROP_INDENT_AUTO, PROP_INDENT_KIND

1.0.222 (app 1.38.5)
* add: dlg_proc: form events "on_mouse_enter", "on_mouse_exit"
* add: dlg_proc: form events "on_show", "on_hide"
* add: dlg_proc: control events "on_mouse_enter", "on_mouse_exit", "on_mouse_down", "on_mouse_up"

1.0.223 (app 1.39.1)
* add: install.inf supports param [info] os=, value is comma separated strings: win, linux, macos
* add: menu_proc: support for spec menu id "_oplugins"

1.0.224 (app 1.39.5)
* add: dlg_proc: control prop "ex"
* deprecated: dlg_proc/dlg_custom prop "props" - now use "ex0"..."ex9"
* deprecated: BOOKMARK_CLEAR_HINTS

1.0.225 (app 1.40.0)
* add: app_proc: PROC_SEND_MESSAGE

1.0.226 (app 1.40.1)
* add: app_proc: PROC_GET_CODETREE
* add: Editor.get_prop/set_prop: PROP_CODETREE
* add: tree_proc: TREE_ITEM_GET_RANGE, TREE_ITEM_SET_RANGE
* deprecated: tree_proc: TREE_ITEM_GET_SYNTAX_RANGE
* change: changed Code-Tree button caption from "Tree" to "Code tree"

1.0.227 (app 1.40.2)
* add: file_open: added options "/passive", "/nonear"

1.0.228 (app 1.40.7)
* add: event on_tab_change

1.0.229 (app 1.43.0)
* add: plugins can force show sidebar button, even if they don't run on start. New install.inf sections [sidebar1] .. [sidebar3]. Examples: ProjectManager, TabsList.

1.0.230 (app 1.44.0)
* add: dlg_proc: control "listview"/"checklistview" has event "on_click_header"
* add: install.inf: allow os= values with bitness: win32, win64, linux32, linux64, freebsd32, freebsd64
* deleted deprecated: PROC_GET_COMMAND
* deleted deprecated: PROC_GET_COMMAND_INITIAL
* deleted deprecated: PROC_GET_COMMAND_PLUGIN

1.0.231 (app 1.45.5)
* add: dlg_proc: control "listview" has prop "columns"
* removed: on_group; on_state(..., APPSTATE_GROUPS) called instead

1.0.232 (app 1.46.2)
* add: dlg_proc: control "radiogroup" has prop "columns"
* add: dlg_proc: control "radiogroup" has prop "ex0"
* add: Editor.get_prop/set_prop: PROP_GUTTER_ALL, PROP_GUTTER_STATES

1.0.233 (app 1.47.0)
* add: dlg_menu: option MENU_NO_FUZZY
* add: dlg_menu: option MENU_NO_FULLFILTER

1.0.233 (app 1.47.1)
* add: app_proc: PROC_GET_GUI_HEIGHT supports name "scrollbar"

1.0.234 (app 1.47.5)
* add: event on_exit
* add: ed.get_prop/set_prop: PROP_UNPRINTED_SPACES_TRAILING
* add: ed.get_prop/set_prop: PROP_LAST_LINE_ON_TOP
* add: ed.get_prop/set_prop: PROP_HILITE_CUR_COL
* add: ed.get_prop/set_prop: PROP_HILITE_CUR_LINE
* add: ed.get_prop/set_prop: PROP_HILITE_CUR_LINE_MINIMAL
* add: ed.get_prop/set_prop: PROP_HILITE_CUR_LINE_IF_FOCUS
* add: ed.get_prop/set_prop: PROP_MODERN_SCROLLBAR

1.0.235 (app 1.48.2)
* add: support for "lazy events"
* add: event on_console_print
* add: lexer_proc: LEXER_REREAD_LIB

1.0.236 (app 1.48.3)
* add: dlg_proc: controls have props "w_min/w_max/h_min/h_max", like forms
* fix: bug in dlg_proc/dlg_custom: controls with changed parent become not findable by name/index (and count of controls becomes smaller)

1.0.237 (app 1.49.1)
* add: statusbar_proc: STATUSBAR_GET_CELL_HINT, STATUSBAR_SET_CELL_HINT

1.0.238 (app 1.49.5)
* add: dlg_proc/dlg_custom: controls have prop "font_style"
* add: file_open: options "/view-text", "/view-binary", "/view-hex", "/view-unicode"
* change: file_open: deleted option "/binary"

1.0.239 (app 1.51.2)
* add: event on_mouse_stop

1.0.240 (app 1.52.1)
* add: ed.bookmark: BOOKMARK_SET can specify "delete bookmark on deleting line"
* deleted deprecated: BOOKMARK_CLEAR_HINTS
* deleted deprecated: TREE_ITEM_GET_SYNTAX_RANGE

1.0.242 (app 1.53.6)
* add/change: dlg_proc: form property "border" is now enum, with DBORDER_nnn values
* deprecated: dlg_proc: form property "resize"
* deleted deprecated: TOOLBAR_SET_BUTTON, TOOLBAR_ADD_BUTTON, TOOLBAR_ENUM, TOOLBAR_GET_CHECKED, TOOLBAR_SET_CHECKED

1.0.243 (app 1.55.0)
* add: event on_click_gutter
* deleted: events on_goto_enter, on_goto_change, on_goto_caret, on_goto_key, on_goto_key_up
* add: additional group indexes 6..8 for floating groups
* add: app_proc: PROC_SHOW_FLOATGROUP1_GET, PROC_SHOW_FLOATGROUP1_SET
* add: app_proc: PROC_SHOW_FLOATGROUP2_GET, PROC_SHOW_FLOATGROUP2_SET
* add: app_proc: PROC_SHOW_FLOATGROUP3_GET, PROC_SHOW_FLOATGROUP3_SET
* add: app_proc: PROC_FLOAT_SIDE_GET, PROC_FLOAT_SIDE_SET
* add: app_proc: PROC_FLOAT_BOTTOM_GET, PROC_FLOAT_BOTTOM_SET

1.0.244 (app 1.55.1)
* deleted: object ed_goto

1.0.245 (app 1.55.3)
* add: Editor.get_prop/set_prop: PROP_CARET_STOP_UNFOCUSED
* add: addon can require other addon(s) via install.inf: [info] req=cuda_aa,cuda_bb

1.0.246 (app 1.56.1)
* add: dlg_proc: controls have prop "autosize"
* add: dlg_proc: added events for control "editor": on_change, on_caret, on_scroll, on_key_down, on_key_up, on_click_gutter, on_click_gap, on_paste
* add: addon can require lexer(s) via install.inf: [info] reqlexer=Name1,Name2

1.0.247 (app 1.56.3)
* add: event on_close_pre
* add: ed.get_prop/set_prop: PROP_FOLD_ALWAYS
* add: ed.get_prop/set_prop: PROP_FOLD_ICONS
* add: ed.get_prop/set_prop: PROP_FOLD_TOOLTIP_SHOW

1.0.248 (app 1.56.6)
* add: app_proc: PROC_GET_FINDER_PROP
* add: app_proc: PROC_SET_FINDER_PROP
* deprecated: app_proc: PROC_GET_FIND_OPTIONS
* deprecated: app_proc: PROC_SET_FIND_OPTIONS
* deprecated: app_proc: PROC_GET_FIND_STRINGS

1.0.249 (app 1.57.5)
* add: dlg_proc: control "pages" supports prop "val"
* add: dlg_proc: control event "on_menu" can return False to disable default menu
* add: Editor.get_token: TOKEN_LIST, TOKEN_LIST_SUB
* add: Editor.get_token: index params are optional now
* deprecated: Editor.get_token: TOKEN_AT_POS, TOKEN_INDEX (now use TOKEN_LIST, TOKEN_LIST_SUB)

1.0.250 (app 1.57.7)
* add: event "on_open_none"
* add: dlg_proc: event "on_menu" has filled param "data" like in "on_mouse_down"

1.0.251 (app 1.57.8)
* add: Editor.get_prop: PROP_ACTIVATION_TIME

1.0.252 (app 1.58.1)
* add: app_log: added optional param "panel"
* add: Editor.bookmark: added 3 optional params: "auto_del", "show", "tag"
* add: Editor.bookmark: BOOKMARK_GET_ALL
* add: Editor.bookmark: BOOKMARK_GET_PROP
* add: Editor.bookmark: BOOKMARK_DELETE_BY_TAG
* change: Editor.bookmark: BOOKMARK_SET sets flag "delete on deleting line" from new param "auto_del" now (before: from "ncolor=1")
* change: on_change/on_change_slow now isn't called on file opening
* deprecated: app_log: LOG_SET_PANEL (now use parameter "panel")
* deprecated: Editor.bookmark: BOOKMARK_GET (now use BOOKMARK_GET_PROP)

1.0.253 (app 1.59.0)
* add: Editor.attr: added optional param "show_on_map"
* add: Editor.get_prop(PROP_ACTIVATION_TIME) is 0 for not yet focused tabs
* add: app_proc: PROC_BOTTOMPANEL_ACTIVATE now handles tuple like PROC_SIDEPANEL_ACTIVATE

1.0.254 (app 1.60.1)
* add: event on_tab_switch

1.0.255 (app 1.60.5)
* add: module cudax_nodejs
* add: menu_proc: support menu id "tab"
* add: menu_proc: MENU_REMOVE
* add: Editor.folding: FOLDING_FOLD_ALL
* add: Editor.folding: FOLDING_FOLD_LEVEL
* add: Editor.folding: FOLDING_UNFOLD_ALL
* add: Editor.folding: FOLDING_UNFOLD_LINE

1.0.256 (app 1.60.7)
* add: Editor.decor()

1.0.257 (app 1.62.0)
* deleted deprecated: TOKEN_AT_POS, TOKEN_INDEX (now use TOKEN_LIST, TOKEN_LIST_SUB)
* deleted deprecated: BOOKMARK_GET (now use BOOKMARK_GET_PROP)
* deleted deprecated: LOG_SET_PANEL (now use parameter "panel")

1.0.258 (app 1.62.5)
* deleted deprecated: PROC_GET_FIND_OPTIONS, PROC_GET_FIND_STRINGS (now use PROC_GET_FINDER_PROP)
* deleted deprecated: PROC_SET_FIND_OPTIONS (now use PROC_SET_FINDER_PROP)

1.0.259 (app 1.63.5)
* add: Editor.get_prop/set_prop: PROP_SAVE_HISTORY
* add: Editor.get_prop/set_prop: PROP_PREVIEW

1.0.260 (app 1.63.6)
* deleted event "on_console"
* deleted event "on_console_print"

1.0.261 (app 1.64.2)
* add: event on_goto_enter
* add: Editor.get_prop/set_prop: PROP_CARET_VIEW, PROP_CARET_VIEW_OVR, PROP_CARET_VIEW_RO
* deleted: Editor.get_prop/set_prop: PROP_CARET_SHAPE, PROP_CARET_SHAPE_OVR, PROP_CARET_SHAPE_RO

1.0.262 (app 1.64.4)
* add: ini_proc()
* add: app_proc: PROC_GET_KEYSTATE returns also state of pressed mouse buttons

1.0.263 (app 1.65.6)
* add: event on_lexer_parsed

1.0.264 (app 1.66.7)
* add: app_proc: PROC_SHOW_TREEFILTER_GET, _SET
* add: Editor.get_prop/set_prop: PROP_UNDO_GROUPED
* add: Editor.get_prop/set_prop: PROP_UNDO_LIMIT

1.0.265 (app 1.66.9)
* add: Editor.get_prop/set_prop: PROP_UNDO_DATA
* add: Editor.get_prop/set_prop: PROP_REDO_DATA
* add: Editor.folding: FOLDING_GET_LIST_FILTERED

1.0.266 (app 1.67.0)
* add: dlg_commands: add COMMANDS_CENTERED
* add: dlg_commands: add "title" param

1.0.267 (app 1.70.3)
* add: on_state: add APPSTATE_CONFIG_REREAD
* add: on_state: add APPSTATE_SESSION_LOAD

1.0.268 (app 1.71.5)
* add: event on_save_naming

1.0.269 (app 1.71.6)
* add: Editor.gap: allowed gap before the first line (line index = -1)
* add: file_open: options "/nontext-view-text", "/nontext-view-binary", "/nontext-view-hex", "/nontext-view-unicode", "/nontext-cancel"
* add: file_open: options "/nozip", "/nopictures"

1.0.270 (app 1.72.1)
* add: app_proc: PROC_WINDOW_TOPMOST_GET, PROC_WINDOW_TOPMOST_SET

1.0.271 (app 1.73.1)
* add: Editor.bookmark: added actions BOOKMARK2_*

1.0.272 (app 1.74.0)
* add: file_open: can open 2 files in a single tab
* add: Editor.get_prop/set_prop: PROP_EDITORS_LINKED
* add: Editor.get_prop/set_prop: PROP_SPLIT
* add: Editor.get_prop: PROP_HANDLE_SELF, PROP_HANDLE_PRIMARY, PROP_HANDLE_SECONDARY
* add: Editor.get_prop: PROP_RECT_CLIENT, PROP_RECT_TEXT
* change: deleted object "ed_bro"
* deprecated: file_save(); use ed.save() instead
* deprecated: Editor.get_split, Editor.set_split; use PROP_SPLIT instead

1.0.273 (app 1.74.1)
* add: Editor.gap: added params "size", "color"
* add: app_proc: PROC_SIDEPANEL_GET
* add: app_proc: PROC_BOTTOMPANEL_GET

1.0.274 (app 1.74.7)
* deleted deprecated: file_save()
* deleted deprecated: Editor.get_split, Editor.set_split
* add: Editor.gap: no limit for gap height

1.0.275 (app 1.76.0)
* add: Editor.get_text_line has optional param "max_len"
* add: dlg_menu: added MENU_CENTERED

1.0.276 (app 1.76.6)
* add: dlg_proc: form has property "p" (handle of parent form)

1.0.277 (app 1.77.2)
* change: dlg_proc: "memo" value must encode tab as chr(3) now (was chr(2))
* add: get_prop: PROP_CODETREE_MODIFIED_VERSION

1.0.278 (app 1.77.4)
* add: Editor.get_prop/set_prop: PROP_SAVING_FORCE_FINAL_EOL
* add: Editor.get_prop/set_prop: PROP_SAVING_TRIM_SPACES
* add: Editor.get_prop/set_prop: PROP_SAVING_TRIM_FINAL_EMPTY_LINES

1.0.279 (app 1.77.6)
* add: app_proc: PROC_CONFIG_READ
* add: app_proc: PROC_CONFIG_NEWDOC_EOL_GET
* add: app_proc: PROC_CONFIG_NEWDOC_EOL_SET
* add: app_proc: PROC_CONFIG_NEWDOC_ENC_GET
* add: app_proc: PROC_CONFIG_NEWDOC_ENC_SET

1.0.280 (app 1.77.6)
* add: Editor.get_prop/set_prop: PROP_NEWLINE
* deprecated: PROP_EOL

1.0.281 (app 1.78.5)
* add: Editor.get_prop/set_prop: PROP_SCROLL_VERT_SMOOTH, PROP_SCROLL_HORZ_SMOOTH
* add: Editor.get_prop: PROP_SCROLL_VERT_INFO, PROP_SCROLL_HORZ_INFO
* add: Editor.get_wrapinfo: dict result has additional "initial" field
* deprecated: PROP_COLUMN_LEFT; use PROP_SCROLL_HORZ instead

1.0.282 (app 1.78.6)
* add: Editor.get_prop: PROP_LINE_STATES
* add: Editor.get_prop: PROP_LINE_STATES_UPDATED
* change: PROP_TAG is now stored per Editor object, so primary and secondary editors (in the same tab) have different PROP_TAG values
* add: TreeHelper getter can return not "list" type, to not refresh code tree

1.0.283 (app 1.78.7)
* add: Editor.get_prop/set_prop: PROP_ZEBRA

1.0.284 (app 1.79.0)
* add: msg_box_ex()
* add: Editor.get_prop/set_prop: COLOR_ID_BorderFocused, COLOR_ID_MinimapTooltipBg, COLOR_ID_MinimapTooltipBorder, COLOR_ID_BlockStapleActive
* add: app_proc: ids SPLITTER_G4, SPLITTER_G5

1.0.285 (app 1.79.1)
* add: Editor.gap: GAP_GET_ALL
* deprecated: GAP_GET_LIST

1.0.286 (app 1.79.2)
* add: Editor.get_prop: PROP_HANDLE_PARENT
* add: const COLOR_DEFAULT

1.0.287 (app 1.79.3)
* add: Editor.set_caret: param "options"
* add: Editor.set_caret: CARET_DELETE_INDEX
* add: Editor.get_filename: param "options"
* add: toolbar_proc: TOOLBAR_GET_BUTTON_HANDLES

1.0.288 (app 1.81.0)
* add: for BOOKMARK_SETUP it's allowed to use COLOR_DEFAULT
* change: removed menu id "_langs"
* change: removed menu id "_themes-ui"
* change: removed menu id "_themes-syntax"

1.0.289 (app 1.81.2)
* add: Editor.action()
* deprecated: Editor.lexer_scan()
* deprecated: Editor.export_html()

1.0.290 (app 1.81.3)
* add: listbox_proc: LISTBOX_GET_COLUMN_SEP, LISTBOX_SET_COLUMN_SEP
* add: listbox_proc: LISTBOX_GET_COLUMNS, LISTBOX_SET_COLUMNS
* removed: LISTBOX_THEME
* removed: event "on_tab_switch"

1.0.291 (app 1.83.0)
* add: emmet()
* add: app_proc: PROC_GET_FINDER_PROP dict has keys for option "Allowed syntax elements"

1.0.292 (app 1.83.2)
* add: Editor.action: EDACTION_UPDATE
* add: Editor.action: EDACTION_SHOW_POS

1.0.293 (app 1.84.2)
* removed deprecated: Editor.lexer_scan
* removed deprecated: Editor.export_html
* removed deprecated: PROP_COLUMN_LEFT
* removed deprecated: GAP_GET_LIST
* removed deprecated: PROP_EOL
* add: install.inf support for data with empty "subdir"
* add: install.inf support for "type=cudatext-package"

1.0.294 (app 1.84.5)
* add: timer_proc: behaviour when TIMER_START* restarts existing timer: now action updates old timer's tag to new value

1.0.295 (app 1.84.6)
* change: msg_status_alt now shows floating tooltip on the top (before: was statusbar on the bottom)

1.0.296 (app 1.85.1)
* add: dlg_proc: control property "texthint"

1.0.297 (app 1.85.1)
* add: dlg_dir: param "caption"
* add: dlg_file: param "caption"

1.0.298 (app 1.85.2)
* add: dlg_proc: control events "on_focus_enter", "on_focus_exit"

1.0.299 (app 1.86.0)
* change: Windows: embedded Python updated to 3.6
* change: Editor.set_text_all now looses Undo information

1.0.300 (app 1.86.2)
* deleted: STATUSBAR_GET_COLOR_FONT, STATUSBAR_SET_COLOR_FONT (other actions allow to get/set it)
* add: statusbar_proc: STATUSBAR_SET_CELL_FONT_NAME, STATUSBAR_GET_CELL_FONT_NAME
* add: statusbar_proc: STATUSBAR_SET_CELL_FONT_SIZE, STATUSBAR_GET_CELL_FONT_SIZE
* add: dlg_proc: DLG_LOCK, DLG_UNLOCK

1.0.301 (app 1.86.2)
* add: lexer_proc: LEXER_GET_STYLES

1.0.302 (app 1.86.3)
* add: app_proc: PROC_THEME_UI_DICT_GET
* add: app_proc: PROC_THEME_SYNTAX_DICT_GET
* deprecated: PROC_THEME_UI_DATA_GET
* deprecated: PROC_THEME_SYNTAX_DATA_GET

1.0.303 (app 1.87.5)
* add: listbox_proc: LISTBOX_GET_SHOW_X, LISTBOX_SET_SHOW_X
* add: listbox_proc: LISTBOX_GET_HOTTRACK, LISTBOX_SET_HOTTRACK
* add: listbox_proc: LISTBOX_GET_ITEM_PROP, LISTBOX_SET_ITEM_PROP
* add: listbox_proc: LISTBOX_ADD returns count of items
* add: dlg_proc: control "listbox_ex" has new event "on_click_x"

1.0.304 (app 1.87.5)
* add: listbox_proc: LISTBOX_ADD_PROP

1.0.305 (app 1.87.6)
* add: Editor.markers, Editor.attr: MARKERS_DELETE_BY_INDEX
* add: Editor.markers, Editor.attr: list allows "duplicate" items with the same x:y pos; attr list applies all "duplicate" items, left to right

1.0.306 (app 1.88.2)
* change: menu_proc: removed special menu id "_lexers"

1.0.307 (app 1.88.6)
* add: Editor.get_token: add TOKEN_GET_KIND
* add: Editor.action: add EDACTION_FIND_BRACKET

1.0.310 (app 1.88.8)
* add: Editor.attr: param "show_on_map" can be int
* add: Editor.attr: param "len" can be <0 to paint multiline micromap mark
* add: Editor.attr: added param "map_only"
* add: Editor.attr: MARKERS_ADD_MANY
* change: Editor.attr: added tuple item in MARKERS_GET
* add: Editor.action: EDACTION_MICROMAPCOL_GET
* add: Editor.action: EDACTION_MICROMAPCOL_ADD
* add: Editor.action: EDACTION_MICROMAPCOL_DELETE

1.0.312 (app 1.89.1)
* add: listbox_proc: LISTBOX_GET_SCROLLPOS_HORZ
* add: listbox_proc: LISTBOX_SET_SCROLLPOS_HORZ
* add: listbox_proc: LISTBOX_GET_SCROLLSTYLE_HORZ
* add: listbox_proc: LISTBOX_SET_SCROLLSTYLE_HORZ
* add: listbox_proc: LISTBOX_GET_SCROLLSTYLE_VERT
* add: listbox_proc: LISTBOX_SET_SCROLLSTYLE_VERT

1.0.313 (app 1.89.2)
* add: Editor.get_prop/set_prop: PROP_SCROLLSTYLE_HORZ, PROP_SCROLLSTYLE_VERT
* add: Editor.attr: MARKERS_GET_DICT

1.0.314 (app 1.89.4)
* add: Editor.action: EDACTION_FIND_BRACKETS
* deleted: EDACTION_FIND_BRACKET
* deleted deprecated: PROC_THEME_UI_DATA_GET
* deleted deprecated: PROC_THEME_SYNTAX_DATA_GET

1.0.315 (app 1.89.6)
* add: Editor.get_prop/set_prop: PROP_CODETREE_SUBLEXER
* add: dlg_commands: COMMANDS_FILES
* add: dlg_commands: COMMANDS_RECENTS

1.0.316 (app 1.90.0)
* add: app_proc: PROC_GET_COMMANDS returns new kinds "openedfile", "recentfile"

1.0.317 (app 1.91.1)
* add: supported encodings UTF-32 (LE and BE, with and without BOM)

1.0.318 (app 1.92.0)
* add: Editor.action: EDACTION_FIND_ONE
* add: Editor.action: EDACTION_FIND_ALL

1.0.319 (app 1.92.5)
* removed: menu_proc special constant "_enc"

1.0.320 (app 1.94.0)
* change: event on_state does not support EDSTATE_ values
* change: events are now called with ed_self==None: on_state, on_start, on_exit, on_output_nav, on_console_nav
* add: event on_state_ed

1.0.321 (app 1.94.5)
* add: events on_open / on_open_pre support filtering via "keys=" in install.inf like on_key

1.0.322 (app 1.96.2)
* add: Editor.action: EDACTION_FIND_ALL supports param3
* add: lexer_proc: LEXER_DETECT can give tuple result

1.0.323 (app 1.97.1)
* add: Editor.get_prop/set_prop: PROP_SCALE_FONT

1.0.324 (app 1.97.2)
* add: app_proc: PROC_SET_FOLDER
* add: file_open: added option "/nolexerdetect"
* add: file_open: added option "/noopenedevent"
* add: file_open: added option "/nononeevent"

1.0.325 (app 1.97.5.2)
* add: Editor.get_prop: PROP_IN_SESSION
* add: Editor.get_prop: PROP_IN_HISTORY

1.0.326 (app 1.98.3)
* add: dlg_proc: DLG_CTL_PROP_GET returns also the parent info in key "p"

1.0.327 (app 1.99.5)
* add: event "on_click_link"
* add: install.inf supports sections "bottombar*" to show button on sidebar near Console/Output, without loading the plugin
* change: install.inf supports sections "item*", earlier allowed sections were limited by regex "item\d+"
* change: install.inf supports sections "sidebar*", earlier allowed sections were limited by regex "sidebar\d+"
* change: install.inf supports sections "treehelper*", earlier allowed sections were limited by regex "treehelper\d+"
* add: dlg_proc: control "radiogroup" supports event "on_change"
* add: dlg_proc: control "checkgroup" supports event "on_change"

1.0.328 (app 1.100.5)
* change: changed format of string in PROP_UNDO_DATA, PROP_REDO_DATA (field "line state" was inserted in the middle of lines, could not append it to the end)

1.0.329 (app 1.101.0)
* add: event "on_message"

1.0.330 (app 1.102.2)
* add: Editor.get_prop/set_prop: PROP_FONT, PROP_FONT_B, PROP_FONT_I, PROP_FONT_BI
* add: Editor.get_prop/set_prop: PROP_ZEBRA_STEP
* add: Editor.get_prop/set_prop: PROP_LINKS_SHOW
* add: Editor.get_prop/set_prop: PROP_LINKS_REGEX
* add: Editor.get_prop/set_prop: PROP_LINKS_CLICKS
* add: dlg_proc: editor event "on_click_link"

1.0.331 (app 1.102.6)
* add: Editor.micromap()
* change: Editor.action: deleted actions EDACTION_MICROMAPCOL_*
* add: constant "cudatext.API", which will replace "cudatext.app_api_version" later

1.0.332 (app 1.102.7)
* add: lexer_proc: LEXER_ADD_VIRTUAL
* add: finder options as string (several APIs): support also "b" for backward search
* add: Editor.action: EDACTION_REPLACE_ONE

1.0.333 (app 1.103.2)
* add: Editor.action: EDACTION_REPLACE_ALL

1.0.334 (app 1.104.1)
* add: dlg_menu: params "clip", "w", "h"
* add: dlg_commands: COMMANDS_CONFIG_LEXER

1.0.335 (app 1.104.5)
* add: dlg_commands: params "w", "h"
* add: support encodings "koi8r", "koi8u", "koi8ru"

1.0.336 (app 1.105.1)
* add: Editor.set_prop: PROP_ENC_RELOAD

1.0.337 (app 1.105.1)
* fix: lexer_proc: LEXER_GET_PROP didn't return "sub" key
* add: lexer_proc: LEXER_GET_PROP returns new "kinds" key
* add: Editor.get_token: TOKEN_LIST_ALT

1.0.338 (app 1.105.2)
* add: msg_box_ex: added param "focused"

1.0.339 (app 1.105.3)
* deleted: Editor.get_token: TOKEN_LIST_ALT
* add: Editor.get_token: TOKEN_LIST* return additional keys "ki", "ks"
* add: lexer_proc: LEXER_GET_STYLES returns additional key "tkind"

1.0.340 (app 1.105.3)
* add: Editor.get_prop/set_prop: PROP_V_WIDTH_HEX, PROP_V_WIDTH_UHEX
* add: Editor.set_prop: setting PROP_V_MODE for usual editor switches editor to viewer, with given mode
* add: Editor.get_prop/set_prop: can switch from viewer to editor using new value VMODE_NONE

1.0.341 (app 1.105.6)
* add: Editor.action: EDACTION_UPDATE handles "param1"

1.0.342 (app 1.106.1)
* add: support loading plugin events also from settings/plugins.ini [events] cuda_modulename=comma_separated_events
* change: Editor.action: EDACTION_FIND* and EDACTION_REPLACE* return False for incorrect RegEx
* add: Editor.set_prop: PROP_PREVIEW is writable now

1.0.343 (app 1.106.8)
* add: app_proc: PROC_CONFIG_SCALE_GET, PROC_CONFIG_SCALE_SET
* add: finder_proc()

1.0.344 (app 1.106.8)
* add: dlg_proc: controls "editor_edit", "editor_combo"
* add: Editor.get_prop/set_prop: PROP_COMBO_ITEMS

1.0.345 (app 1.107.0)
* add: override Python's builtins.input() with CudaText implementation

1.0.346 (app 1.107.2)
* add: dlg_menu: option MENU_EDITORFONT

1.0.347 (app 1.107.6)
* add: app_proc: PROC_GET_FINDER_PROP/PROC_SET_FINDER_PROP support new keys: "find_h", "rep_h"

1.0.348 (app 1.108.0)
* add: Editor.action: EDACTION_UNDOGROUP_BEGIN, EDACTION_UNDOGROUP_END

1.0.349 (app 1.108.6)
* add: dlg_proc: DLG_POS_GET_STR, DLG_POS_SET_FROM_STR

1.0.350 (app 1.110.0)
* add: toolbar_proc: actions TOOLBAR_ADD_ITEM, TOOLBAR_ADD_MENU return handle of new button
* change: Editor.complete: "tooltip" line part is now shown only in tooltip, but not in listbox
* add: Editor.complete: "tooltip" line part can be multi-line, separator is chr(2)

1.0.351 (app 1.114.0)
* add: event on_app_activate
* add: event on_app_deactivate

1.0.352 (app 1.115.6)
* add: app_path: APP_DIR_SETTINGS_DEF
* add: file_open: added possible option "/nontext-view-uhex"

1.0.353 (app 1.115.7)
* add: Editor.markers: add param "line_len"
* add: Editor.markers: MARKERS_GET_DICT

1.0.354 (app 1.117.2)
* add: Editor.bookmark: BOOKMARK_GET_ALL gets additional dict key "auto_delx"

1.0.355 (app 1.119.5)
* change: Editor.attr: change param 'map_only' type from bool to int (enum)

1.0.357 (app 1.121.4)
* add: statusbar_proc: STATUSBAR_SET_CELL_CALLBACK, STATUSBAR_GET_CELL_CALLBACK
* add: app_proc: PROC_GET_UNIQUE_TAG

1.0.358 (app 1.123.0)
* add: dlg_proc: added props for control "listview": "imagelist_small", "imagelist_large", "imageindexes"
* add: tree_proc: added param "data" for TREE_ITEM_ADD
* add: tree_proc: added result key "data" for TREE_ITEM_GET_PROPS
* add: tree_proc: added TREE_ITEM_ENUM_EX

1.0.360 (app 1.123.0)
* add: dlg_proc: controls "tabs"/"pages" give property "tab_hovered"
* add: dlg_proc: control "pages" gives property "ex0" for tab-position
* change: dlg_proc: control "tabs" gives prop "ex0" with 4 values '0'..'3', previously it was bool '0'/'1'
* add: toolbar_proc: add TOOLBAR_GET_INDEX_HOVERED

</details>

---
