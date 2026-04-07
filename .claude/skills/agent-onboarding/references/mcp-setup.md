# MCP Setup

Читай этот файл на Шаге 4 блока 3.

---

## Уточнения по сервисам

### Jira

Задай через ask_user_input:
> Как ты заходишь в Jira?
> Это важно: от этого зависит можно ли подключить Jira автоматически.

- Через браузер, адрес **jira.atlassian.net/...** — стандартный облачный Jira
- Через браузер, адрес **другой** (например jira.mycompany.com) — Jira на сервере компании
- Не уверен — сейчас проверю

Если «Не уверен»:
> Открой Jira в браузере, посмотри адрес и напиши что там —
> например "jira.atlassian.net/..." или "jira.acme.com/..."

Домен `.atlassian.net` → Cloud → добавляем. Любой другой → Self-hosted → пропускаем.

### Notion

Задай через ask_user_input:
> Как ты заходишь в Notion?
> Корпоративный Notion требует ручной настройки через IT.

- Через браузер или приложение, адрес **notion.so** — стандартный Notion
- Через **адрес компании** (например notion.mycompany.com) — корпоративный Notion
- Не уверен

Если «Не уверен»:
> Открой Notion в браузере — адрес начинается на notion.so или на что-то другое?

notion.so → добавляем. Корпоративный → пропускаем.

---

## Серверы

Включай только прошедшие фильтрацию:

```json
"figma":        { "type": "http", "url": "https://mcp.figma.com/mcp" }
"atlassian":    { "type": "http", "url": "https://mcp.atlassian.com/v1/mcp" }
"linear":       { "type": "stdio", "command": "npx", "args": ["-y", "linear-mcp-server"] }
"notion":       { "type": "stdio", "command": "npx", "args": ["-y", "@notionhq/notion-mcp-server"] }
"google-drive": { "type": "http", "url": "https://drive.mcp.claude.com/mcp" }
```

---

## Путь к глобальному конфигу

```
Mac:     ~/Library/Application Support/Claude/claude_desktop_config.json
Windows: %APPDATA%\Claude\claude_desktop_config.json
Linux:   ~/.config/Claude/claude_desktop_config.json
```

---

## Патч конфига

Читай существующий конфиг, мержи серверы, записывай обратно.
Не затирай существующие серверы.

```bash
python3 << PYEOF
import json, os, sys

os_name = "[Q2]"  # Mac / Windows / Linux

paths = {
    "Mac":     os.path.expanduser("~/Library/Application Support/Claude/claude_desktop_config.json"),
    "Windows": os.path.expandvars(r"%APPDATA%\Claude\claude_desktop_config.json"),
    "Linux":   os.path.expanduser("~/.config/Claude/claude_desktop_config.json"),
}

config_path = paths.get(os_name)
if not config_path:
    print("Неизвестная ОС"); sys.exit(1)

os.makedirs(os.path.dirname(config_path), exist_ok=True)

config = {}
if os.path.exists(config_path):
    with open(config_path) as f:
        config = json.load(f)

if "mcpServers" not in config:
    config["mcpServers"] = {}

# Подставь только нужные серверы из списка выше
new_servers = {
    # "figma": { ... },
    # "atlassian": { ... },
}

config["mcpServers"].update(new_servers)

with open(config_path, "w") as f:
    json.dump(config, f, indent=2, ensure_ascii=False)

print("Подключено:", list(new_servers.keys()))
PYEOF
```

---

## Перезапуск Claude Desktop

После записи конфига перезапусти Claude Desktop:

```bash
# Mac
osascript -e 'quit app "Claude"' && sleep 2 && open -a "Claude"

# Linux
pkill -f "claude" && sleep 2 && nohup claude > /dev/null 2>&1 &

# Windows (PowerShell)
Stop-Process -Name "Claude" -ErrorAction SilentlyContinue; Start-Sleep 2; Start-Process "Claude"
```

---

## Ограничения

**Jira self-hosted** — MCP работает только с Jira Cloud (atlassian.net).
Для корпоративного Jira: попроси IT подключить Atlassian Cloud или
передавай тикеты в агент вручную через текст.

**Notion корпоративный** — требует настройки OAuth через IT-отдел.
Пока что: копируй нужные страницы в чат с агентом вручную.
