import telebot
import random
import json
import os
import time

from telebot.util import user_link

ADMIN_ID = 6891895481

from telebot import types
from telegram._utils import markup

# Botni kuchaytirish uchun boshlang'ich ma'lumotlar
start_level = 1
start_mrcoins = 50
bonus_percentage = 40

API_TOKEN = '8036170668:AAHUUI0lKHaUc0Pj5Xom1hi4EtL3278CKG0'

CHANNELS = [
    '@Muminov_designer_channel',  # Birinchi kanal
    '@mominov_designer_chat',          # Ikkinchi kanal
    # '@+AO4p3PusGmRjMDFi',          # Uchinchi kanal
    # '@+rkLj-mUp_rYwYWIy'           # To'rtinchi kanal
]


# Botni kuchaytirish uchun boshlang'ich ma'lumotlar
start_level = 1
start_mrcoins = 50
bonus_percentage = 40

# Asosiy menyu📞 Aloqa
def main_menu():
    """Asosiy menyu tugmalari."""
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True, one_time_keyboard=False)
    markup.row('🧮 Oddiy masallar', '📘 Murakkab masallar')
    markup.row('👤 Profilim', '💰 Kunlik bonus')
    markup.row( '📞 Aloqa', '💾 Haqida')  # Random coin tugmasi qo'shildi
   # markup.row('🎲 Random coin')
    return markup
     # Foydali kanallar tugmasi qo'shildi

    return markup


bot = telebot.TeleBot(API_TOKEN)

user_data = {

}


#
# # Foydalanuvchi tangalarini olish funksiyasi
# def get_random_coins():
#     if random.randint(1, 100) <= 11:  # 11% ehtimol bilan 40-50 oralig'idan tanga chiqadi
#         return random.randint(40, 50)
#     else:
#         return random.randint(1, 39)
#
# # "🎲 Random coin" tugmasi uchun handler
# @bot.message_handler(func=lambda message: message.text == "🎲 Random coin")
# def random_bonus(message):
#     user_id = message.from_user.id
#     user = user_data.get(user_id, {'coins': 0, 'last_bonus_time': None, 'random_coins': 0})
#     last_time = user['last_bonus_time']
#
#     if last_time is None or time.time() - last_time >= 86400:  # 24 soat o'tgan bo'lsa
#         coins = get_random_coins()
#         user['coins'] += coins
#         user['random_coins'] += coins  # Random coinlar sonini saqlaymiz
#         user['last_bonus_time'] = time.time()
#         user_data[user_id] = user
#
#         bot.send_message(
#             message.chat.id,
#             f"🎉 Tabriklaymiz! Siz {coins} ta tanga yutib oldingiz. Sizning umumiy tangalaringiz: {user['coins']}."
#         )
#     else:
#         hours, minutes, seconds = time_remaining(last_time)
#         bot.send_message(
#             message.chat.id,
#             f"📅 Siz tangalarni olish uchun {hours} soat, {minutes} daqiqa, {seconds} soniya kutishingiz kerak."
#         )




# Foydalanuvchining barcha kanallarga obuna bo'lganligini tekshirish
def check_all_subscriptions(user_id):
    not_subscribed = []
    for channel in CHANNELS:
        try:
            member = bot.get_chat_member(channel, user_id)
            # Foydalanuvchi obuna bo'lgan statuslarni tekshiramiz
            if member.status not in ['member', 'administrator', 'creator']:
                not_subscribed.append(channel)
        except telebot.apihelper.ApiException:
            not_subscribed.append(channel)  # Kanalni tekshirishda xato bo'lsa, obuna bo'lmagan deb hisoblanadi
    return not_subscribed


