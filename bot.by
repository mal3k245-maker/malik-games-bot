
import urllib.request
import urllib.parse
import json
import time
import os

TOKEN = os.environ.get("BOT_TOKEN", "")
CHANNEL = "@gamez0ne111"

if not TOKEN:
    raise ValueError("BOT_TOKEN is missing!")

API = "https://api.telegram.org/bot" + TOKEN + "/"

GAMES = {
    "game1": ("صلي على النبي", "عليه أفضل الصلاة والسلام"),
    "game2": ("Game 2", "١١١")
}

def request(method, data=None):
    url = API + method
    encoded = urllib.parse.urlencode(data or {}).encode()
    req = urllib.request.Request(url, data=encoded)

    with urllib.request.urlopen(req, timeout=35) as response:
        result = json.loads(response.read().decode())

    if not result.get("ok"):
        raise Exception(result.get("description"))

    return result

def send(chat_id, text, keyboard=None):
    data = {"chat_id": chat_id, "text": text}

    if keyboard:
        data["reply_markup"] = json.dumps(
            keyboard, ensure_ascii=False
        )

    request("sendMessage", data)

def subscribed(user_id):
    result = request("getChatMember", {
        "chat_id": CHANNEL,
        "user_id": user_id
    })

    member = result["result"]

    return (
        member["status"] in ("member", "administrator", "creator")
        or (
            member["status"] == "restricted"
            and member.get("is_member", False)
        )
    )

def menu(chat_id):
    buttons = []

    for key, game in GAMES.items():
        buttons.append([{
            "text": game[0],
            "callback_data": key
        }])

    send(chat_id, "🎮 اختار اللعبة:", {
        "inline_keyboard": buttons
    })

def main():
    print("MALIK GAMES IS RUNNING!")
    offset = 0

    while True:
        try:
            updates = request("getUpdates", {
                "offset": offset,
                "timeout": 25
            })

            for update in updates["result"]:
                offset = update["update_id"] + 1

                if "message" in update:
                    msg = update["message"]
                    chat_id = msg["chat"]["id"]

                    if msg.get("text") == "/start":
                        send(
                            chat_id,
                            "🎮 Welcome to MALIK GAMES!\nاشترك في القناة ثم اضغط تحقق.",
                            {
                                "inline_keyboard": [
                                    [{
                                        "text": "📢 اشترك في القناة",
                                        "url": "https://t.me/" + CHANNEL[1:]
                                    }],
                                    [{
                                        "text": "✅ تحقق من الاشتراك",
                                        "callback_data": "check"
                                    }]
                                ]
                            }
                        )

                elif "callback_query" in update:
                    query = update["callback_query"]

                    request("answerCallbackQuery", {
                        "callback_query_id": query["id"]
                    })

                    user_id = query["from"]["id"]
                    chat_id = query["message"]["chat"]["id"]
                    data = query["data"]

                    try:
                        is_member = subscribed(user_id)
                    except Exception as e:
                        print("Subscription Error:", e)
                        send(chat_id, "⚠️ حصل خطأ في التحقق. حاول تاني.")
                        continue

                    if data == "check":
                        if is_member:
                            send(chat_id, "✅ تم التحقق! اختار لعبتك.")
                            menu(chat_id)
                        else:
                            send(chat_id, "❌ اشترك في القناة الأول.")

                    elif data in GAMES:
                        if is_member:
                            send(
                                chat_id,
                                "🔐 كلمة سر " + GAMES[data][0] +
                                " هي:\n" + GAMES[data][1]
                            )
                        else:
                            send(chat_id, "❌ لازم تشترك الأول.")

        except Exception as e:
            print("Error:", e)
            time.sleep(5)

if __name__ == "__main__":
    main()
