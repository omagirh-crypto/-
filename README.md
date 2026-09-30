# -*- coding: utf-8 -*-
"""
بوت متجر التميز العلمي
المتطلبات:  pip install python-telegram-bot==21.6
التشغيل:     python bot.py
"""
import json
import logging
import os
from html import escape

from telegram import (
    BotCommand, InlineKeyboardButton, InlineKeyboardMarkup,
    MenuButtonWebApp, Update, WebAppInfo,
)
from telegram.constants import ParseMode
from telegram.ext import (
    Application, CallbackQueryHandler, CommandHandler, ContextTypes,
    ConversationHandler, MessageHandler, filters,
)

# ====================== الإعدادات (عدّل هذي فقط) ======================
TOKEN = "8883229810:AAFRZyiT3tVjKwGNnPZDCiUPgaXuW2C3qtI"
ADMIN_ID = 5711820423
INSTAGRAM = "https://instagram.com/altamaez77"
STORE_PAGE = ""

PRICES_TEXT = (
    "💰 <b>الأسعار</b>\n\n"
    "الأسعار تختلف حسب نوع الخدمة وحجم العمل والموعد المطلوب.\n"
    "ارسل طلبك من زر <b>اطلب الآن</b> وراح نعطيك سعر مناسب 💚"
)
PAYMENT_TEXT = (
    "💳 <b>طريقة الدفع</b>\n\n"
    "يتم الاتفاق على طريقة الدفع وياك بعد تأكيد الطلب.\n"
    "تواصل وياانا وراح نوضحلك كل التفاصيل."
)
# ======================================================================

logging.basicConfig(level=logging.INFO)
USERS_FILE = "users.json"

SERVICES = {
            "نصمم عروضك بأسلوب احترافي وجذاب، بتنسيق مرتب وألوان متناسقة تخدم فكرتك وتبهر الحضور."),
    "trans": ("🌐 ترجمة علمية",
              "دقة في المصطلحات وسلاسة في التعبير، مع الحفاظ على المعنى الأصلي والأسلوب الأكاديمي."),
    "notes": ("📄 تصميم ملازم",
              "ملازم منظمة وواضحة بشكل مريح للعين، بتنسيق موحد وفهرسة مرتبة."),
    "reports": ("📝 تقارير وبحوث",
                "كتابة وتنسيق احترافي مع الالتزام بالمعايير الأكاديمية والتوثيق الصحيح."),
    "icons": ("🖼 تصميم أيقونات وبوسترات",
              "تصاميم للمشاريع والعروض والبوسترات العلمية بجودة عالية وأسلوب عصري."),
    "other": ("➕ خدمات أخرى",
              "عندك طلب مختلف؟ نوفره لك حسب احتياجاتك."),
}

WELCOME = (
    "أهلاً بيك في <b>التميز العلمي</b> ✨\n\n"
    "<i>لأن نجاحك يستحق التميز</i>\n\n"
    "خدمات متكاملة للطلاب والباحثين بأسلوب عصري وجودة عالية.\n"
    "اختار من القائمة 👇"
)
WHY_TXT = (
    "⭐ <b>لماذا التميز العلمي؟</b>\n\n"
    "🛡 <b>جودة عالية</b> في كل تفصيلة\n"
    "⏱ <b>التزام بالمواعيد</b> ودقة في التسليم\n"
    "🎧 <b>تواصل مستمر</b> لراحتك\n"
    "💡 <b>أسعار مناسبة</b> لجودة تليق بطموحك"
)
CONTACT_TXT = "📞 <b>تواصل معنا الآن</b>\nأفكارك.. تصير إنجازات معنا 💚"

CHOOSE, DETAILS = range(2)

# ---------------------- حفظ المشتركين ----------------------
def load_users():
    if os.path.exists(USERS_FILE):
        try:
            with open(USERS_FILE, "r", encoding="utf-8") as f:
                return set(json.load(f))
        except Exception:
            return set()
    return set()

def save_users():
    with open(USERS_FILE, "w", encoding="utf-8") as f:
        json.dump(list(USERS), f)
USERS = load_users()