# Foydalanuvchini kanallarga obuna bo'lishga yo'naltirish
def send_subscription_request(chat_id, not_subscribed):
    markup = types.InlineKeyboardMarkup()
    for channel in not_subscribed:
        channel_button = types.InlineKeyboardButton("Obuna Bo'ling", url=f"https://t.me/{channel[1:]}")
        markup.add(channel_button)

    check_button = types.InlineKeyboardButton("✅ Tekshirish", callback_data="check_subscription")
    markup.add(check_button)

    bot.send_message(chat_id,
                     "Quyidagi kanallarga obuna bo'ling va \"✅ Tekshirish\" tugmasini bosing:",
                     reply_markup=markup)


# "/start" komandasi uchun handler
@bot.message_handler(commands=['start'])
def start(message):
    user_id = message.from_user.id
    not_subscribed = check_all_subscriptions(user_id)

    if not not_subscribed:
        bot.send_message(message.chat.id,
                         "Assalomu Alaykum! Matematika Botga xush kelibsiz 😊",
                         reply_markup=main_menu())  # `main_menu()` funksiyasini o'zingiz yarating
    else:
        send_subscription_request(message.chat.id, not_subscribed)


# Tekshirish tugmasi uchun handler
@bot.callback_query_handler(func=lambda call: call.data == "check_subscription")
def check_subscription(call):
    user_id = call.from_user.id
    not_subscribed = check_all_subscriptions(user_id)

    if not not_subscribed:
        bot.answer_callback_query(call.id, "Siz barcha kanallarga obuna bo'lgansiz!")
        bot.edit_message_reply_markup(call.message.chat.id, call.message.message_id, reply_markup=None)
        bot.send_message(call.message.chat.id,
                         "Assalomu Alaykum! Matematika Botga xush kelibsiz 😊",
                         reply_markup=main_menu())  # `main_menu()` funksiyasini o'zingiz yarating
    else:
        bot.answer_callback_query(call.id, "Hali barcha kanallarga obuna bo'lmagansiz!")
        send_subscription_request(call.message.chat.id, not_subscribed)


@bot.message_handler(func=lambda message: message.text == '💾 Haqida')
def send_info(message):
    info = """
    ⚖️ **Bot Haqida** ⚖️

    1️⃣ **Matematik Masallar**:
    - Oddiy va murakkab matematik masallarni yechib, **mrcoin** yutib oling.

    2️⃣ **Random Coin**:
    - Har safar tasodifiy miqdorda **mrcoin** yutish imkoniyati mavjud. (Bu xususiyat hali tayyor emas)

    3️⃣ **Profil**:
    - Foydalanuvchi profilini ko'rish va o'zgartirish imkoniyatlari mavjud.

    4️⃣ **Aloqa**:
    - Foydalanuvchi bilan bog'lanish uchun bu tugmani bosing.

    5️⃣ **Orqaga**:
    - Bosh menyuga qaytish uchun bu tugmani bosing.
    
    6️⃣ **Tez kunda yangi bo'lim:**
    - Agar siz 10.000 mrcoin yig'sangiz, **Ayriboshlash** degan bo'lim ochiladi. 
       🚧 (Tez kunda ...)

    **Tez kunda Shop bo'limi ochiladi!** 

    🪙 **mrcoin haqida** 🪙

    - **mrcoin** — bu botda ishlatiladigan virtual tanga.

    🔹 **Qanday ishlaydi?**
    - Har bir masala yoki bonus orqali **mrcoin** yutib olasiz.

    🔹 **Foydalari:**
    - **mrcoin** yordamida botda o'zingizga mahsulotlar sotib olishingiz mumkin (hozircha mahsulotlar mavjud emas, lekin kelajakda yangilanishlar kutilmoqda).

    🔹 **Maxfiy imkoniyatlar:**
    - **mrcoin** yig'ish va undan foydalanish o'yin tajribangizni yaxshilaydi va ko'proq mukofotlar olish imkoniyatini yaratadi.

                Ko'proq mrcoin yig'ing! 😊
    """
    bot.send_message(message.chat.id, info, parse_mode="Markdown", reply_markup=main_menu())


# Foydalanuvchi ma'lumotlari uchun lug'at
data_file = 'user_data.json'
user_data = {}

