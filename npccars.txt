import pygame
import random

# Initialize Pygame
pygame.init()
WIDTH, HEIGHT = 900, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("City Traffic FSM - Enhanced Animation")
clock = pygame.time.Clock()
font = pygame.font.SysFont("arial", 24)

# Colors
WHITE = (255, 255, 255)
GRAY = (200, 200, 200)
GREEN = (0, 255, 0)
RED = (255, 0, 0)
YELLOW = (255, 255, 0)
BLUE = (0, 100, 255)
BLACK = (0, 0, 0)
LAVENDER = (230, 230, 250)

# FSM Buttons
class Button:
    def __init__(self, x, y, w, h, text, action):
        self.rect = pygame.Rect(x, y, w, h)
        self.color = GRAY
        self.text = text
        self.action = action

    def draw(self):
        pygame.draw.rect(screen, self.color, self.rect, border_radius=10)
        txt = font.render(self.text, True, BLACK)
        text_rect = txt.get_rect(center=self.rect.center)
        screen.blit(txt, text_rect)

    def is_clicked(self, pos):
        return self.rect.collidepoint(pos)

# Traffic Light
class TrafficLight:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.states = ["GREEN", "YELLOW", "RED"]
        self.state_index = 0

    def next_state(self):
        self.state_index = (self.state_index + 1) % len(self.states)

    def draw(self):
        pygame.draw.rect(screen, BLACK, (self.x, self.y, 60, 160), border_radius=10)
        for i, state in enumerate(["RED", "YELLOW", "GREEN"]):
            color = GRAY
            glow_radius = 20
            if self.states[self.state_index] == state:
                color = RED if state == "RED" else YELLOW if state == "YELLOW" else GREEN
                pygame.draw.circle(screen, (color[0], color[1], color[2], 80), (self.x + 30, self.y + 30 + i * 45), glow_radius, width=0)
            pygame.draw.circle(screen, color, (self.x + 30, self.y + 30 + i * 45), 15)
        label = font.render(self.states[self.state_index], True, BLACK)
        screen.blit(label, (self.x - 10, self.y + 170))

    def current_state(self):
        return self.states[self.state_index]

# NPC Car
class Car:
    def __init__(self, x, y, color):
        self.x = x
        self.y = y
        self.color = color
        self.state = "DRIVING"
        self.speed = 3
        self.width = 60
        self.height = 30
        self.image = pygame.Surface((self.width, self.height), pygame.SRCALPHA)
        pygame.draw.ellipse(self.image, color, self.image.get_rect())

    def update(self, light_state):
        if self.state == "DRIVING":
            if light_state == "RED" and self.x < 400:
                self.state = "STOPPING"
            else:
                self.x += self.speed
        elif self.state == "STOPPING":
            if light_state == "GREEN":
                self.state = "DRIVING"
        elif self.state == "CRASHING":
            self.x = 100
            self.state = "DRIVING"

    def force_crash(self):
        self.state = "CRASHING"

    def draw(self):
        screen.blit(self.image, (self.x, self.y))
        label = font.render(self.state, True, BLACK)
        screen.blit(label, (self.x, self.y - 25))

# Initialize game objects
traffic_light = TrafficLight(420, 120)
cars = [Car(50, 300, BLUE)]

# Buttons
buttons = [
    Button(700, 100, 180, 50, "Next Traffic Light", lambda: traffic_light.next_state()),
    Button(700, 170, 180, 50, "Crash Car", lambda: cars[0].force_crash()),
    Button(700, 240, 180, 50, "Reset Car", lambda: reset_car(cars[0]))
]

def reset_car(car):
    car.x = 50
    car.state = "DRIVING"

# Game loop
running = True
while running:
    screen.fill(LAVENDER)

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        elif event.type == pygame.MOUSEBUTTONDOWN:
            for button in buttons:
                if button.is_clicked(event.pos):
                    button.action()

    for car in cars:
        car.update(traffic_light.current_state())

    # Draw background road
    pygame.draw.rect(screen, (50, 50, 50), (0, 320, WIDTH, 80))
    for i in range(40, WIDTH, 80):
        pygame.draw.line(screen, WHITE, (i, 360), (i + 40, 360), 2)

    # Draw buildings
    for i in range(0, WIDTH, 150):
        pygame.draw.rect(screen, (100, 100, 200), (i, 50, 80, 100), border_radius=5)

    traffic_light.draw()
    for car in cars:
        car.draw()
    for button in buttons:
        button.draw()

    pygame.display.flip()
    clock.tick(60)

pygame.quit()