async def track(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user = update.effective_user
    if user and user.id not in USERS:
        USERS.add(user.id)
        save_users()

# ---------------------- لوحات الأزرار ----------------------
def main_menu():
    rows = [
        [InlineKeyboardButton("📋 خدماتنا", callback_data="services"),
         InlineKeyboardButton("🛒 اطلب الآن", callback_data="order")],
        [InlineKeyboardButton("⭐ لماذا نحن؟", callback_data="why"),
         InlineKeyboardButton("💰 الأسعار", callback_data="prices")],
        [InlineKeyboardButton("💳 طريقة الدفع", callback_data="payment"),
         InlineKeyboardButton("📞 تواصل معنا", callback_data="contact")],
        [InlineKeyboardButton("📸 انستغرام", url=INSTAGRAM)],
    ]

def back_home():
    return InlineKeyboardMarkup([[InlineKeyboardButton("🔙 القائمة الرئيسية", callback_data="home")]])

def services_menu():
    rows = [[InlineKeyboardButton(t, callback_data=f"srv_{k}")] for k, (t, _) in SERVICES.items()]
    rows.append([InlineKeyboardButton("🔙 رجوع", callback_data="home")])
    return InlineKeyboardMarkup(rows)

def order_menu():
    rows = [[InlineKeyboardButton(t, callback_data=f"ord_{k}")] for k, (t, _) in SERVICES.items()]
    rows.append([InlineKeyboardButton("❌ إلغاء", callback_data="cancel_order")])
    return InlineKeyboardMarkup(rows)

def contact_menu():
    return InlineKeyboardMarkup([
        [InlineKeyboardButton("📸 انستغرام: altamaez77", url=INSTAGRAM)],
        [InlineKeyboardButton("🔙 رجوع", callback_data="home")],
    ])

# ---------------------- الأوامر ----------------------
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    context.user_data.clear()
    await update.message.reply_text(WELCOME, reply_markup=main_menu(), parse_mode=ParseMode.HTML)
    return ConversationHandler.END

    await update.message.reply_text("📋 <b>خدماتنا:</b>\nاختار الخدمة لتفاصيلها",
                                    reply_markup=services_menu(), parse_mode=ParseMode.HTML)

async def cmd_why(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(WHY_TXT, reply_markup=back_home(), parse_mode=ParseMode.HTML)

async def cmd_contact(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(CONTACT_TXT, reply_markup=contact_menu(), parse_mode=ParseMode.HTML)

async def cmd_prices(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(PRICES_TEXT, reply_markup=back_home(), parse_mode=ParseMode.HTML)

async def cmd_payment(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(PAYMENT_TEXT, reply_markup=back_home(), parse_mode=ParseMode.HTML)

# ---------------------- أزرار القائمة ----------------------
async def buttons(update: Update, context: ContextTypes.DEFAULT_TYPE):
    q = update.callback_query
    await q.answer()
    d = q.data
    H = ParseMode.HTML

    if d == "home":
        await q.edit_message_text(WELCOME, reply_markup=main_menu(), parse_mode=H)
    elif d == "services":
        await q.edit_message_text("📋 <b>خدماتنا:</b>\nاختار الخدمة لتفاصيلها",
                                  reply_markup=services_menu(), parse_mode=H)
    elif d.startswith("srv_"):
        title, desc = SERVICES[d[4:]]
        kb = InlineKeyboardMarkup([
            [InlineKeyboardButton("🛒 اطلب هذي الخدمة", callback_data="order")],
            [InlineKeyboardButton("🔙 رجوع", callback_data="services")],
        ])
        await q.edit_message_text(f"<b>{title}</b>\n\n{desc}", reply_markup=kb, parse_mode=H)
    elif d == "why":
        await q.edit_message_text(WHY_TXT, reply_markup=back_home(), parse_mode=H)
    elif d == "prices":
        await q.edit_message_text(PRICES_TEXT, reply_markup=back_home(), parse_mode=H)
    elif d == "payment":
        await q.edit_message_text(PAYMENT_TEXT, reply_markup=back_home(), parse_mode=H)
    elif d == "contact":
        await q.edit_message_text(CONTACT_TXT, reply_markup=contact_menu(), parse_mode=H)

# ---------------------- نظام الطلبات ----------------------
async def order_start_btn(update: Update, context: ContextTypes.DEFAULT_TYPE):
    q = update.callback_query
    await q.answer()
    await q.edit_message_text("🛒 اختار نوع الخدمة المطلوبة:", reply_markup=order_menu())
    return CHOOSE

async def order_start_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("🛒 اختار نوع الخدمة المطلوبة:", reply_markup=order_menu())
    return CHOOSE

async def order_choose(update: Update, context: ContextTypes.DEFAULT_TYPE):
    q = update.callback_query
    await q.answer()
    title = SERVICES[q.data[4:]][0]
    context.user_data["service"] = title
    await q.edit_message_text(
        f"تمام ✅ اخترت: <b>{title}</b>\n\n"
        "اكتب تفاصيل طلبك:\n"
        "• الموضوع\n• عدد الصفحات / الشرائح\n• موعد التسليم المطلوب\n\n"
        "(أو اكتب /cancel للإلغاء)",
        parse_mode=ParseMode.HTML,
    )
    return DETAILS

async def order_details(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user = update.effective_user
    service = context.user_data.get("service", "-")
    uname = f"@{user.username}" if user.username else "لا يوجد"

    msg = (
        "🔔 <b>طلب جديد</b>\n\n"
        f"👤 الاسم: {escape(user.full_name)}\n"
        f"🔗 اليوزر: {escape(uname)}\n"
        f"🆔 ID: <code>{user.id}</code>\n"
        f"🛎 الخدمة: {service}\n\n"
        f"📝 التفاصيل:\n{escape(update.message.text)}\n\n"
        f"للرد: <code>/reply {user.id} </code>"
    )
    await context.bot.send_message(ADMIN_ID, msg, parse_mode=ParseMode.HTML)
    await update.message.reply_text(
        "✅ وصلنا طلبك! راح نتواصل وياك بأقرب وقت 💚",
        reply_markup=main_menu(),
    )
    context.user_data.clear()
    return ConversationHandler.END

async def cancel(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if update.callback_query:
        await update.callback_query.answer()
        await update.callback_query.edit_message_text(WELCOME, reply_markup=main_menu(), parse_mode=ParseMode.HTML)
    else:
        await update.message.reply_text("تم إلغاء الطلب ❌", reply_markup=main_menu())
    return ConversationHandler.END

# ---------------------- أوامر الأدمن ----------------------
async def reply_user(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if update.effective_user.id != ADMIN_ID:
        return
    try:
        uid = int(context.args[0])
        text = " ".join(context.args[1:])
        if not text:
            raise ValueError
        await context.bot.send_message(uid, f"💬 <b>رد من التميز العلمي:</b>\n\n{escape(text)}",
                                       parse_mode=ParseMode.HTML)
        await update.message.reply_text("تم الإرسال ✅")
    except Exception:
        await update.message.reply_text("الصيغة: /reply رقم_الزبون النص")

async def broadcast(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if update.effective_user.id != ADMIN_ID:
        return
    text = " ".join(context.args)
        await update.message.reply_text("الصيغة: /broadcast النص")
        return
    ok = fail = 0
    for uid in list(USERS):
        try:
            await context.bot.send_message(uid, text)
            ok += 1
        except Exception:
            fail += 1
    await update.message.reply_text(f"✅ أُرسلت لـ {ok} مستخدم\n❌ فشلت: {fail}")

async def stats(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if update.effective_user.id != ADMIN_ID:
        return
    await update.message.reply_text(f"📊 عدد مستخدمي البوت: {len(USERS)}")

# ---------------------- تشغيل البوت ----------------------
async def post_init(app: Application):
    # قائمة الأوامر للزبائن (تنضبط تلقائياً بدون BotFather)
    await app.bot.set_my_commands([
        BotCommand("start", "🏠 القائمة الرئيسية"),
        BotCommand("services", "📋 عرض كل خدماتنا"),
        BotCommand("order", "🛒 اطلب خدمتك الآن"),
        BotCommand("prices", "💰 الأسعار"),
        BotCommand("payment", "💳 طريقة الدفع"),
        BotCommand("why", "⭐ لماذا التميز العلمي"),
        BotCommand("contact", "📞 تواصل معنا"),
        BotCommand("cancel", "❌ إلغاء الطلب"),
    ])
    # زر فتح صفحة المتجر
    if STORE_PAGE.startswith("https://"):
        await app.bot.set_chat_menu_button(
            menu_button=MenuButtonWebApp(text="المتجر 🛍", web_app=WebAppInfo(url=STORE_PAGE))
        )

def main():
    app = Application.builder().token(TOKEN).post_init(post_init).build()

    order_conv = ConversationHandler(
        entry_points=[
            CallbackQueryHandler(order_start_btn, pattern="^order$"),
            CommandHandler("order", order_start_cmd),
        ],
        states={
            CHOOSE: [CallbackQueryHandler(order_choose, pattern="^ord_"),
                     CallbackQueryHandler(cancel, pattern="^cancel_order$")],
            DETAILS: [MessageHandler(filters.TEXT & ~filters.COMMAND, order_details)],
        },
        fallbacks=[CommandHandler("cancel", cancel), CommandHandler("start", start)],
        allow_reentry=True,
    )

    app.add_handler(MessageHandler(filters.ALL, track), group=1)
    app.add_handler(CommandHandler("start", start))
    app.add_handler(order_conv)
    app.add_handler(CommandHandler("services", cmd_services))
    app.add_handler(CommandHandler("why", cmd_why))
    app.add_handler(CommandHandler("contact", cmd_contact))
    app.add_handler(CommandHandler("prices", cmd_prices))
    app.add_handler(CommandHandler("payment", cmd_payment))
    app.add_handler(CommandHandler("reply", reply_user))
    app.add_handler(CommandHandler("broadcast", broadcast))
    app.add_handler(CommandHandler("stats", stats))
    app.add_handler(CallbackQueryHandler(buttons))

    print("Bot is running...")
    app.run_polling()

if __name__ == "__main__":
    main()
