# BotdiscTeste
Bot de discord para tocar música em Python

import discord 
from discord.ext import commands
from youtubesearchpython import VideosSearch
import yt_dlp
import asyncio

intents = discord.Intents.all()
bot = commands.Bot(".", intents=intents)

intents = discord.Intents.default()
intents.message_content = True

bot = commands.Bot(command_prefix="!", intents=intents)

# Opções do yt-dlp
ytdlp_format_options = {
    "format": "bestaudio/best",
    "quiet": True,
    "noplaylist": True,
}

ffmpeg_options = {
    "options": "-vn"
}

ytdlp = yt_dlp.YoutubeDL(ytdlp_format_options)

@bot.event
async def on_ready():
    print("Bot inicializado com sucesso!")

def get_audio_url(url):
    info = ytdlp.extract_info(url, download=False)
    return info["url"], info["title"]


#@bot.command serve para criar um comando
@bot.command() 
async def falar(ctx:commands.Context, *,texto):
    await ctx.send(texto)

@bot.command(name="yt")
async def buscar_youtube(ctx, *, termo: str):
    """
    Comando: !yt nome da música
    """
    search = VideosSearch(termo, limit=5)
    resultados = search.result()["result"]

    if not resultados:
        await ctx.send("❌ Nenhum resultado encontrado.")
        return

    mensagem = "🎵 Resultados do YouTube:\n\n"
    for i, video in enumerate(resultados, start=1):
        titulo = video["title"]
        link = video["link"]
        duracao = video.get("duration", "N/A")
        mensagem += f"**{i}.** [{titulo}]({link}) ⏱️ `{duracao}`\n"

    await ctx.send(mensagem)

    async def play(ctx, *, busca: str):
     """
     Comando: !play nome da música ou link
     """
    if not ctx.author.voice:
        await ctx.send("❌ Você precisa estar em um canal de voz.")
        return

    canal = ctx.author.voice.channel

    if ctx.voice_client is None:
        await canal.connect()
    else:
        await ctx.voice_client.move_to(canal)

    vc = ctx.voice_client

    if vc.is_playing():
        vc.stop()

    # Busca no YouTube se não for link
    if not busca.startswith("http"):
        busca = f"ytsearch:{busca}"

    loop = asyncio.get_event_loop()
    data = await loop.run_in_executor(None, lambda: ytdlp.extract_info(busca, download=False))

    if "entries" in data:
        data = data["entries"][0]

    audio_url = data["url"]
    titulo = data["title"]

    source = discord.FFmpegPCMAudio(audio_url, **ffmpeg_options)
    vc.play(source)

    await ctx.send(f"🎶 Tocando agora: ")

@bot.command()
async def stop(ctx):
    """
    Comando: !stop
    """
    if ctx.voice_client:
        await ctx.voice_client.disconnect()
        await ctx.send("⏹️ Música parada e bot desconectado.")
    else:
        await ctx.send("❌ O bot não está em um canal de voz.")


bot.run("Chave do token aqui.")
