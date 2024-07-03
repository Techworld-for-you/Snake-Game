# Snake-Game
Snake Game (Python Mini Project) Developed classic Snake Game showcasing Python programming skills
import pygame
import sys
import random

# Initialize Pygame
pygame.init()

# Constants
WIDTH, HEIGHT = 640, 480
GRID_SIZE = 20
FPS = 6  # Speed of the snake
BLACK = (0, 0, 0)
# Snake class
class Snake:
    def __init__(self):
        self.length = 1
        self.positions = [((WIDTH // 2), (HEIGHT // 2))]
        self.direction = random.choice([UP, DOWN, LEFT, RIGHT])
        self.head_image = pygame.transform.scale(pygame.image.load("snake_head.png"), (GRID_SIZE + 8, GRID_SIZE + 8))
        self.body_image = pygame.transform.scale(pygame.image.load("snake_head.png"), (GRID_SIZE + 6, GRID_SIZE + 6))
        self.speed = FPS
        self.score = 0
        self.lifelines = 3

    def get_head_position(self):
        return self.positions[0]

    def update(self):
        cur = self.get_head_position()
        x, y = self.direction
        new = (((cur[0] + (x * GRID_SIZE)) % WIDTH), (cur[1] + (y * GRID_SIZE)) % HEIGHT)
        if len(self.positions) > 2 and new in self.positions[2:]:
            self.lifelines -= 1
            if self.lifelines <= 0:
                self.reset()
                return False  # Game over
            else:
                self.reset()
        else:
            self.positions.insert(0, new)
            if len(self.positions) > self.length:
                self.positions.pop()
        return True

    def reset(self):
        self.length = 1
        self.positions = [((WIDTH // 2), (HEIGHT // 2))]
        self.direction = random.choice([UP, DOWN, LEFT, RIGHT])

    def rotate_head(self):
        angle = 0
        if self.direction == UP:
            angle = 180
        elif self.direction == DOWN:
            angle = 0
        elif self.direction == LEFT:
            angle = 90
        elif self.direction == RIGHT:
            angle = 270
        self.head_image_rotated = pygame.transform.rotate(self.head_image, angle)

    def render(self, surface):
        self.rotate_head()
        surface.blit(self.head_image_rotated, (self.positions[0][0], self.positions[0][1]))
        for i, p in enumerate(self.positions[1:]):
            surface.blit(self.body_image, (p[0], p[1]))

# Fruit class
class Fruit:
    def __init__(self):
        self.position = (0, 0)
        self.image = pygame.transform.scale(pygame.image.load("fruit.png"), (GRID_SIZE + 10, GRID_SIZE + 10))

    def randomize_position(self):
        self.position = (random.randint(0, (WIDTH // GRID_SIZE) - 1) * GRID_SIZE,
                         random.randint(0, (HEIGHT // GRID_SIZE) - 1) * GRID_SIZE)

    def render(self, surface):
        surface.blit(self.image, (self.position[0], self.position[1]))

# Direction vectors
UP = (0, -1)
DOWN = (0, 1)
LEFT = (-1, 0)
RIGHT = (1, 0)

def intro_screen(screen):
    screen.fill(BLACK)
    font = pygame.font.Font(None, 36)
    text_surface = font.render("Python", True, (255, 255, 255))
    
    text_rect = text_surface.get_rect(center=(WIDTH // 2, HEIGHT // 2 - 50))
    screen.blit(text_surface, text_rect)

    text_surface = font.render("Press any key to start", True, (255, 255, 255))
    text_rect = text_surface.get_rect(center=(WIDTH // 2, HEIGHT // 2 + 50))
    screen.blit(text_surface, text_rect)

    pygame.display.flip()
    waiting_for_key = True
    while waiting_for_key:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            elif event.type == pygame.KEYDOWN:
                waiting_for_key = False

def display_score_and_lifelines(screen, snake):
    font = pygame.font.Font(None, 24)
    score_text = font.render(f"Score: {snake.score}", True, (255, 255, 255))
    screen.blit(score_text, (10, 10))

    lifeline_text = font.render(f"Lifelines: {snake.lifelines}", True, (255, 255, 255))
    screen.blit(lifeline_text, (WIDTH - 140, 10))

def pause_screen(screen):
    screen.fill(BLACK)
    font = pygame.font.Font(None, 36)
    text_surface = font.render("Game Paused", True, (255, 255, 255))
    text_rect = text_surface.get_rect(center=(WIDTH // 2, HEIGHT // 2 - 50))
    screen.blit(text_surface, text_rect)

    text_surface = font.render("Press any key to resume", True, (255, 255, 255))
    text_rect = text_surface.get_rect(center=(WIDTH // 2, HEIGHT // 2 + 50))
    screen.blit(text_surface, text_rect)

    pygame.display.flip()
    waiting_for_key = True
    while waiting_for_key:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            elif event.type == pygame.KEYDOWN:
                waiting_for_key = False

def main():
    clock = pygame.time.Clock()
    screen = pygame.display.set_mode((WIDTH, HEIGHT), 0, 32)
    surface = pygame.Surface(screen.get_size())
    surface = surface.convert()

    intro_screen(screen)

    snake = Snake()
    fruit = Fruit()
    fruit.randomize_position()

    paused = False

    while True:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_UP and snake.direction != DOWN:
                    snake.direction = UP
                elif event.key == pygame.K_DOWN and snake.direction != UP:
                    snake.direction = DOWN
                elif event.key == pygame.K_LEFT and snake.direction != RIGHT:
                    snake.direction = LEFT
                elif event.key == pygame.K_RIGHT and snake.direction != LEFT:
                    snake.direction = RIGHT
                elif event.key == pygame.K_p:
                    paused = not paused
                    if paused:
                        pause_screen(screen)

        if not paused:
            snake.update()
            if snake.get_head_position() == fruit.position:
                snake.length += 1
                snake.score += 10  # Increase score when eating fruit
                fruit.randomize_position()

            surface.fill(BLACK)
            snake.render(surface)
            fruit.render(surface)
            display_score_and_lifelines(surface, snake)
            screen.blit(surface, (0, 0))
            pygame.display.update()

        clock.tick(snake.speed)

if __name__ == "__main__":
    main()
