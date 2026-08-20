import random
import time
import os
import threading

# Characters -
goku = ("Goku", "Dragon Ball", 100, 98, 80, 99, 85)
vegeta = ("Vegeta", "Dragon Ball", 99, 97, 85, 98, 82)
saitama = ("Saitama", "One Punch Man", 100, 99, 70, 85, 80)
gojo = ("Gojo", "Jujutsu Kaisen", 97, 96, 96, 99, 98)
sukuna = ("Sukuna", "Jujutsu Kaisen", 99, 96, 94, 100, 88)
luffy = ("Luffy", "One Piece", 96, 93, 75, 98, 82)
zoro = ("Zoro", "One Piece", 94, 91, 70, 94, 87)
sanji = ("Sanji", "One Piece", 90, 97, 86, 93, 91)
naruto = ("Naruto", "Naruto", 96, 94, 82, 96, 86)
sasuke = ("Sasuke", "Naruto", 95, 96, 91, 97, 94)
madara = ("Madara", "Naruto", 99, 94, 95, 99, 90)
itachi = ("Itachi", "Naruto", 91, 92, 98, 100, 96)
levi = ("Levi", "Attack on Titan", 82, 99, 88, 100, 97)
mikasa = ("Mikasa", "Attack on Titan", 80, 95, 82, 94, 96)
eren = ("Eren", "Attack on Titan", 94, 82, 88, 91, 85)
tanjiro = ("Tanjiro", "Kimetsu no Yaiba", 84, 88, 80, 92, 91)
rengoku = ("Rengoku", "Kimetsu no Yaiba", 88, 90, 78, 94, 94)
akaza = ("Akaza", "Kimetsu no Yaiba", 92, 94, 85, 97, 90)
killua = ("Killua", "Hunter x Hunter", 86, 98, 90, 96, 95)
hisoka = ("Hisoka", "Hunter x Hunter", 88, 93, 92, 98, 97)
meruem = ("Meruem", "Hunter x Hunter", 99, 94, 100, 100, 84)
edward_elric = ("Edward Elric", "Fullmetal Alchemist", 82, 84, 96, 93, 88)
light_yagami = ("Light Yagami", "Death Note", 35, 60, 100, 95, 93)
lelouch = ("Lelouch", "Code Geass", 40, 55, 100, 98, 94)
thorfinn = ("Thorfinn", "Vinland Saga", 83, 91, 82, 93, 89)
guts = ("Guts", "Berserk", 94, 89, 78, 97, 86)
ichigo = ("Ichigo", "Bleach", 97, 96, 80, 96, 90)
aizen = ("Aizen", "Bleach", 98, 93, 100, 100, 94)
mob = ("Mob", "Mob Psycho 100", 98, 88, 82, 89, 80)
denji = ("Denji", "Chainsaw Man", 89, 87, 65, 84, 86)

characters = [goku, vegeta, saitama, gojo, sukuna,
    luffy, zoro, sanji, naruto, sasuke,
    madara, itachi, levi, mikasa, eren,
    tanjiro, rengoku, akaza, killua, hisoka,
    meruem, edward_elric, light_yagami, lelouch,
    thorfinn, guts, ichigo, aizen, mob, denji
]

# Game Config
BUDGET = 1000
TEAM_SIZE = 5
HUMAN_PLAYERS = 3 # Change according to players
BOT_PLAYERS = 0
TOTAL_PLAYERS = HUMAN_PLAYERS + BOT_PLAYERS
MIN_INCREMENT = 50
BID_TIME_LIMIT = 30
SECRET_MONEY = False
LIFELINE_COST = 10 # Lifeline ka price

def clear():
    os.system('cls' if os.name == 'nt' else 'clear')

def calc_power(char):
    return sum(char[2:])

def calc_starting_bid(char):
    return 100 # Fixed base price sabke liye

def show_rules():
    clear()
    print("="*55)
    print("🎮 ANIME AUCTION - GAME RULE'S ('PLAY FAIRLY')🎮")
    print("="*55)
    print("1. Har player: ₹1000 budget, 5 players ki team banao")
    print("2. Character ke sirf 5 stats dikhenge, name/anime nahi dikhega!")
    print("3. Base bid ₹100, min ₹50 increment, 30s time limit")
    print(f"4. LIFELINE: 1 baar per game ₹{LIFELINE_COST} me full reveal, sirf tumhe dikhega")
    print("5. RANDOM EVENTS: Round 3,6,9... me bonus/penalty")
    print("6. Team full ya paise khatam = auction se bahar")
    print("7. End me sabse zyada Total Power wali team jeetegi")
    print("="*55)
    input("\nPress Enter to start auction...")

def input_with_timeout(prompt, timeout):
    result = [None]
    def get_input():
        result[0] = input(prompt)
    thread = threading.Thread(target=get_input)
    thread.daemon = True
    thread.start()
    thread.join(timeout)
    if thread.is_alive():
        print("\n⏰ Time Up! Auto PASS")
        return 'q'
    return result[0]

