import os
import random
from telegram import Update
from telegram.ext import ApplicationBuilder, CommandHandler, ContextTypes

# Fonction pour générer une grille avec des mines
def generer_grille(nb_mines=3):
    grille = ['❓'] * 25
    mines = random.sample(range(25), nb_mines)
    for i in mines:
        grille[i] = '💣'
    return grille

# Convertit la grille en texte affichable
def grille_texte(grille):
    return "\n".join([" | ".join(grille[i:i+5]) for i in range(0, 25, 5)])

# Stratégie intelligente pour choisir la meilleure case
def strategie(grille, jouees):
    priorites = [12, 7, 11, 13, 17, 6, 8, 16, 18, 1, 3, 21, 23]
    for i in priorites:
        if i not in jouees and grille[i] == '❓':
            return i
    restantes = [i for i in range(25) if i not in jouees and grille[i] == '❓']
    return random.choice(restantes) if restantes else None

# Commande /start
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    grille = generer_grille()
    context.user_data['grille'] = grille
    context.user_data['jouees'] = []
    await update.message.reply_text("🎮 Grille générée ! Tape /jouer pour une suggestion.")

# Commande /jouer
async def jouer(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if 'grille' not in context.user_data:
        await update.message.reply_text("Tu dois d'abord taper /start pour générer la grille.")
        return
    grille = context.user_data['grille']
    jouees = context.user_data['jouees']
    case = strategie(grille, jouees)
    if case is None:
        await update.message.reply_text("✅ Plus de cases sûres. Tape /start pour recommencer.")
        return
    jouees.append(case)
    if grille[case] == '💣':
        grille[case] = '💥'
        await update.message.reply_text(f"💣 Tu as touché une mine à la case {case + 1} !")
    else:
        grille[case] = '💎'
        await update.message.reply_text(f"✅ Case {case + 1} sûre !")
    await update.message.reply_text(grille_texte(grille))

# Lancement du bot
if __name__ == '__main__':
    bot_token = os.getenv("BOT_TOKEN")
    app = ApplicationBuilder().token(bot_token).build()
    app.add_handler(CommandHandler("start", start))
    app.add_handler(CommandHandler("jouer", jouer))
    app.run_polling()
