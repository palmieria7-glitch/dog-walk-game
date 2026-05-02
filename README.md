import pygame
import random
import sys
import os
import asyncio  # 🔥 REQUIRED FOR PYGBAG

# -------------------------
# INIT
# -------------------------
pygame.init()

WIDTH, HEIGHT = 800, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Bone Collector")

clock = pygame.time.Clock()
FONT = pygame.font.SysFont("Arial", 24)

# -------------------------
# FIX FILE PATH
# -------------------------
BASE_DIR = os.path.dirname(os.path.abspath(__file__))

# -------------------------
# LOAD IMAGES
# -------------------------
try:
    player_img = pygame.image.load(os.path.join(BASE_DIR, "player.png")).convert_alpha()
    player_img = pygame.transform.scale(player_img, (120, 120))

    poop_img = pygame.image.load(os.path.join(BASE_DIR, "poop.png")).convert_alpha()
    poop_img = pygame.transform.scale(poop_img, (60, 60))

    bone_img = pygame.image.load(os.path.join(BASE_DIR, "bone.png")).convert_alpha()
    bone_img = pygame.transform.scale(bone_img, (25, 25))

except Exception as e:
    print("ERROR LOADING IMAGES:")
    print(e)
    pygame.quit()
    sys.exit()

# -------------------------
# PLAYER CLASS
# -------------------------
class Player:
    def __init__(self):
        self.rect = pygame.Rect(400, 300, 120, 120)
        self.speed = 5

    def move(self, keys):
        if keys[pygame.K_LEFT] or keys[pygame.K_a]:
            self.rect.x -= self.speed
        if keys[pygame.K_RIGHT] or keys[pygame.K_d]:
            self.rect.x += self.speed
        if keys[pygame.K_UP] or keys[pygame.K_w]:
            self.rect.y -= self.speed
        if keys[pygame.K_DOWN] or keys[pygame.K_s]:
            self.rect.y += self.speed

        self.rect.clamp_ip(screen.get_rect())

    def draw(self):
        screen.blit(player_img, self.rect)

# -------------------------
# BONE CLASS
# -------------------------
class Bone:
    def __init__(self):
        self.rect = pygame.Rect(
            random.randint(0, WIDTH - 25),
            random.randint(0, HEIGHT - 25),
            25,
            25
        )

    def draw(self):
        screen.blit(bone_img, self.rect)

# -------------------------
# POOP CLASS
# -------------------------
class Poop:
    def __init__(self):
        self.pos = pygame.Vector2(
            random.randint(100, WIDTH - 100),
            random.randint(100, HEIGHT - 100)
        )
        self.speed = random.uniform(1, 2)

    def move_toward_player(self, player):
        direction = pygame.Vector2(player.rect.center) - self.pos
        if direction.length() > 0:
            direction = direction.normalize()
            self.pos += direction * self.speed

    def draw(self):
        rect = poop_img.get_rect(center=self.pos)
        screen.blit(poop_img, rect)

    def get_rect(self):
        return poop_img.get_rect(center=self.pos)

# -------------------------
# GAME SETUP
# -------------------------
player = Player()
bones = [Bone() for _ in range(5)]
poops = [Poop() for _ in range(2)]

score = 0
game_over = False

# -------------------------
# MAIN GAME LOOP (ASYNC)
# -------------------------
async def main():
    global player, bones, poops, score, game_over

    while True:
        clock.tick(60)
        screen.fill((30, 30, 50))

        pygame.event.pump()  # 🔥 helps browser responsiveness

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                return

        if not game_over:
            keys = pygame.key.get_pressed()
            player.move(keys)

            for poop in poops:
                poop.move_toward_player(player)

                if player.rect.colliderect(poop.get_rect()):
                    game_over = True

            for bone in bones[:]:
                if player.rect.colliderect(bone.rect):
                    bones.remove(bone)
                    bones.append(Bone())
                    score += 1

            player.draw()

            for bone in bones:
                bone.draw()

            for poop in poops:
                poop.draw()

            text = FONT.render(f"Score: {score}", True, (255, 255, 255))
            screen.blit(text, (10, 10))

        else:
            text = FONT.render("GAME OVER - Press R to Restart", True, (255, 50, 50))
            screen.blit(text, (180, 280))

            keys = pygame.key.get_pressed()
            if keys[pygame.K_r]:
                player = Player()
                bones = [Bone() for _ in range(5)]
                poops = [Poop() for _ in range(2)]
                score = 0
                game_over = False

        pygame.display.flip()

        await asyncio.sleep(0)  # 🔥 CRITICAL FOR WEB

# -------------------------
# RUN GAME
# -------------------------
if __name__ == "__main__":
    asyncio.run(main())
