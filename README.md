import os
import subprocess
from telegram import Update
from telegram.ext import ApplicationBuilder, CommandHandler, MessageHandler, filters, ContextTypes

TOKEN = "8116047112:AAGUItwUw20E2t686gNLMC6lSefzROPfIs8"

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("مرحباً! أرسل لي ملف بايثون (*.py) لأشغله لك.")

async def handle_document(update: Update, context: ContextTypes.DEFAULT_TYPE):
    document = update.message.document
    if document.file_name.endswith(".py"):
        file = await document.get_file()
        await file.download_to_drive("user_script.py")
        await update.message.reply_text("تم استلام الملف، جاري تشغيله...")

        try:
            result = subprocess.run(
                ["python3", "user_script.py"], capture_output=True, text=True, timeout=30
            )
            output = result.stdout + result.stderr
            if len(output) > 4000:
                output = output[:4000] + "\n...Output truncated."
            await update.message.reply_text(f"نتيجة التشغيل:\n{output}")
        except subprocess.TimeoutExpired:
            await update.message.reply_text("انتهى الوقت المحدد للتشغيل (30 ثانية).")
        except Exception as e:
            await update.message.reply_text(f"حدث خطأ أثناء التشغيل:\n{e}")

    else:
        await update.message.reply_text("من فضلك أرسل ملف بايثون بصيغة .py فقط.")

async def unknown(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("عذراً، لم أفهم الأمر. أرسل ملف بايثون لأشغله.")

if __name__ == "__main__":
    app = ApplicationBuilder().token(TOKEN).build()

    app.add_handler(CommandHandler("start", start))
    app.add_handler(MessageHandler(filters.Document.ALL, handle_document))
    app.add_handler(MessageHandler(filters.COMMAND, unknown))

    print("بوت تيليجرام يعمل...")
    app.run_polling()
