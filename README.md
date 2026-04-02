#Program id : Create your own game with Pygame
#Author:Tahamidul Hoque
#Purpose: TO be able to create a mini game with pygame




import pygame
import pickle
from pygame import mixer
from os import path
pygame.mixer.pre_init(44100,-16,2,512)
mixer.init()
pygame.init()

clock=pygame.time.Clock()
fps=60
win_max_x=1000
win_max_y=1000
win=pygame.display.set_mode((win_max_x,win_max_y))
pygame.display.set_caption('War Flashback')
#load image
bg=pygame.image.load('bg.jpg')
restart_img=pygame.image.load("restart.png")
start_img=pygame.image.load("start.png")
exit_img=pygame.image.load("exit.png")

#load sounds
pygame.mixer.music.load('bg_music.wav')
pygame.mixer.music.play(-1,0.0,5000)

gernade_fx=pygame.mixer.Sound('item.wav')
gernade_fx.set_volume(0.5)

jump_fx=pygame.mixer.Sound('jump.wav')
jump_fx.set_volume(0.5)

game_over_fx=pygame.mixer.Sound('game_over.wav')
game_over_fx.set_volume(0.3)

click_fx=pygame.mixer.Sound('click.wav')

#define font
font=pygame.font.SysFont('Bauhuas 93',70)
font_score=pygame.font.SysFont('Bauhuas 93',30)
#define game variables
tile_size=50
game_over=0
main_menu=True
level=0
max_level=7
score=0

#define colors
white=(255,255,255)
blue=(0,0,255)
#draw text function
def draw_text(text,font,text_col,x,y):
    img=font.render(text,True,text_col)
    win.blit(img,(x,y))
#function to reset level
def reset_level(level):
    player.reset(100,win_max_y-130)
    blob_group.empty()
    platform_group.empty()
    spike_group.empty()
    exit_group.empty()
    if path.exists(f"level{level}_data"):
        pickle_in=open(f'level{level}_data',"rb")
        world_data=pickle.load(pickle_in)   
    world=World(world_data)
    return world

class Button():
    def __init__(self,x,y,image):
        self.image=image 
        self.rect=self.image.get_rect()
        self.rect.x=x
        self.rect.y=y
        self.clicked=False
    def draw(self):
        action=False
        #get mouse position
        pos=pygame.mouse.get_pos()

        #check mouseover postions and lciked
        if self.rect.collidepoint(pos):
           if pygame.mouse.get_pressed()[0]==1 and self.clicked==False:
               action=True
               self.clicked=True
               click_fx.play()
        if pygame.mouse.get_pressed()[0]==0:
            self.clicked=False
               
        #draw button
        win.blit(self.image,self.rect)
        return action