def save_user_data():
    with open(data_file, 'w') as file:
        json.dump(user_data, file)

def load_user_data():
    global user_data
    try:
        with open(data_file, 'r') as file:
            user_data = json.load(file)
    except FileNotFoundError:
        user_data = {}

load_user_data()

# Oddiy va murakkab masallarni yaratish
def generate_problem(difficulty):
    """Masala yaratish (oddiy yoki murakkab)."""
    if difficulty == 'easy':
        num1 = random.randint(1, 20)
        num2 = random.randint(1, 20)
    elif difficulty == 'hard':
        num1 = random.randint(10, 100)
        num2 = random.randint(10, 100)

    operator = random.choice(['+', '-', '*', '/'])

    # Sonlarni to'g'ri tartibda joylashtirish (katta-kichik)
    if operator == '-' and num1 < num2:
        num1, num2 = num2, num1
    elif operator == '/':
        num1 = num1 * num2  # Bo'linmada qoldiq chiqmasligi uchun

    question = f"{num1} {operator} {num2}"
    answer = round(eval(question), 2)  # Javobni yaxlitlash
    return question, answer

# Yangi masala menyusi
def problem_menu():
    """Yangi masala uchun menyu."""
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True)
    markup.add('Yangi misol', 'Orqaga')
    return markup

# Foydalanuvchiga matematik masalalar berish
@bot.message_handler(func=lambda message: message.text in ['🧮 Oddiy masallar', '📘 Murakkab masallar'])
def give_problem(message):
    """Foydalanuvchiga masala berish."""
    difficulty = 'easy' if message.text == '🧮 Oddiy masallar' else 'hard'
    question, answer = generate_problem(difficulty)

    user_data[str(message.chat.id)] = user_data.get(str(message.chat.id), {})
    user_data[str(message.chat.id)]['current_answer'] = answer
    user_data[str(message.chat.id)]['last_menu'] = message.text
    save_user_data()

    bot.send_message(message.chat.id, f"Masala: {question}\nJavobni yuboring.", reply_markup=problem_menu())

# Yangi misol so'rash
@bot.message_handler(func=lambda message: message.text == 'Yangi misol')
def new_problem(message):
    """Yangi masala berish."""
    chat_id = str(message.chat.id)
    difficulty = 'easy' if user_data.get(chat_id, {}).get('last_menu') == '🧮 Oddiy masallar' else 'hard'
    question, answer = generate_problem(difficulty)

    user_data[chat_id]['current_answer'] = answer
    save_user_data()

    bot.send_message(message.chat.id, f"Yangi masala: {question}\nJavobni yuboring.", reply_markup=problem_menu())

# Orqaga qaytish
@bot.message_handler(func=lambda message: message.text == '🔙 Orqaga')
def go_back(message):
    """Asosiy menyuga qaytish."""
    bot.send_message(message.chat.id, "Asosiy menyu:", reply_markup=main_menu())

# Foydalanuvchi javobini tekshirish
@bot.message_handler(func=lambda message: message.text.replace('.', '', 1).isdigit())
def check_answer(message):
    """Foydalanuvchi javobini tekshirish."""
    user_answer = float(message.text)
    correct_answer = user_data.get(str(message.chat.id), {}).get('current_answer', None)

    if correct_answer is not None and abs(user_answer - correct_answer) < 0.01:
        user_data[str(message.chat.id)]['mrcoin'] = user_data[str(message.chat.id)].get('mrcoin', 0) + 9
        save_user_data()
        bot.send_message(message.chat.id, "✅ To'g'ri javob! Sizga 9 mrcoin qo'shildi.", reply_markup=main_menu())
    else:
        bot.send_message(message.chat.id, "❌ Noto'g'ri javob. Qayta urinib ko'ring!", reply_markup=problem_menu())


# Yangi masala menyusi
def problem_menu():
    """Yangi masala uchun menyu."""
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True)
    markup.add('Yangi misol', '🔙 Orqaga')
    return markup


