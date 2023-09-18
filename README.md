# robin_in_space
help
import pygame
import random
import math
import time

pygame.init()


SCREEN_WIDTH = 1600
SCREEN_HEIGHT = 800

screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("Robin In Space")

#images
title_page = pygame.image.load("title_page.png")
tutorial_page = pygame.image.load("tutorial_page.png")
background = pygame.image.load("custom_space_background.png")

robin_spaceship = pygame.image.load("robin_spaceship.png")
small_enemy_ship = pygame.image.load("small_enemy_ship.png")
boss_enemy = pygame.image.load("boss_enemy.png")

planet_1 = pygame.image.load("planet1.png")
planet_2 = pygame.image.load("planet2.png")
planet_3 = pygame.image.load("planet3.png")
planet_4 = pygame.image.load("planet4.png")
planet_5 = pygame.image.load("planet5.png")
planet_6 = pygame.image.load("planet6.png")
sun = pygame.image.load("sun.png")

asteroid_1 = pygame.image.load("asteroid_1.png")
asteroid_2 = pygame.image.load("asteroid_2.png")
asteroid_3 = pygame.image.load("asteroid_3.png")

fast_asteroid = pygame.image.load("flying_asteroid.png")

fuel = pygame.image.load("fuel.png")
health_kit = pygame.image.load("health_kit.png")
bullet_pack = pygame.image.load("bullet_pack.png")

bullet = pygame.image.load("bullet_shot.png")
enemy_bullet = pygame.image.load("enemy_bullet.png")

play_button_pic = pygame.image.load("play_button.png")
start_button_pic = pygame.image.load("start_button.png")
try_again_button_pic = pygame.image.load("try_again_button.png")

#game variables
clock = pygame.time.Clock()

score = 0

asteroid_list = [asteroid_1, asteroid_2, asteroid_3]
background_planets = [planet_1, planet_2, planet_3,planet_4, planet_5, planet_6, sun]

bg_width = background.get_width()
tiles = math.ceil(SCREEN_WIDTH / bg_width) + 1
scroll = 0
text_font = pygame.font.SysFont("Helvetica", 60)



def displayed_text(text, font, text_col, x, y):

    img = font.render(text, True, text_col)
    screen.blit(img,(x,y))


def scrolling_background():

    global scroll

    #scrolling background
    for i in range (0, tiles):

        screen.blit(background,(i * bg_width + scroll, 0))

    scroll -= 4

    #reset scroll
    if abs(scroll) > bg_width:

        scroll = 0


#planets that scroll in the background. Purely aesthetic
class Scrolling_planets(pygame.sprite.Sprite):

    def __init__(self, x, y):

        pygame.sprite.Sprite.__init__(self)
        self.image = random.choice(background_planets)
        self.rect = self.image.get_rect()
        self.rect.x = x
        self.rect.y = y

    def draw(self):

       screen.blit(self.image, self.rect)

    def update(self):

       self.rect.x -= 2

       if self.rect.right < 0:

           self.kill()


class Button():
    
    def __init__ (self, x, y, image):

        self.image = image
        self.rect = self.image.get_rect()
        self.rect.topleft = (x, y)
        self.clicked = False

    def draw(self):
        action = False
        #get mouse positions
        pos = pygame.mouse.get_pos()

        #check mouseover and clicked conditions
        if self.rect.collidepoint(pos):
            if pygame.mouse.get_pressed()[0] == 1 and self.clicked == False:
                self.clicked = True
                action = True


        if pygame.mouse.get_pressed()[0] == 0:
            self.clicked = False
        screen.blit(self.image, (self.rect.x, self.rect.y))

        return action


#gas tank class
class Gas:

    def __init__ (self, x, y, width, height, max_gas):

        self.x = x
        self.y = y
        self.width = width
        self.height = height
        self.gas = 300
        self.max_gas = max_gas


    def draw(self, surface):
        
        ratio = self.gas / self.max_gas
        
        pygame.draw.rect(surface, "red", (self.x, self.y, self.width, self.height))
        pygame.draw.rect(surface, "green", (self.x, self.y, self.width * ratio, self.height))


class Health_bar:

    def __init__(self, x, y, width, height, max_health):

        self.x = x
        self.y = y
        self.width = width
        self.height = height
        self.hp = 300
        self.max_health = max_health

    def draw(self, surface):
        
        ratio = self.hp / self.max_health
        
        pygame.draw.rect(surface, "red", (self.x, self.y, self.width, self.height))
        pygame.draw.rect(surface, "green", (self.x, self.y, self.width * ratio, self.height))




class Asteroids(pygame.sprite.Sprite):

    def __init__(self, x, y):

        pygame.sprite.Sprite.__init__(self)
        self.image = random.choice(asteroid_list)
        self.rect = self.image.get_rect()
        self.rect.x = x
        self.rect.y = y


    def draw(self):

       screen.blit(self.image, self.rect)

    def update(self):

       self.rect.x -= random.randint(5,15)

       if self.rect.colliderect(robin):

           spaceship_health.hp -= 30
           spaceship_health.draw(screen)
           self.kill()

       if self.rect.right < 0:

           self.kill()


