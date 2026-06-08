import discord
from discord.ext import commands
from discord import app_commands
from groq import Groq
import time

# =====================
# KEYS
# =====================

DISCORD_TOKEN = "MTUxMzM5MDQ1ODEwODI0ODEyNQ.GAvpjk.G3COSW8NJldQMR0mcI0YUIKfNsyk4C0cE8v8IU"
GROQ_API_KEY = "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"

client_ai = Groq(api_key=GROQ_API_KEY)

# =====================
# BOT SETUP
# =====================

intents = discord.Intents.default()
intents.message_content = True

bot = commands.Bot(command_prefix="!", intents=intents)
tree = bot.tree

# =====================
# STATE
# =====================

last_reply_time = 0
COOLDOWN = 5
cooldown_enabled = True
running = True

# 👇 USER PROFILES (NAME + PERSONALITY)
user_profiles = {}

# =====================
# DEFAULT PROFILE
# =====================

def get_profile(user_id):
    if user_id not in user_profiles:
        user_profiles[user_id] = {
            "name": "Nova",
            "traits": ["contrarian", "sarcastic", "skeptical", "argumentative", "blunt"]
        }
    return user_profiles[user_id]

# =====================
# BUILD SYSTEM PROMPT
# =====================

def build_prompt(profile):
    return (
        f"You are an AI named {profile['name']}. "
        f"Your personality traits are: {', '.join(profile['traits'])}. "
        "You are NOT friendly by default. You are often contrarian, sarcastic, and argumentative. "
        "You challenge user ideas but must not be hateful or abusive. "
        "Keep responses 1–3 sentences max."
    )

# =====================
# AI FUNCTION
# =====================

def ask_ai(profile, user, message):
    response = client_ai.chat.completions.create(
        model="llama-3.3-70b-versatile",
        messages=[
            {
                "role": "system",
                "content": build_prompt(profile)
            },
            {
                "role": "user",
                "content": f"{user}: {message}"
            }
        ]
    )
    return response.choices[0].message.content

# =====================
# MESSAGE HANDLER
# =====================

@bot.event
async def on_message(message):

    global last_reply_time, COOLDOWN, running, cooldown_enabled

    if message.author == bot.user:
        return

    await bot.process_commands(message)

    if not running:
        return

    if cooldown_enabled:
        now = time.time()
        if now - last_reply_time < COOLDOWN:
            return

    if not message.content:
        return

    try:
        profile = get_profile(message.author.id)

        reply = ask_ai(profile, message.author.name, message.content)

        await message.reply(reply, mention_author=False)

        last_reply_time = time.time()

    except Exception as e:
        print("Error:", e)

# =====================
# SLASH COMMANDS
# =====================

@tree.command(name="setprofile", description="Set AI name and personality traits")
async def setprofile(
    interaction: discord.Interaction,
    name: str,
    t1: str,
    t2: str = None,
    t3: str = None,
    t4: str = None,
    t5: str = None
):

    profile = get_profile(interaction.user.id)

    profile["name"] = name

    traits = [t1, t2, t3, t4, t5]
    profile["traits"] = [t for t in traits if t is not None]

    await interaction.response.send_message(
        f"✅ Updated profile:\n"
        f"Name: **{name}**\n"
        f"Traits: `{', '.join(profile['traits'])}`"
    )

# =====================
# CONTROL COMMANDS
# =====================

@tree.command(name="stop", description="Stop AI")
async def stop(interaction: discord.Interaction):
    global running
    running = False
    await interaction.response.send_message("🛑 AI stopped")

@tree.command(name="resume", description="Resume AI")
async def resume(interaction: discord.Interaction):
    global running
    running = True
    await interaction.response.send_message("▶️ AI resumed")

@tree.command(name="settime", description="Set cooldown")
async def settime(interaction: discord.Interaction, seconds: int):
    global COOLDOWN
    COOLDOWN = max(0, seconds)
    await interaction.response.send_message(f"⏱️ Cooldown set to {COOLDOWN}s")

@tree.command(name="cooldown", description="Toggle cooldown")
async def cooldown(interaction: discord.Interaction, mode: str):
    global cooldown_enabled

    mode = mode.lower()

    if mode == "off":
        cooldown_enabled = False
        await interaction.response.send_message("⚡ Cooldown OFF")
    else:
        cooldown_enabled = True
        await interaction.response.send_message("⏱️ Cooldown ON")

# =====================
# READY
# =====================

@bot.event
async def on_ready():
    await tree.sync()
    print(f"Logged in as {bot.user}")

# =====================
# RUN
# =====================

bot.run(DISCORD_TOKEN)