# Reklama yuborish uchun admin funksiyasi
@bot.message_handler(func=lambda message: message.text == '📣 Reklama yuborish')
def send_advertisement(message):
    """Admin uchun reklama yuborish funksiyasi."""
    # Admin ID sini tekshirish
    admin_id = 6891895481  # O'zingizning admin ID ni bu yerga yozing

    # Admin bo'lsa, reklama yuboradi
    if message.chat.id == admin_id:
        bot.send_message(message.chat.id, "Iltimos, reklama matnini kiriting:")

        # Reklama matnini olish
        @bot.message_handler(func=lambda m: m.chat.id == admin_id)
        def send_ad(m):
            ad_text = m.text
            users = user_data.keys()  # Barcha foydalanuvchilarni olish

            # Foydalanuvchilarga yuborish
            for user in users:
                bot.send_message(user, f"📢 <b>Yangi Reklama:</b>\n\n{ad_text}", parse_mode='HTML')

            bot.send_message(admin_id, "Reklama muvaffaqiyatli yuborildi!", reply_markup=main_menu())
            return

    else:
        bot.send_message(message.chat.id, "Siz bu funksiyani ishlatishga huquqingiz yo'q.")


# Profil bo‘limi
@bot.message_handler(func=lambda message: message.text == '👤 Profilim')
def profile(message):
    """Foydalanuvchi profilini ko'rsatish."""
    user_id = str(message.chat.id)
    user = user_data.get(user_id, {})

    # Foydalanuvchilarni tartib raqami bilan chiqarish uchun ro'yxat yaratamiz
    sorted_users = sorted(user_data.items(), key=lambda x: x[1].get('mrcoin', 0), reverse=True)
    rank = next((i + 1 for i, u in enumerate(sorted_users) if u[0] == user_id), None)

    # Profil ma'lumotlarini tayyorlash
    profile_info = f"""
    
          Profil ma'lumotlari 📂 
       
    👤 Ism: {message.from_user.first_name}
    👥 Familya: {message.from_user.last_name or 'Berkitilgan 🔒'}
    📱 Telegram Username: @{message.from_user.username or 'Berkitilgan 🔒'}
    🆔 Telegram ID: {message.chat.id}
    🔢 Bot ID: {rank if rank else 'Noma'}
    🔰 Aktivlik: {'Aktiv' if user.get('mrcoin', 0) >= 20 else 'Pasiv'}
    💎 Yeg'ilgan tangalar: {user.get('mrcoin', 0)}
    """

    bot.send_message(message.chat.id, profile_info, reply_markup=main_menu())

# Foydalanuvchining bonusni qachon olganligini saqlash
  # Bu yerda ma'lumotlar bazasi yoki faylni saqlash kerak bo'ladi

# Foydalanuvchilarning ma'lumotlari (user_id asosida saqlanadi)


# Foydalanuvchi tangalarini olish funksiyasi
def get_random_coins():
    if random.randint(1, 100) <= 11:  # 11% ehtimol bilan 40-50 oralig'idan tanga chiqadi
        return random.randint(40, 50)
    else:
        return random.randint(1, 39)


# Foydalanuvchining qancha vaqt qolgani haqida ma'lumot
def time_remaining(last_time):
    remaining_time = 86400 - (time.time() - last_time)
    hours, remainder = divmod(remaining_time, 3600)
    minutes, seconds = divmod(remainder, 60)
    return int(hours), int(minutes), int(seconds)


