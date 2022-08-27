import pygame
from pygame.locals import KEYDOWN, KEYUP, K_RIGHT, K_LEFT, QUIT
from random import randint as ri
import sys

pygame.init()

SIZE = 600,600
SUR = pygame.display.set_mode(SIZE)
FPS = pygame.time.Clock()

class Obj:
    def __init__(self):
        self.x, self.y = 0, 0
        self.a = 0
    def setpic(self, pic):
        self.img = pygame.image.load(pic)
    def setsize(self, x, y):
        self.img = pygame.transform.scale(self.img, (x,y))
        self.sx, self.sy = x, y
    def show(self):
        SUR.blit(self.img, (self.x, self.y))


def crash(a,b):
    if a.x - b.sx < b.x < a.x + a.sx:
        if a.y - b.sy < b.y < a.y + a.sy:
            return True
    return False


emess = pygame.font.SysFont(None, 50) 
smess = pygame.font.SysFont(None, 40)

dlist = []
stars = []
avoids = 0
eat_star = 0
game_over = False
ghost = Obj()
ghost.setpic("goast.png")
ghost.setsize(50,80)
ghost.x = SIZE[0]//2 - ghost.sx//2
ghost.y = SIZE[1] - ghost.sy - 10
go_right, go_left = False, False

while True:
    SUR.fill((255,255,255))
    for i in pygame.event.get():
        if i.type == QUIT:
            pygame.quit()
            sys.exit()
        elif i.type == KEYUP:
            if i.key == K_RIGHT:
                go_right = False
            elif i.key == K_LEFT:
                go_left = False
        elif i.type == KEYDOWN:
            if i.key == K_RIGHT:
                go_right = True
            elif i.key == K_LEFT:
                go_left = True
    
    if not game_over:
        if go_right:
            ghost.a += 3
        elif go_left:
            ghost.a -= 3
        elif ghost.a > 0:
            ghost.a -= 3
        elif ghost.a < 0:
            ghost.a += 3


        
        ghost.x += ghost.a / 10
        if ghost.x > SIZE[0]-ghost.sx:
            ghost.x = SIZE[0]-ghost.sx
            ghost.a = 0
        if ghost.x < 0:
            ghost.x = 0
            ghost.a = 0

        if ri(1,20) == 1:
            d = Obj()
            d.setpic("shit.png")
            d.setsize(35,30)
            d.x = ri(0, SIZE[0]-d.sx)
            d.y = -d.sy
            d.a = ri(0,1)
            dlist.append(d)

        if ri(1,100) == 1:
            star = Obj()
            star.setpic("stars.png")
            star.setsize(40,40)
            star.x = ri(0, SIZE[0]-star.sx)
            star.y = -star.sy
            stars.append(star)

        for i in dlist:
            i.y += i.a
            i.a += 0.1
            if i.y > SIZE[1]:
                del dlist[dlist.index(i)]
                avoids += 1
            i.show()

        for i in stars:
            i.y += 5
            if i.y > SIZE[1]:
                del stars[stars.index(i)]
            i.show()

        for i in dlist:
            if crash(i, ghost):
                game_over = True

        for i in stars:
            if crash(i, ghost):
                eat_star += 1
                del stars[stars.index(i)]

        s = smess.render(f"SCORE {avoids*100 + eat_star*500}", True, (62,0,0))
        sx = s.get_width()
        SUR.blit(s, (SIZE[0]-sx-5, 5))
        ghost.show()
    else:
        e = emess.render(f"SCORE {avoids*100 + eat_star*500}", True, (255,0,0))
        ex, ey = e.get_width(), e.get_height()
        SUR.blit(e, (SIZE[0]//2 - ex//2, SIZE[1]//2 - ey//2))


    pygame.display.flip()
    FPS.tick(60)
    
