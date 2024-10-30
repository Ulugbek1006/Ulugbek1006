import asyncio  # asyncio kutubxonasini import qilish
from telegram import Update
from telegram.ext import Application, CommandHandler, ContextTypes
import logging
import json
from datetime import datetime

# Logging sozlamalari
logging.basicConfig(format='%(asctime)s - %(name)s - %(levelname)s - %(message)s', level=logging.INFO)

# O'qituvchilar va to'lovlar ma'lumotlarini saqlash
data = {
    'teachers': {},  # O'qituvchilar
    'payments': {}   # To'lovlar
}

# Ma'lumotlarni fayldan yuklash
def load_data():
    global data
    try:
        with open('data.json', 'r') as f:
            data = json.load(f)
    except FileNotFoundError:
        data = {'teachers': {}, 'payments': {}}

# Ma'lumotlarni faylga saqlash
def save_data():
    with open('data.json', 'w') as f:
        json.dump(data, f)

# /start komandasi
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("Assalomu alaykum! O'quv markazining to'lov botiga xush kelibsiz!")

# O'qituvchi qo'shish funksiyasi
async def add_teacher(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if len(context.args) < 2:
        await update.message.reply_text("Iltimos, o'qituvchining ismi va familiyasini kiriting.")
        return
    name = context.args[0]
    surname = context.args[1]
    key = f"{name} {surname}"
    if key not in data['teachers']:
        data['teachers'][key] = []
        await update.message.reply_text(f"O'qituvchi qo'shildi: {key}")
    else:
        await update.message.reply_text("Bu o'qituvchi allaqachon mavjud.")
    save_data()

# To'lov qo'shish funksiyasi
async def add_payment(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if len(context.args) < 3:
        await update.message.reply_text("Iltimos, o'qituvchi ismi, familiyasi va to'lov summasini kiriting.")
        return
    teacher_name = f"{context.args[0]} {context.args[1]}"
    amount = float(context.args[2])
    if teacher_name in data['teachers']:
        payment_date = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        payment_entry = {
            'date': payment_date,
            'amount': amount,
            'user': update.message.from_user.username
        }
        data['teachers'][teacher_name].append(payment_entry)
        await update.message.reply_text(f"To'lov qo'shildi: {amount} so'm {teacher_name} uchun.")
        save_data()
    else:
        await update.message.reply_text("Bunday o'qituvchi mavjud emas.")

# O'qituvchi bo'yicha jami to'lovlarni ko'rsatish
async def show_payments(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if len(context.args) < 2:
        await update.message.reply_text("Iltimos, o'qituvchi ismi va familiyasini kiriting.")
        return
    teacher_name = f"{context.args[0]} {context.args[1]}"
    if teacher_name in data['teachers']:
        payments = data['teachers'][teacher_name]
        total_amount = sum(payment['amount'] for payment in payments)
        response = f"{teacher_name} uchun jami to'lovlar:\n"
        response += f"Jami summa: {total_amount} so'm\n"
        response += "To'lovlar:\n"
        for payment in payments:
            response += f"{payment['date']}: {payment['amount']} so'm (To'lov qilgan: {payment['user']})\n"
        await update.message.reply_text(response)
    else:
        await update.message.reply_text("Bunday o'qituvchi mavjud emas.")

# Asosiy funksiya
async def run_bot():
    load_data()  # Fayldan ma'lumotlarni yuklash
    application = Application.builder().token("8002094853:AAGnBJMTt75WpM3zvrk-kuzrQcyImsMK6wI").build()
    
    # Bot komandalarini qo'shish
    application.add_handler(CommandHandler("start", start))
    application.add_handler(CommandHandler("add_teacher", add_teacher))
    application.add_handler(CommandHandler("add_payment", add_payment))
    application.add_handler(CommandHandler("show_payments", show_payments))
    
    # Botni ishga tushirish
    await application.run_polling()

# Botni ishga tushirish
if __name__ == '__main__':
    import nest_asyncio
    nest_asyncio.apply()  # Asinxron muhit bilan moslash uchun qo'shimcha
    asyncio.run(run_bot())