# Aloqa bo‘limi
@bot.message_handler(func=lambda message: message.text == '📞 Aloqa')
def contact(message):
    """Aloqa ma'lumotlarini chiroyli ko'rsatish."""
    admin_telegram = "@Mr_Mominov"  # Adminning Telegram username'ini shu yerga kiriting

    contact_info = (
        f"📱 <b>Telefon:</b> +998 95 150 64 65\n\n"
        f"👤 <b>Ism:</b> Muhammadsolixon\n"
        f"👥 <b>Familya:</b> Muminov\n\n"
        f"🔗 <b>Telegram:</b> {admin_telegram}\n\n"
        f"📧 <b>Email:</b> uzsenior0@gmail.com\n"
        f"📍 <b>Manzil:</b> Farg'ona, O'zbekiston"
    )

    bot.send_message(
        message.chat.id,
        contact_info,
        reply_markup=main_menu(),
        parse_mode='HTML'
    )

# Orqaga tugmasi uchun ishlov berish
@bot.message_handler(func=lambda message: message.text == '🔙 Orqaga')
def back(message):
    """Orqaga qaytish."""
    bot.send_message(message.chat.id, "Orqaga qaytdik.", reply_markup=main_menu())


# Bot holatini ko'rsatish
@bot.message_handler(commands=['status'])
def bot_status(message):
    """Botning umumiy holatini faqat admin uchun ko'rsatish."""
    if message.chat.id == ADMIN_ID:
        total_users = len(user_data)
        active_users = len([user for user in user_data.values() if user.get('mrcoin', 0) >= 20])
        total_coins = sum(user.get('mrcoin', 0) for user in user_data.values())
        active_percentage = (active_users / total_users) * 100 if total_users > 0 else 0

        # Tangalar foizini hisoblash (1000 tanga 100% bo'ladi)
        max_coins = 10000  # 1000 tanga = 100%
        coins_percentage = (total_coins / max_coins) * 100 if total_coins <= max_coins else 100

        bot.send_message(
            message.chat.id,
            f"📊 Bot Statusi:\n\n"
            f"👤 Foydalanuvchilar soni: {total_users}\n"
            f"👥 Aktiv foydalanuvchilar: {active_users}\n"
            f"🏦 Botda yig'ilgan tangalar: {total_coins}\n"
            f"📊 Aktiv foydalanuvchilar foizi: {active_percentage:.2f}%\n"
            f"💰 Tangalar foizi: {coins_percentage:.2f}%\n"
            f"🤖 Bot holati: {'Aktiv' if active_percentage >= 50 else 'Pasiv'}"
        )
    else:
        bot.send_message(message.chat.id, "Sizda bu buyruqni ishlatish huquqi yo'q.")

from telebot.types import ReplyKeyboardMarkup, KeyboardButton

# Admin uchun reklama yuborish formatlarini tanlash
@bot.message_handler(func=lambda message: message.text == '/rek')
def send_advertisement(message):
    """Admin uchun reklama yuborish formatlarini tanlash."""
    if message.chat.id == ADMIN_ID:
        markup = ReplyKeyboardMarkup(resize_keyboard=True, one_time_keyboard=True)
        markup.row('📝 Matn reklama', '🖼️ Rasm reklama')
        markup.row('📹 Video reklama', '📄 Hujjat reklama')
        markup.row('🔙 Orqaga')  # Adding the "Back" button
        bot.send_message(
            message.chat.id,
            "Reklama formatini tanlang:\nMatn, Rasm, Video yoki Hujjat tanlang.",
            reply_markup=markup
        )
    else:
        bot.send_message(message.chat.id, "Sizda bu funksiyani ishlatish huquqi yo'q.")

# Handle the "Back" button to go back to the main menu
@bot.message_handler(func=lambda message: message.text == '🔙 Orqaga')
def go_back_to_main_menu(message):
    """Orqaga tugmasi bosilganda bosh menyuga qaytish."""
    markup = ReplyKeyboardMarkup(resize_keyboard=True, one_time_keyboard=True)
    markup.row('📝 Matn reklama', '🖼️ Rasm reklama')
    markup.row('📹 Video reklama', '📄 Hujjat reklama')
    markup.row('🔙 Orqaga')  # Keep the "Back" button in the menu
    bot.send_message(
        message.chat.id,
        "Bosh menyu.\nReklama formatini tanlang.",
        reply_markup=markup
    )