class Flying_asteroid(pygame.sprite.Sprite):

    def __init__(self, x, y):

        pygame.sprite.Sprite.__init__(self)
        self.image = fast_asteroid
        self.rect = self.image.get_rect()
        self.rect.x = x
        self.rect.y = y


    def draw(self):

       screen.blit(self.image, self.rect)

    def update(self):

       self.rect.x -= random.randint(10,15)

       if self.rect.colliderect(robin):

           spaceship_health.hp -= 100
           spaceship_health.draw(screen)
           self.kill()


       if self.rect.right < 0:

           self.kill()


#fuel box
class Fuel(pygame.sprite.Sprite):

    def __init__ (self, x, y):

        pygame.sprite.Sprite.__init__(self)
        self.image = fuel
        self.rect = self.image.get_rect()
        self.rect.x = x
        self.rect.y = y

    def draw(self):

        screen.blit(self.image, self.rect)

    def update(self):

        self.rect.x -= 3

        if self.rect.right < 0:

            self.kill()


        if self.rect.colliderect(robin):

            gas_tank.gas += 100
            self.kill()

            if gas_tank.gas > 300:
                gas_tank.gas = 300


class Health_kit(pygame.sprite.Sprite):

    def __init__ (self, x, y):

        pygame.sprite.Sprite.__init__(self)
        self.image = health_kit
        self.rect = self.image.get_rect()
        self.rect.x = x
        self.rect.y = y

    def draw(self):

        screen.blit(self.image, self.rect)

    def update(self):

        self.rect.x -= 3

        if self.rect.right < 0:

            self.kill()

        if self.rect.colliderect(robin):

            spaceship_health.hp += 100
            self.kill()

            if spaceship_health.hp > 300:
                spaceship_health.hp = 300


class Small_enemy(pygame.sprite.Sprite):

    def __init__(self, x, y):

        pygame.sprite.Sprite.__init__(self)
        self.image = pygame.transform.scale(small_enemy_ship,(250,100))
        self.rect = self.image.get_rect()
        self.rect.x = x
        self.rect.y = y


    def draw(self):
        
        screen.blit(self.image, self.rect)

    def update(self):

        self.rect.x -= 15

        if self.rect.right < 0:

            self.kill()

        if self.rect.colliderect(robin):

            spaceship_health.hp -=100
            self.kill()



class Player_bullet(pygame.sprite.Sprite):

    def __init__(self, x, y):

        pygame.sprite.Sprite.__init__(self)
        self.image = bullet
        self.rect = self.image.get_rect()
        self.rect.x = x
        self.rect.y = y
        self.on_screen = False


    def draw(self):

        screen.blit(self.image, self.rect)
        
    
    def update(self):

        self.rect.x += 12
        self.on_screen = True

        if self.rect.left > 1600:

            self.kill()
            self.on_screen = False
      


class Player:

    def __init__(self, x, y):
  
        self.image = pygame.transform.scale(robin_spaceship,(250, 200))
        self.rect = self.image.get_rect()

        self.rect.x = x
        self.rect.y = y
        self.vel_y = 0
        
        self.width = 100
        self.height = 100
		
    def draw(self):

        screen.blit(self.image, self.rect)

    def move(self):

        scroll = 0
        dx = 0
        dy = 0

        #movement
        key = pygame.key.get_pressed()

        if key[pygame.K_RIGHT]:

            dx += 10

        if key[pygame.K_LEFT]:

            dx -= 10

        if key[pygame.K_DOWN]:

            dy += 5

        if key[pygame.K_UP]:

            dy -=5

        #bounderies

        if self.rect.x < 0:

            self.rect.x = 0

        if self.rect.x > 1200:

            self.rect.x = 1200


        if self.rect.y < 0:

            self.rect.y = 0

        if self.rect.y > 550:
            self.rect.y = 550

        self.rect.x += dx       
        self.rect.y += dy


#health and gass instances
spaceship_health = Health_bar(900,750,180,80,300)
gas_tank = Gas(300,750,180,80,300)

#gas depletion
gas_depletion = pygame.USEREVENT + 0
pygame.time.set_timer(gas_depletion, 5000)

#player instance
robin = Player(400,400)

#asteroid instance
asteroid_group = pygame.sprite.Group()
space_rocks = Asteroids(1600,random.randint(0,600))
asteroid_group.add(space_rocks)

#spawn random asteroid
asteroid_spawner = pygame.USEREVENT + 1
pygame.time.set_timer(asteroid_spawner, random.randint(800,2000))

#spawn fast random asteroid
fast_asteroid_spawner = pygame.USEREVENT + 2
pygame.time.set_timer(fast_asteroid_spawner, random.randint(10000,30000))

#fast random asteroid instance
fast_asteroid_group = pygame.sprite.Group()
fast_space_rock = Flying_asteroid(1600,random.randint(0,600))
fast_asteroid_group.add(fast_space_rock)

#button instances
start_button = Button(550,500, start_button_pic)
play_button = Button(1000,600, play_button_pic)