def bot_decision(player_id, money, current_bid, char_power, team_full):
    if team_full or money < current_bid + MIN_INCREMENT:
        return 'q'
    max_willing = money * 0.4 if char_power > 450 else money * 0.25
    if current_bid < max_willing and random.random() < 0.8:
        bid = current_bid + random.randint(MIN_INCREMENT, MIN_INCREMENT * 5)
        return str(min(bid, money))
    return 'q'

# Game Setup
player_teams = {i: [] for i in range(1, TOTAL_PLAYERS + 1)}
player_money = {i: BUDGET for i in range(1, TOTAL_PLAYERS + 1)}
player_types = {i: 'human' if i <= HUMAN_PLAYERS else 'bot' for i in range(1, TOTAL_PLAYERS + 1)}
player_stats = {i: {'spent': 0, 'wins': 0, 'max_bid': 0} for i in range(1, TOTAL_PLAYERS + 1)}
lifeline_used = {i: False for i in range(1, TOTAL_PLAYERS + 1)}

available_chars = characters.copy()
random.shuffle(available_chars)

clear()
print("=" * 55)
print(" ANIME BLIND AUCTION - FULL MYSTERY MODE v4.0")
print("=" * 55)
show_rules()

# Main Game Loop
round_num = 1
while any(len(player_teams[p]) < TEAM_SIZE for p in range(1, TOTAL_PLAYERS + 1)):
    if not available_chars:
        print("Characters khatam!")
        break

    current_char = available_chars.pop()
    char_power = calc_power(current_char)
    starting_bid = calc_starting_bid(current_char)
    bonus_power = 0

    # Random Event Check - Har 3rd round
    event_msg = ""
    if round_num % 3 == 0:
        event = random.choice(['DOUBLE_MONEY', 'HALF_PRICE', 'BONUS_POWER'])
        if event == 'DOUBLE_MONEY':
            event_msg = "\n💰 LUCKY EVENT: Sabko ₹200 bonus mila!"
            for p in player_money: player_money[p] += 200
        elif event == 'HALF_PRICE':
            event_msg = "\n🔥 HALF PRICE ROUND: Is character ki base bid ₹50!"
            starting_bid = 50
        elif event == 'BONUS_POWER':
            event_msg = "\n❓ MYSTERY BONUS: Is character ke saath +30 power free!"
            bonus_power = 30

    clear()
    print(f"\n{'='*15} ROUND {round_num} {'='*15}")
    if event_msg: print(event_msg)
    print("🔥 MYSTERY CHARACTER FOR AUCTION 🔥")
    print("Guess kaun hai? Sirf only stats dhke bolo!")
    print(f"STR: {current_char[2]} | SPD: {current_char[3]} | IQ: {current_char[4]} | PWR: {current_char[5]} | SKL: {current_char[6]}")
    print("(Name, Anime, Total Power HIDDEN)")
    print("-" * 55)

    # Pre-round PASS option
    eligible_players = []
    for p in range(1, TOTAL_PLAYERS + 1):
        if len(player_teams[p]) >= TEAM_SIZE or player_money[p] < starting_bid:
            continue
        if player_types[p] == 'human':
            lifeline_status = "✅ Available" if not lifeline_used[p] else "❌ Used"
            print(f"\nPlayer {p} | Money: ₹{player_money[p]} | Team: {len(player_teams[p])}/{TEAM_SIZE} | Lifeline: {lifeline_status}")
            choice = input("Is character ke liye auction me baithega? y/n: ").lower()
            if choice == 'y':
                eligible_players.append(p)
        else:
            if random.random() < 0.9:
                eligible_players.append(p)
                print(f"Bot Player {p} is IN")
            else:
                print(f"Bot Player {p} passed")

    if not eligible_players:
        print("\n😢 Koi nahi khela, UNSOLD!")
        time.sleep(2)
        round_num += 1
        continue

    # Auction Start
    current_bid = starting_bid - MIN_INCREMENT
    current_winner = None
    active_players = eligible_players.copy()

    while len(active_players) > 1:
        for player in active_players[:]:
            if player not in active_players or len(active_players) == 1:
                continue

            clear()
            print(f"\n{'='*15} ROUND {round_num} - BIDDING WAR {'='*15}")
            print(f"Mystery Stats: STR:{current_char[2]} SPD:{current_char[3]} IQ:{current_char[4]} PWR:{current_char[5]} SKL:{current_char[6]}")
            print(f"Current Bid: ₹{current_bid} by Player {current_winner if current_winner else 'None'}")
            print(f"Starting Bid: ₹{starting_bid}")
            print("-" * 55)

            for p in range(1, TOTAL_PLAYERS + 1):
                status = "ACTIVE" if p in active_players else "OUT"
                money_display = f"₹{player_money[p]}" if not SECRET_MONEY or p <= HUMAN_PLAYERS else "₹???"
                print(f"Player {p} ({player_types[p]}): {money_display} | {len(player_teams[p])}/{TEAM_SIZE} | {status}")

            min_next_bid = current_bid + MIN_INCREMENT
            if player_money[player] < min_next_bid:
                print(f"\nPlayer {player} ke paas paise kam hai! Auto QUIT")
                active_players.remove(player)
                time.sleep(1.5)
                continue

            # Get action
            if player_types[player] == 'human':
                lifeline_text = f" | 'L' Lifeline ₹{LIFELINE_COST}" if not lifeline_used[player] else ""
                prompt = f"\nPlayer {player} | ₹{min_next_bid}+ bid | 'q' quit{lifeline_text} [{BID_TIME_LIMIT}s]: "
                action = input_with_timeout(prompt, BID_TIME_LIMIT)

                # Lifeline Logic - PRIVATE REVEAL + ₹10 COST
                if action and action.lower() == 'l' and not lifeline_used[player]:
                    if player_money[player] < LIFELINE_COST:
                        print(f"\n❌ Gareeb hai kya? Lifeline ke liye ₹{LIFELINE_COST} chahiye.")
                        print(f"Tere paas hai sirf ₹{player_money[player]}")
                        time.sleep(2)
                        continue

                    player_money[player] -= LIFELINE_COST
                    lifeline_used[player] = True
                    clear()
                    print(f"\n🔮 PRIVATE LIFELINE - PLAYER {player} ONLY 🔮")
                    print(f"💰 ₹{LIFELINE_COST} Deducted | New Balance: ₹{player_money[player]}")
                    print("="*55)
                    print(f"SECRET REVEAL: {current_char[0]} from {current_char[1]}")
                    print(f"Stats: STR:{current_char[2]} | SPD:{current_char[3]} | IQ:{current_char[4]} | PWR:{current_char[5]} | SKL:{current_char[6]}")
                    print(f"TOTAL POWER: {char_power + bonus_power}")
                    print("="*55)
                    print("\n⚠️ Ye info sirf tumhare liye hai. Screen chupao!")
                    input("\nPress Enter to hide and continue bidding...")
                    clear()
                    continue

                if action is None or action.lower() == 'q':
                    action = 'q'
            else:
                time.sleep(1)
                action = bot_decision(player, player_money[player], current_bid, char_power, False)
                print(f"\nBot Player {player} chooses: {action}")

            if action == 'q':
                print(f"Player {player} QUIT")
                active_players.remove(player)
                time.sleep(1)
            else:
                try:
                    bid_amount = int(action)
                    if bid_amount >= min_next_bid and bid_amount <= player_money[player]:
                        current_bid = bid_amount
                        current_winner = player
                        print(f"Player {player} bids ₹{current_bid}!")
                        time.sleep(1)
                    else:
                        print(f"Invalid! ₹{min_next_bid} se ₹{player_money[player]} tak")
                        time.sleep(1.5)
                except:
                    print("Invalid input, QUIT kar diya")
                    active_players.remove(player)
                    time.sleep(1.5)

    # Final winner
    if current_winner and current_bid >= starting_bid:
        # Bonus Power Event apply karo
        final_char = list(current_char)
        final_char[5] += bonus_power # PWR stat me add
        final_power = char_power + bonus_power

        player_money[current_winner] -= current_bid
        player_teams[current_winner].append(tuple(final_char))
        player_stats[current_winner]['spent'] += current_bid
        player_stats[current_winner]['wins'] += 1
        player_stats[current_winner]['max_bid'] = max(player_stats[current_winner]['max_bid'], current_bid)

        print(f"\n🎉 SOLD! Player {current_winner} bought for ₹{current_bid}")
        print(f"Revealed: {current_char[0]} from {current_char[1]} - Power {final_power}")
        if bonus_power > 0:
            print(f"✨ BONUS POWER +{bonus_power} Applied!")
    else:
        print(f"\n😢 UNSOLD! No valid bids above ₹{starting_bid}")

    round_num += 1
    input("\nPress Enter for next round...")

