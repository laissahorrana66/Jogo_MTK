import pygame
import sys
import time

pygame.init()

# ================= CONFIG =================
WIDTH, HEIGHT = 900, 500
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("ANIME FIGHT - Pixel Kombat")

clock = pygame.time.Clock()
FONT = pygame.font.SysFont("pixel", 22)

# CORES
WHITE = (255,255,255)
RED = (200,0,0)
GREEN = (0,200,0)
BLACK = (0,0,0)
BLUE = (50,50,255)

# ================= PERSONAGEM =================
class Fighter:
    def __init__(self, name, x, color, power):
        self.name = name
        self.rect = pygame.Rect(x, 300, 60, 100)
        self.color = color
        self.life = 100
        self.power = power
        self.vel = 5
        self.jump = False
        self.jump_force = 10
        self.combo = []

    def draw(self):
        pygame.draw.rect(screen, self.color, self.rect)

        # Nome
        name_text = FONT.render(self.name, True, WHITE)
        screen.blit(name_text, (self.rect.x, self.rect.y - 20))

        # Vida
        pygame.draw.rect(screen, RED, (self.rect.x, self.rect.y - 10, 60, 5))
        pygame.draw.rect(screen, GREEN, (self.rect.x, self.rect.y - 10, self.life * 0.6, 5))

    def move(self, keys, left, right, jump):
        if keys[left]:
            self.rect.x -= self.vel
        if keys[right]:
            self.rect.x += self.vel
        if not self.jump and keys[jump]:
            self.jump = True

        if self.jump:
            self.rect.y -= self.jump_force
            self.jump_force -= 1
            if self.jump_force < -10:
                self.jump = False
                self.jump_force = 10

    def attack(self, enemy, dmg):
        if self.rect.colliderect(enemy.rect):
            enemy.life -= dmg

# ================= TELAS =================
def show_controls():
    screen.fill(BLACK)
    texts = [
        "ANIME FIGHT - PIXEL KOMBAT",
        "",
        "CONTROLES:",
        "A / D  - ANDAR",
        "W      - PULAR",
        "J      - SOCO",
        "K      - CHUTE",
        "L      - ESPECIAL",
        "",
        "COMBOS:",
        "J + J + K = COMBO",
        "K + J + L = COMBO ESPECIAL",
        "",
        "APERTE ENTER PARA COMEÇAR"
    ]

    for i, t in enumerate(texts):
        txt = FONT.render(t, True, WHITE)
        screen.blit(txt, (200, 50 + i * 25))

    pygame.display.update()

    waiting = True
    while waiting:
        for e in pygame.event.get():
            if e.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if e.type == pygame.KEYDOWN and e.key == pygame.K_RETURN:
                waiting = False

def choose_character():
    screen.fill(BLACK)
    options = ["GOKU", "NARUTO", "SUKUNA"]
    powers = [8, 6, 10]

    for i, o in enumerate(options):
        txt = FONT.render(f"{i+1} - {o}", True, WHITE)
        screen.blit(txt, (350, 150 + i * 40))

    pygame.display.update()

    while True:
        for e in pygame.event.get():
            if e.type == pygame.KEYDOWN:
                if e.key == pygame.K_1:
                    return Fighter("Goku", 150, BLUE, powers[0])
                if e.key == pygame.K_2:
                    return Fighter("Naruto", 150, (255,150,0), powers[1])
                if e.key == pygame.K_3:
                    return Fighter("Sukuna", 150, (150,0,0), powers[2])

# ================= MAIN =================
show_controls()

player = choose_character()
enemy = Fighter("CPU", 650, RED, 6)

running = True
while running:
    clock.tick(60)
    screen.fill((30,30,30))

    keys = pygame.key.get_pressed()

    player.move(keys, pygame.K_a, pygame.K_d, pygame.K_w)

    # ATAQUES
    if keys[pygame.K_j]:
        player.attack(enemy, player.power)
        player.combo.append("J")

    if keys[pygame.K_k]:
        player.attack(enemy, player.power + 2)
        player.combo.append("K")

    if keys[pygame.K_l]:
        if player.combo[-3:] == ["K","J","L"]:
            player.attack(enemy, 20)
        else:
            player.attack(enemy, 10)
        player.combo.clear()

    if len(player.combo) > 5:
        player.combo.clear()

    player.draw()
    enemy.draw()

    if enemy.life <= 0:
        win = FONT.render("VOCÊ VENCEU!", True, GREEN)
        screen.blit(win, (380, 200))

    pygame.display.update()

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

pygame.quit()
