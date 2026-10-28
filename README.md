# jalako-bot
import os
import requests
from telegram import Update
from telegram.ext import (
    Application,
    CommandHandler,
    MessageHandler,
    ContextTypes,
    filters,
)

TOKEN = os.environ["BOT_TOKEN"]

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(
        "مرحبًا! أرسل اسم ما تبحث عنه، "
        "وسأبحث عن الصور والفيديو والصوت."
    )

async def search(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.message.text.strip()

    await update.message.reply_text("🔎 جارٍ البحث...")

    try:
        response = requests.get(
            "https://commons.wikimedia.org/w/api.php",
            params={
                "action": "query",
                "generator": "search",
                "gsrsearch": query,
                "gsrnamespace": 6,
                "gsrlimit": 8,
                "prop": "imageinfo",
                "iiprop": "url|mime",
                "format": "json",
            },
            timeout=20,
        )
        response.raise_for_status()
        pages = response.json().get("query", {}).get("pages", {})

        if not pages:
            await update.message.reply_text("لم أجد نتائج. جرّب اسمًا آخر.")
            return

        for page in list(pages.values())[:8]:
            info = page.get("imageinfo", [{}])[0]
            url = info.get("url", "")
            title = page.get("title", "ملف")

            if url:
                await update.message.reply_text(
                    f"📁 {title}\n{url}"
                )

    except Exception:
        await update.message.reply_text(
            "تعذر البحث الآن. حاول مرة أخرى لاحقًا."
        )

app = Application.builder().token(TOKEN).build()
app.add_handler(CommandHandler("start", start))
app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, search))
app.run_polling()
