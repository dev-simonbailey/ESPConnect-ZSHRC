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