# Final Results
clear()
print("\n" + "="*55)
print(" FINAL STANDINGS + STATS")
print("="*55)

def calc_team_power(team):
    return sum(sum(char[2:]) for char in team)

final_scores = []
all_bought = []
for player in range(1, TOTAL_PLAYERS + 1):
    team = player_teams[player]
    power = calc_team_power(team)
    all_bought.extend(team)
    final_scores.append((player, power, player_money[player]))

    print(f"\nPlayer {player} ({player_types[player]}) | Final Power: {power} | Money Left: ₹{player_money[player]}")
    print(f"Stats: Spent ₹{player_stats[player]['spent']} | Wins: {player_stats[player]['wins']} | Max Bid: ₹{player_stats[player]['max_bid']} | Lifeline: {'Used' if lifeline_used[player] else 'Saved'}")
    for char in team:
        print(f" - {char[0]} ({char[1]}) - {calc_power(char)}")

# MVP Character
if all_bought:
    mvp = max(all_bought, key=calc_power)
    print(f"\n🌟 MVP CHARACTER: {mvp[0]} ({mvp[1]}) - {calc_power(mvp)} Power! 🌟")

final_scores.sort(key=lambda x: x[1], reverse=True)
winner = final_scores[0][0]
print(f"\n👑 CHAMPION: PLAYER {winner} with {final_scores[0][1]} TOTAL POWER! �
