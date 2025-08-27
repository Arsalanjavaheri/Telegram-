import logging
from telegram import Update, ChatPermissions
from telegram.ext import Application, CommandHandler, MessageHandler, filters, ContextTypes

TOKEN = '8375415243:AAGnUvgxUupglabW3OAJXPNAOjAhJ2lwpbc'  # توکن ربات خود را اینجا قرار دهید

# تنظیم لاگ‌ها
logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)

# خوش‌آمدگویی به عضو جدید
async def welcome(update: Update, context: ContextTypes.DEFAULT_TYPE):
    for member in update.message.new_chat_members:
        await update.message.reply_text(f'سلام {member.full_name} به گروه خوش آمدی!')

# حذف پیام‌های شامل کلمات ممنوعه
BAD_WORDS = ['بد', 'زشت', 'فحش']

async def filter_bad_words(update: Update, context: ContextTypes.DEFAULT_TYPE):
    text = update.message.text.lower()
    if any(word in text for word in BAD_WORDS):
        await update.message.delete()
        await update.message.reply_text('استفاده از کلمات نامناسب ممنوع است!')

# بن کردن کاربر
async def ban(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not update.message.reply_to_message:
        await update.message.reply_text('این دستور را در پاسخ به پیام کاربر ارسال کنید.')
        return
    user_id = update.message.reply_to_message.from_user.id
    await update.effective_chat.ban_member(user_id)
    await update.message.reply_text('کاربر بن شد.')

# کیک کردن کاربر
async def kick(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not update.message.reply_to_message:
        await update.message.reply_text('این دستور را در پاسخ به پیام کاربر ارسال کنید.')
        return
    user_id = update.message.reply_to_message.from_user.id
    await update.effective_chat.ban_member(user_id)
    await update.effective_chat.unban_member(user_id)
    await update.message.reply_text('کاربر کیک شد.')

# سکوت کردن کاربر
async def mute(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not update.message.reply_to_message:
        await update.message.reply_text('این دستور را در پاسخ به پیام کاربر ارسال کنید.')
        return
    user_id = update.message.reply_to_message.from_user.id
    await update.effective_chat.restrict_member(
        user_id,
        ChatPermissions(can_send_messages=False)
    )
    await update.message.reply_text('کاربر به حالت سکوت رفت.')

# ساخت اپلیکیشن ربات و اضافه کردن هندلرها
def main():
    application = Application.builder().token(TOKEN).build()

    application.add_handler(MessageHandler(filters.StatusUpdate.NEW_CHAT_MEMBERS, welcome))
    application.add_handler(MessageHandler(filters.TEXT & (~filters.COMMAND), filter_bad_words))
    application.add_handler(CommandHandler('ban', ban))
    application.add_handler(CommandHandler('kick', kick))
    application.add_handler(CommandHandler('mute', mute))

    application.run_polling()

if __name__ == '__main__':
    main()