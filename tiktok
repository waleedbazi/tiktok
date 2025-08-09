import os
import re
import yt_dlp
import asyncio  # **Pêwîst e ji bo birêvebirina karên hevdem**
from telegram import Update, InputMediaVideo
from telegram.ext import ApplicationBuilder, CommandHandler, MessageHandler, filters, ContextTypes

# ==================🔐 تۆکنا بۆتی 🔐==================
BOT_TOKEN = "7150349798:AAGJ1Y5wJPzms58lLvM_S1ZUjuSr7uILCS8" # ل ڤێرێ تۆکنا خۆ یا نهێنی بنڤیسە
# =================================================================

TIKTOK_REGEX = r"(https?://(?:www\.|vm\.|vt\.)?tiktok\.com/[^\s]+)"

# Fenkşenek ku karê daxistinê yê 'blokker' dike
def download_video_sync(url, ydl_opts):
    """Ev fenkşene karê daxistinê dike û dê di thread'ek cûda de were xebitandin."""
    with yt_dlp.YoutubeDL(ydl_opts) as ydl:
        info_dict = ydl.extract_info(url, download=True)
        return ydl.prepare_filename(info_dict)

# Karipêkerê fermana /start
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Peyama bi xêrhatinê dişîne dema ku fermana /start tê bikaranîn."""
    await update.message.reply_text(
        "👋 سلاڤ و بخێرهاتی!\n\n"
        "ئەز بوتێ دابەزاندنا ڤیدیۆیێن تیکتۆکی مە 📥\n\n"
        "هیڤیە لینکێ ڤیدیویا خوو ڤرێکە داکوو بوتە دانلودبکەم."
    )

# Karipêkerê sereke yê peyaman
async def handle_message(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Berpirsiyar e ji wergirtina peyaman ji bo daxistina vîdyoyên TikTok."""
    text = update.message.text or update.message.caption
    if not text:
        return

    match = re.search(TIKTOK_REGEX, text.strip())

    if not match:
        await update.message.reply_text(
            "🤔 ببورە، ئەز د لینکێ تە نەگەهشتم!\n\n"
            "⚠️ هیڤیە پشتراست بە کو ئەڤ لینکە یێ دروستە."
        )
        return

    url = match.group(1)
    processing_message = await update.message.reply_text(
        "⏳ دخازم چاڤەرێ بی...\n\n"
        "🚀 ڤیدیۆیا تە یا دهێتە ئامادەکرن."
    )

    file_path = None
    try:
        ydl_opts = {
            'outtmpl': f'%(id)s_{update.effective_message.message_id}.%(ext)s', # Navek yekta ji bo her daxwazekê
            'format': 'bestvideo[ext=mp4]+bestaudio[ext=m4a]/best[ext=mp4]/best',
            'merge_output_format': 'mp4',
            'quiet': True,
            'noprogress': True,
        }

        # **Lİ VÊRÊ ÇARESARÎYA SEREKÎ YE**
        # Karê daxistinê di thread'ek cûda de dimeşîne da ku bot blok nebe
        loop = asyncio.get_running_loop()
        file_path = await loop.run_in_executor(
            None, download_video_sync, url, ydl_opts
        )

        media = InputMediaVideo(
            media=open(file_path, 'rb'),
            caption="✅ فەرموو، ڤیدیۆیا تە هاتە دابەزاندن\n\n @waleedownloader_bot"
        )
        await processing_message.edit_media(media=media)

    except Exception as e:
        print(f"خەلەتی د پرۆسێسا {url} دا: {e}")
        await processing_message.edit_text(
            "🚫 ببورە، خەلەتیەکا تەکنیکی رویدا!\n\n"
            "⚠️ هیڤیە جاره‌کا دی تاقی بکە یان لینکەکێ دی فرێکە."
        )

    finally:
        # Paqijkirina pelê piştî ku kar bi dawî dibe
        if file_path and os.path.exists(file_path):
            try:
                os.remove(file_path)
            except OSError as e:
                print(f"Xeta di jêbirina pelê {file_path} de: {e}")

# Fenkşena sereke ji bo sazkirin û xebitandina botê
def main():
    """Botê dest pê dike."""
    print("🚀 بۆت یا هێتە دروستکرن...")
    # **Bi `concurrent_updates=True` piştrast dibe ku bot dikare gelek daxwazan bi hev re bi rê ve bibe**
    app = ApplicationBuilder().token(BOT_TOKEN).concurrent_updates(True).build()

    app.add_handler(CommandHandler("start", start))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_message))

    print("✅ بۆت ب سەرکەفتیانە کار دکەت و ئامادەیە!")
    app.run_polling()

if __name__ == "__main__":
    main()
