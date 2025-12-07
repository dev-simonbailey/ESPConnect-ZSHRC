🚀 espconnect Command

The espconnect command is a convenient macOS function that streamlines running the ESPConnect UI from any terminal session. It automatically starts a local server, handles port conflicts safely, and opens the UI in Google Chrome.

⸻

📥 Installation

Add the function below to your ~/.zshrc:
```
espconnect() {
  local dir="/Applications/MAMP/htdocs/github/ESPConnect/dist"
  local port="${1:-3000}"

  echo "▶ Using port: $port"
  echo "▶ Changing directory to: $dir"

  cd "$dir" || {
    echo "❌ Cannot cd into $dir"
    return 1
  }

  # Find all PIDs using this port
  local existing_pids
  existing_pids=$(lsof -ti tcp:"$port" 2>/dev/null || true)

  if [[ -n "$existing_pids" ]]; then
    echo "⚠️  Found process(es) listening on port $port:"

    # Show human-readable info for each PID
    for pid in $existing_pids; do
      local info
      info=$(ps -o pid=,comm=,args= -p "$pid")
      echo "   → $info"
    done

    printf "❓ Kill these process(es) before starting a new server? [y/N] "
    read -r answer

    if [[ "$answer" =~ ^[Yy]$ ]]; then
      echo "🔪 Killing process(es): $existing_pids"
      kill $existing_pids 2>/dev/null || {
        echo "❌ Failed to kill some process(es) on port $port"
        return 1
      }
      sleep 1
    else
      echo "🚫 Aborting without starting a new server."
      return 1
    fi
  fi

  echo "🚀 Starting npx serve on port $port..."
  npx serve -l "$port" &

  local serve_pid=$!
  echo "✅ npx serve started with PID $serve_pid"

  sleep 2

  local url="http://localhost:$port"
  echo "🌐 Opening $url in Chrome..."
  open -a "Google Chrome" "$url"
}
```

Reload your terminal configuration:

```source ~/.zshrc```

🧠 What This Command Does
- Accepts an optional port argument
- Defaults to 3000 if none is provided
- Automatically navigates to the ESPConnect build directory
- Detects existing processes already using the selected port
- Displays human-readable details for those processes:
- PID
- Command
- Full execution arguments
- Prompts the user before terminating anything
- Safely kills existing processes only when confirmed
- Launches a new npx serve instance on the chosen port
- Reports the PID of the running server
- Waits briefly to ensure the server initializes
- Opens Google Chrome with a new tab pointing at the URL
- Brings Chrome to the foreground
- Exits gracefully if:
- The directory cannot be accessed
- The user chooses not to kill existing processes

🕹 Usage

Default port (3000)

```espconnect```

Custom Port

```espconnect 4173```

Example prompt when a process is running on that port

```
⚠️ Found process(es) listening on port 4173:
   → 12345 node    node /usr/local/bin/serve
❓ Kill these process(es) before starting a new server? [y/N]
```

🛠 Troubleshooting

❌ Chrome doesn’t open

Ensure Chrome exists at:

```/Applications/Google Chrome.app```

If you use a different browser, update this line:

```open -a "Google Chrome" "$url"```

❌ npx serve reports “port already in use” after killing

macOS sometimes delays port freeing by a few seconds.
Run again:

```espconnect <port>```

or add a longer delay:

```sleep 3```

❌ serve command not found

Install the package:

```npm install -g serve```

(Or keep using npx — both work.)

❌ Permission denied

Ensure you have access to the directory:

```cd /Applications/MAMP/htdocs/github/ESPConnect/dist```

🔧 Future Enhancements (Optional)

These can be added later if you want a deluxe version:
- Port availability spinner
  - Show an animated spinner while waiting for the port to free.
- Server start verification
  - Poll http://localhost:<port> until it responds before opening Chrome.
- Live log tailing
  - Capture and display output from the serve process.
- Auto-restart detection
  - Restart the server if it crashes.
- Environment variables
  - Allow $ESPCONNECT_DIR and $ESPCONNECT_DEFAULT_PORT.

🧹 Uninstall or Modify

Remove the command

Open your zshrc:

```nano ~/.zshrc```

Delete the espconnect() function block.

Reload:

```source ~/.zshrc```

Change the directory or default port

Edit the top of the function:

```
local dir="..." local port="${1:-3000}"
```

📝 How to Use Nano (Open, Edit, Save, Exit)

Nano is a simple terminal-based text editor included with macOS.
Here’s a quick guide to the key commands you’ll need.

⸻

📂 Opening a File in Nano

```nano <filename>```

Example:
```nano ~/.zshrc```

If the file doesn’t exist, Nano will create it when you save.

⸻

✏️ Editing Inside Nano

Just start typing — everything you type is inserted directly into the file.

⸻

💾 Saving Your Changes

Press:

```Ctrl + O```

Nano will show

```File Name to Write: <filename>```

Press Enter to confirm and save.

⸻

🚪 Exiting Nano

Press:

```Ctrl + X```

If you have unsaved changes, Nano will ask:

```Save modified buffer (ANSWERING "No" WILL DESTROY CHANGES)?```

Press:
- Y = save changes
- N = exit without saving

If you press Y, Nano will ask you to confirm the filename again — just hit Enter.

🔁 Summary Table

-  Action
  - Keys
- Save changes
 -  Ctrl + O
- Exit Nano
  - Ctrl + X
- Save + Exit combo
  - Ctrl + O, then Enter, then Ctrl + X
- Cancel an action
  - Ctrl + C