class Player():
    def __init__(self,x,y):
        self.reset(x,y)

    def update(self,game_over):
        dx=0
        dy=0
        walk_cooldown=5
        col_thresh=20
        if game_over==0:
        #get keypresses
            key=pygame.key.get_pressed()
            if key[pygame.K_SPACE] and self.jumped==False and self.in_air==False:
                jump_fx.play()
                self.vel_y=-15
                self.jumped=True
            if key[pygame.K_SPACE]==False:
                self.jumped=False
            if key[pygame.K_LEFT]:
                dx-=5
                self.counter+=1
                self.direction=-1
                img_right = pygame.image.load("animation.move.right1.png")
                img_right = pygame.transform.scale(img_right, (40, 80))
                img_left = pygame.transform.flip(img_right, True, False)
            if key[pygame.K_RIGHT]:
                dx+=5
                self.counter += 1
                self.direction=1
            if  key[pygame.K_LEFT]==False and key[pygame.K_RIGHT]==False:
                self.counter=0
                self.direction = -1
                img_right = pygame.image.load("animation.move.right1.png")
                img_right = pygame.transform.scale(img_right, (40, 80))
                img_left = pygame.transform.flip(img_right, True, False)



            #handle animation

            #add gravity
            self.vel_y+=1
            if self.vel_y>10:
                self.vel_y=10
            dy+=self.vel_y

            # check for collision
            self.in_air=True
            for tile in world.tile_list:
            #check for collison in x direction
                if tile[1].colliderect(self.rect.x+dx, self.rect.y , self.width, self.height):
                    dx=0
                    # check for collison in y coordinate
                if tile[1].colliderect(self.rect.x,self.rect.y + dy,self.width, self.height):
                        #check if below the ground
                        if self.vel_y<0:
                            dy=tile[1].bottom-self.rect.top
                            self.vel_y=0
                        #check if above ground
                        elif self.vel_y>=0:
                            dy=tile[1].top-self.rect.bottom
                            self.vel_y = 0
                            self.in_air=False
                #check for collision with enemys
                if pygame.sprite.spritecollide(self,blob_group,False):
                    game_over=-1
                    game_over_fx.play()
                if pygame.sprite.spritecollide(self,spike_group,False):
                    game_over=-1
                    game_over_fx.play()
                if pygame.sprite.spritecollide(self,exit_group,False):
                    game_over=1
                    game_over_fx.play()
                #check colison for platform
                for platform in platform_group:
                    # collsion in x direct
                    if platform.rect.colliderect(self.rect.x+dx, self.rect.y , self.width, self.height):
                        dx=0
                        #collsion in y direct
                    if platform.rect.colliderect(self.rect.x+dx, self.rect.y+dy , self.width, self.height):
                        #check if below platform
                        if abs((self.rect.top+dy)-platform.rect.bottom)<col_thresh:
                            self.vel_y=0
                            dy=platform.rect.bottom-self.rect.top
                        #check if above platform
                        elif abs((self.rect.bottom+dy)-platform.rect.top)<col_thresh:
                            self.rect.bottom=platform.rect.top-1
                            self.in_air=False
                            dy=0
                        #move sideways with platform
                        if platform.move_x!=0:
                            self.rect.x+=platform.move_direction


            #update player coordinates
            self.rect.x+=dx
            self.rect.y+=dy
        elif game_over==-1:
            self.image=self.dead_image
            draw_text('GAME OVER!',font,blue,(win_max_x//2)-150,win_max_y//2)
            if self.rect.y>200 :
                self.rect.y-=5



        #draw player onto screen
        win.blit(self.image,self.rect)
        return game_over
    

    def reset(self,x,y):
        self.images_right=[]
        self.images_left=[]
        self.index=0
        self.counter=0
        img_right=pygame.image.load("animation.move.right1.png")
        img_right=pygame.transform.scale(img_right,(40,80))
        img_left=pygame.transform.flip(img_right,True,False)
        self.dead_image=pygame.image.load("ghost.png")
        self.images_right.append(img_right)
        self.images_right.append(img_left)
        self.image= self.images_right[self.index]
        self.rect=self.image.get_rect()
        self.rect.x=x
        self.rect.y=y
        self.width=self.image.get_width()
        self.height = self.image.get_height()
        self.vel_y=0
        self.jumped=False
        self.direction=0
        self.in_air=True
class World():
    def __init__(self,data):
        self.tile_list=[]
        #load images
        dirt_img=pygame.image.load("dirt.png")
        grass_img=pygame.image.load("grass.png")
        row_count=0
        for row in data:
            col_count=0
            for tile in row:
                if tile==1:
                    img=pygame.transform.scale(dirt_img,(tile_size,tile_size))
                    img_rect=img.get_rect()
                    img_rect.x=col_count*tile_size
                    img_rect.y=row_count*tile_size
                    tile=(img,img_rect)
                    self.tile_list.append(tile)
                if tile==2:
                    img=pygame.transform.scale(grass_img,(tile_size,tile_size))
                    img_rect=img.get_rect()
                    img_rect.x=col_count*tile_size
                    img_rect.y=row_count*tile_size
                    tile=(img,img_rect)
                    self.tile_list.append(tile)
                if tile==3:
                    blob=Enemy(col_count*tile_size,row_count*tile_size+20)
                    blob_group.add(blob)
                if tile==4:
                    platform=Platform(col_count*tile_size,row_count*tile_size,1,0)
                    platform_group.add(platform)
                if tile==5:
                    platform=Platform(col_count*tile_size,row_count*tile_size,0,1)
                    platform_group.add(platform)
                if tile==6:
                    spike=Spike(col_count*tile_size,row_count*tile_size+int(tile_size//2))
                    spike_group.add(spike)
                if tile==7:
                    gernade=Gernade(col_count*tile_size+(tile_size//2),row_count*tile_size+int(tile_size//2))
                    gernade_group.add(gernade)
                if tile==8:
                    exit=Exit(col_count*tile_size,row_count*tile_size)
                    exit_group.add(exit)

                col_count+=1
            row_count+=1


    def draw(self):
        for tile in self.tile_list:
            win.blit(tile[0],tile[1])
            


class Enemy(pygame.sprite.Sprite):
    def __init__(self,x,y):
        pygame.sprite.Sprite.__init__(self)
        self.image=pygame.image.load('slimeWalk1.png')
        self.rect=self.image.get_rect()
        self.rect.x=x
        self.rect.y=y
        self.move_direction=1
        self.move_counter=0
    def update(self):
        #movements for enemy copy this for creating ally
        self.rect.x += self.move_direction
        self.move_counter+=1
        if abs(self.move_counter)>50:
            self.move_direction*=-1
            self.move_counter*=-1

class Platform(pygame.sprite.Sprite):
    def __init__(self,x,y,move_x,move_y):
        pygame.sprite.Sprite.__init__(self)
        img=pygame.image.load('grass.png')
        self.image=pygame.transform.scale(img,(tile_size,tile_size//2))
        self.rect=self.image.get_rect()
        self.rect.x=x
        self.rect.y=y
        self.move_counter=0
        self.move_direction=1
        self.move_x=move_x
        self.move_y=move_y

    def update(self):
        #movements for enemy copy this for creating ally
        self.rect.x += self.move_direction*self.move_x
        self.rect.y += self.move_direction*self.move_y
        self.move_counter+=1
        if abs(self.move_counter)>50:
            self.move_direction*=-1
            self.move_counter*=-1



class Spike(pygame.sprite.Sprite):
    def __init__(self,x,y):
        pygame.sprite.Sprite.__init__(self)
        img=pygame.image.load('spikes.png')
        self.image=pygame.transform.scale(img,(tile_size,tile_size//2))
        self.rect=self.image.get_rect()
        self.rect.x=x
        self.rect.y=y


class Gernade(pygame.sprite.Sprite):
    def __init__(self,x,y):
        pygame.sprite.Sprite.__init__(self)
        img=pygame.image.load('gernade.png')
        self.image=pygame.transform.scale(img,(tile_size//1.2,tile_size//1.2))
        self.rect=self.image.get_rect()
        self.rect.center=(x,y)
        




class Exit(pygame.sprite.Sprite):
    def __init__(self,x,y):
        pygame.sprite.Sprite.__init__(self)
        img=pygame.image.load('gate.png')
        self.image=pygame.transform.scale(img,(tile_size,int(tile_size*1.5)))
        self.rect=self.image.get_rect()
        self.rect.x=x
        self.rect.y=y
#ally 

class Ally(pygame.sprite.Sprite):
    def __init__(self,x,y):
        pygame.sprite.Sprite.__init__(self)
        img=pygame.image.load('ally.png')
        self.image=pygame.transform.scale(img,(60,80))
        self.rect=self.image.get_rect()
        self.rect.x=x
        self.rect.y=y
        self.move_direction=1
        self.move_counter=0
    def update(self):
         #movements for enemy copy this for creating ally
        self.rect.x += self.move_direction
        self.move_counter+=1
        if self.move_counter>50:
           self.move_direction*=-1
           self.move_counter*=-1
          





player=Player(100,win_max_y-130)

blob_group=pygame.sprite.Group()
platform_group=pygame.sprite.Group()
spike_group=pygame.sprite.Group()
gernade_group=pygame.sprite.Group()
exit_group=pygame.sprite.Group()


#load in level data and create world
if path.exists(f"level{level}_data"):
    pickle_in=open(f'level{level}_data',"rb")
    world_data=pickle.load(pickle_in)
    spike_group=pygame.sprite.Group()
world=World(world_data)


#create buttons
restart_button=Button(win_max_x//2-50,win_max_y//2+100,restart_img)
start_button=Button(win_max_x//2-350,win_max_y//2,start_img)
exit_button=Button(win_max_x//2+150,win_max_y//2,exit_img)

run=True
while run:
    clock.tick(fps)
    win.blit(bg,(0,0))
    if main_menu==True:
        if exit_button.draw():
            run=False
        if start_button.draw():
            main_menu=False

    else:
        world.draw()
        if game_over==0:
            blob_group.update()
            platform_group.update()
            #update score 
            #check if coin collected
            if pygame.sprite.spritecollide(player,gernade_group,True):
                score+=1
                gernade_fx.play()
            draw_text('X '+str(score),font_score,white,tile_size-10,10)
        blob_group.draw(win)
        platform_group.draw(win)
        spike_group.draw(win)
        gernade_group.draw(win)
        exit_group.draw(win)
        
        game_over=player.update(game_over)
    #player dead
        if game_over==-1:
            if restart_button.draw():
                world_data=[]
                world=reset_level(level)
                game_over=0
                score=0
        if game_over==1:
            #rest game and go to next level
            level+=1
            if level<=max_level:
                #reset level
                world_data=[]
                world=reset_level(level)
                game_over=0
            else:
                draw_text('YOU WINNNN!!!!!',font,blue,(win_max_x//2)-140,win_max_y//2)
                if restart_button.draw():
                    level=1
                    world_data=[]
                    world=reset_level(level)
                    game_over=0
                    score=0

 


    for event in pygame.event.get():
        if event.type==pygame.QUIT:
            run=False

    pygame.display.update()

pygame.quit()
war_flashback.py
Displaying war_flashback.py.
Final Pygame Project
Adam Abrego Iii
•
Jun 12, 2024 (Edited Jun 13, 2024)
•
MP3
Exams, Quizzes and Projects
•
50
/50
50 points out of possible 50
Due Jun 13, 2024, 11:59 PM
This is it!  You made it!  Congratulations!!!

Please submit your final PY file here.

Include any sound or graphics files also.

REMEMBER:  THE CUTOFF IS MIDNIGHT JUNE 13 - NO LATE SUBMISSIONS!

IF YOU HAVE TOO MANY FILES TO UPLOAD, CREATE A FOLDER ON YOUR GOOGLE DRIVE WITH ALL THE FILES AND SUBMIT A SHARE LINK TO FOLDER.
Your work
Graded
Graphics
Google Drive Folder

war_flashback.py
Text

Private comments
