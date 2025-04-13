from telegram import Update
from telegram.ext import ApplicationBuilder, CommandHandler, MessageHandler, filters, ContextTypes
import json, os

FILE_NAME = "anime_links.json"
ADMIN_USERS = [ 7264292442 ]  # O‘zingiz va do‘stlaringiz Telegram ID’lari bu yerga yoziladi

# Fayldan linklarni o'qish
def load_links():
    if os.path.exists(FILE_NAME):
        with open(FILE_NAME, "r") as f:
            return json.load(f)
    return {}

# Faylga linklarni saqlash
def save_links(data):
    with open(FILE_NAME, "w") as f:
        json.dump(data, f, indent=4)

# /start komandasi
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    print(f"Foydalanuvchi ID: {user_id}")  # Telegram ID’ni terminalda ko‘rsatadi
    await update.message.reply_text(
        "Salom! Bu bot orqali O'zbek tilidagi animelarni raqam orqali ko‘rishingiz mumkin.\n\n"
        "1️⃣ Raqam yuboring: masalan, 1 yoki 2\n"
        "➕ Yangi anime qo‘shish: +raqam link (faqat adminlar uchun)\n"
        "❗️Eslatma: Har bir raqamga faqat 1 ta link qo‘shilishi mumkin!"
    )

# Xabarlarni ishlash
async def handle_message(update: Update, context: ContextTypes.DEFAULT_TYPE):
    text = update.message.text.strip()
    user_id = update.effective_user.id
    anime_links = load_links()  # Fayldan o‘qish

    # Yangi link qo‘shish uchun
    if text.startswith("+"):
        if user_id not in ADMIN_USERS:
            await update.message.reply_text("Kechirasiz, sizga link qo‘shishga ruxsat yo‘q.")
            return
        try:
            parts = text[1:].split(" ", 1)
            key = parts[0].strip()
            value = parts[1].strip()

            if key in anime_links:
                await update.message.reply_text(f"{key}-raqam allaqachon mavjud!\n"
                                                "Yangi link qo‘shish uchun boshqa raqamdan foydalaning.")
                return

            anime_links[key] = value
            save_links(anime_links)
            await update.message.reply_text(f"{key}-raqamli anime muvaffaqiyatli qo‘shildi!")
        except:
            await update.message.reply_text("Xato format! To‘g‘ri yozing: +raqam link")
        return

    # Anime linkni yuborish
    if text in anime_links:
        await update.message.reply_text(f"Siz tanlagan anime:\n{anime_links[text]}")
    else:
        await update.message.reply_text("Bu raqam topilmadi. Yangi anime qo‘shish uchun: +raqam link")

# Botni ishga tushirish
app = ApplicationBuilder().token("8110132245:AAGY-4vtXMeBTMgru78aJnzU6WpNI5ySCSs").build()
app.add_handler(CommandHandler("start", start))
app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_message))

print("Bot ishga tushdi...")
app.run_polling()