#fuel instance
fuel_tank_group = pygame.sprite.Group()
fuel_tank = Fuel(1600,random.randint(0,700))
fuel_tank_group.add(fuel_tank)

#spawn fuel tanks
fuel_tank_spawner = pygame.USEREVENT + 3
pygame.time.set_timer(fuel_tank_spawner, random.randint(10000, 15000))

#health kit instance
health_kit_group = pygame.sprite.Group()
health_kit_box = Health_kit(1600,random.randint(0,700))
health_kit_group.add(health_kit_box)

#spawn health kits
health_kit_spawner = pygame.USEREVENT + 4
pygame.time.set_timer(health_kit_spawner, random.randint(10000, 15000))

#scrolling planets instance
scrolling_planets_group = pygame.sprite.Group()
planets = Scrolling_planets(1600, random.randint(0,700))
scrolling_planets_group.add(planets)

#scrolling planets spawner
scrolling_planets_spawner = pygame.USEREVENT + 5
pygame.time.set_timer(scrolling_planets_spawner, 25000)

#small enemy ship spawner
scrolling_enemy_spawner = pygame.USEREVENT + 6
pygame.time.set_timer(scrolling_enemy_spawner, 10000)

#small enemy ship instance
small_enemy_group = pygame.sprite.Group()
tiny_spaceship = Small_enemy(1600,0)
small_enemy_group.add(tiny_spaceship)

#player bullet instance
player_bullet_group = pygame.sprite.Group()
bullets_shot = Player_bullet(robin.rect.x, robin.rect.x)
player_bullet_group.add(bullets_shot)


#start menu
def start_menu():


    run = True

    while run:

        screen.blit(title_page,(0,0))
        if start_button.draw() == True:

            tutorial()




        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                run = False

        pygame.display.update()
    

    pygame.quit()

def tutorial():

    run = True

    while run:


        screen.blit(tutorial_page,(0,0))
        if play_button.draw() == True:

            game()
    

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                run = False

        pygame.display.update()
    

    pygame.quit()


def game():

    global bullets_shot

    run = True

    #time
    clock.tick(60)
    start_time = time.time()
    elasped_time = 0
    
  

    while run:

        score = 0
        
        scrolling_background()

        elasped_time = round(time.time() - start_time)

        for space_rocks in asteroid_group:


            if pygame.sprite.spritecollide(bullets_shot,asteroid_group, True, collided=None):

                #RIGHT HERE. if i uncomment out line 583 the bug happens

                #bullets_shot.kill()
                bullets_shot.on_screen = False


        for event in pygame.event.get():

            if event.type == pygame.KEYDOWN:

                if event.key == pygame.K_SPACE and bullets_shot.on_screen == False:
                  
                    bullets_shot = Player_bullet(robin.rect.x + 200, robin.rect.y + 80)
                    player_bullet_group.add(bullets_shot)



            if event.type == gas_depletion:

                gas_tank.gas -= 20
                gas_tank.draw(screen)

            if event.type == asteroid_spawner:
                
                space_rocks = Asteroids(1600,random.randint(0,600))
                asteroid_group.add(space_rocks)


                            
            if event.type == fast_asteroid_spawner:

                fast_space_rock = Flying_asteroid(1600,random.randint(0,600))
                fast_asteroid_group.add(fast_space_rock)
                
            if event.type == fuel_tank_spawner:

                fuel_tank = Fuel(1600,random.randint(0,800))
                fuel_tank_group.add(fuel_tank)

            if event.type == health_kit_spawner:
                
                health_kit_box = Health_kit(1600,random.randint(0,200))
                health_kit_group.add(health_kit_box)           
                
            if event.type == scrolling_planets_spawner:

                planets = Scrolling_planets(1600, random.randint(0,800))
                scrolling_planets_group.add(planets)

            if event.type == scrolling_enemy_spawner:

                tiny_spaceship = Small_enemy(1600,0)
                small_enemy_group.add(tiny_spaceship)


            if event.type == pygame.QUIT:
                run = False



        scrolling_planets_group.draw(screen)
        scrolling_planets_group.update()

        fuel_tank_group.draw(screen)
        fuel_tank_group.update()

        player_bullet_group.draw(screen)
        player_bullet_group.update()

        robin.draw()
        robin.move()

        gas_tank.draw(screen)

        spaceship_health.draw(screen)

        asteroid_group.draw(screen)
        asteroid_group.update()

        fast_asteroid_group.draw(screen)
        fast_asteroid_group.update()

        health_kit_group.draw(screen)
        health_kit_group.update()

        small_enemy_group.draw(screen)
        small_enemy_group.update()

        
        displayed_text("Time: {} ".format(elasped_time), text_font, "white", 0,0)
        displayed_text("Gas", text_font, "white", 50, 740)
        displayed_text("Health", text_font, "white", 600, 740)
        displayed_text("Score: ", text_font, "white", 1350, 0)


        pygame.display.update()
    

    pygame.quit()


run = True

while run:

    start_menu()

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            run = False

    pygame.display.update()
    

pygame.quit()