# Admin tanlagan formatga qarab reklama yuborish
@bot.message_handler(
    func=lambda message: message.text in ['📝 Matn reklama', '🖼️ Rasm reklama', '📹 Video reklama', '📄 Hujjat reklama'])
def handle_ad_format(message):
    """Admin tanlagan reklama formatiga qarab reklama yuborish."""
    ad_type = message.text

    if ad_type == '📝 Matn reklama':
        bot.send_message(message.chat.id, "Iltimos, reklama matnini kiriting:")

        @bot.message_handler(func=lambda m: m.chat.id == ADMIN_ID)
        def send_text_ad(m):
            ad_text = m.text
            users = user_data.keys()  # Barcha foydalanuvchilarni olish

            # Foydalanuvchilarga yuborish
            for user in users:
                bot.send_message(user, f"📢 <b>Yangi Matn Reklama:</b>\n\n{ad_text}", parse_mode='HTML')

            bot.send_message(ADMIN_ID, "Matn reklama muvaffaqiyatli yuborildi!")

    elif ad_type == '🖼️ Rasm reklama':
        bot.send_message(message.chat.id, "Iltimos, reklama rasm faylini yuboring:")

        @bot.message_handler(content_types=['photo'])
        def send_image_ad(m):
            ad_photo = m.photo[-1].file_id  # Eng katta rasmni olish
            bot.send_message(message.chat.id, "Iltimos, rasmga tegishli matnni kiriting:")

            # Reklama matnini olish
            @bot.message_handler(func=lambda m: m.chat.id == ADMIN_ID)
            def send_image_with_text(m):
                ad_text = m.text
                users = user_data.keys()  # Barcha foydalanuvchilarni olish

                # Foydalanuvchilarga rasmni yuborish
                for user in users:
                    bot.send_photo(user, ad_photo, caption=f"📢 Yangi Rasm Reklama:\n\n{ad_text}")

                bot.send_message(ADMIN_ID, "Rasm reklama muvaffaqiyatli yuborildi!")

    elif ad_type == '📹 Video reklama':
        bot.send_message(message.chat.id, "Iltimos, reklama video faylini yuboring:")

        @bot.message_handler(content_types=['video'])
        def send_video_ad(m):
            ad_video = m.video.file_id  # Video faylni olish
            bot.send_message(message.chat.id, "Iltimos, video haqida matnni kiriting:")

            # Reklama matnini olish
            @bot.message_handler(func=lambda m: m.chat.id == ADMIN_ID)
            def send_video_with_text(m):
                ad_text = m.text
                users = user_data.keys()  # Barcha foydalanuvchilarni olish

                # Foydalanuvchilarga video yuborish
                for user in users:
                    bot.send_video(user, ad_video, caption=f"📢 Yangi Video Reklama:\n\n{ad_text}")

                bot.send_message(ADMIN_ID, "Video reklama muvaffaqiyatli yuborildi!")

    elif ad_type == '📄 Hujjat reklama':
        bot.send_message(message.chat.id, "Iltimos, reklama hujjatini yuboring:")

        @bot.message_handler(content_types=['document'])
        def send_document_ad(m):
            ad_document = m.document.file_id  # Hujjatni olish
            bot.send_message(message.chat.id, "Iltimos, hujjat haqida matnni kiriting:")

            # Reklama matnini olish
            @bot.message_handler(func=lambda m: m.chat.id == ADMIN_ID)
            def send_document_with_text(m):
                ad_text = m.text
                users = user_data.keys()  # Barcha foydalanuvchilarni olish

                # Foydalanuvchilarga hujjat yuborish
                for user in users:
                    bot.send_document(user, ad_document, caption=f"📢 Yangi Hujjat Reklama:\n\n{ad_text}")

                bot.send_message(ADMIN_ID, "Hujjat reklama muvaffaqiyatli yuborildi!")

print("Bot ishga tushdi...")
# Botni ishga tushurish
bot.polling(none_stop=True)


