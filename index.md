# Dashboard Decor

This project is managed by Yaqi (yg298) and Alina (lw584) during Fall 2023 and demonstrate on 12/7/2023.
<center><img src="/img/grouppic.jpg" width="300" height="400"></center>

## Objective
In this project, we want to make our home or study area more comfortable, creating this safe space where we would love to study or live in, therefore we came up with the idea of creating a desktop gadget. We envision this to be a versatile, fun, and aesthetic decor for someone who is into Pixel art. This dashboard will combine different functionalities like controlling Spotify song playback, playing drawing and guessing game, entering Zen mode with weather forecast, and displaying cute animations

## Introduction
We imagine this dashboard to be like a small widget where the user will be able to check for the current date, time, a short greeting based on the time of the day, current weather conditions, temperature/humidity, and sunrise/sunset time. For other interactive functions, we want it to be able to connect to the user’s Spotify account so it can control the playlist, skip back and forth in the playlist, and display the album cover. We also want to add a game to the dashboard, at first we wanted to implement Dianasour’s Jumping game in Google, but the LED panel’s pitch is too large and the animation on it doesn’t look very good, so we pivoted to a drawing game in which the player can either play with a friend, or it can be single player and just let the LED display the artwork user draws.

In terms of the technical components involved in implementation, we want to make use of the PyGame library and TFT screen as we have learned during this semester’s lab sections. Those hardware and software can be customized in so many different ways, giving us many options of how to use them. We also wanted to explore aspects of human-centered deisgn, since this is a highly interactive project. In addition, we want to try out some additional hardware like LED Panel and learn to access external APIs for real-time data and HTTP requests.


## Design and Testing

We followed modular design principles throughout the project. 

### LED Panel

#### Wiring

To wire up the LED panel to Raspberry Pi, we referenced the pinouts decribed in Adafruit's [RGB matrix bonnet tutorial](https://learn.adafruit.com/adafruit-rgb-matrix-bonnet-for-raspberry-pi/pinouts). 



We also check to make sure that the GPIO pins used for the LED panel does not overlap pins used by TFT. It does turnout that the GPIO pins connected to the buttons on the TFT screen is used by the LED panel. Therefore we incorporated an external button to function as the quit button. 

#### Installing library and running provided examples

We took advantage of hzeller's [Raspberry Pi RGB matrix library](https://github.com/hzeller/rpi-rgb-led-matrix) to control the LED panel with Raspberry Pi. After installing the library, we tried running a few example, with flags `--led-cols=64 --led-rows=32`We observed some issues, where the bottom half of the panel does not light up at all, and we couldn't really tell what is on the top half, which looks like it is glitching. So we went back to double check our wiring and configuration. We found a misplaced wire for the matrix E pin. Plugging it in the right place gives us pixels lighting up on the bottom half. We then found suggestions in the tutorial that with Pi 4, the matrix control speed needs to be dialed back slightly. We changed the `--led-slowdown-gpio` setting to 4, which fixed the glitches. 

#### Creating views individually

We then study the examples that came with the library to understand the flow of projecting an image to the LED panel. 

#### Accessing real-time weather information with API



### TFT Screen

#### Installing and testing Spotipy library

#### Creating user interfaces with Pygame

##### Home Screen

##### Drawing

##### Spotify



### Other

#### Spotifyd

turn Raspberry Pi device into a device on your Spotify account



## Result
<iframe width="500" height="295" src="https://www.youtube.com/embed/SPaAW63JxDM?rel=0" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

## Conclusion


## Future Work

functionality to save user's artwork

## Budget

| Part         | Quantity          | Cost  |
|:-------------|:------------------|:------|
| RPi          | 1                 | 0     |
| TFT screen   | 1                 | 0     |
| Button       | 1                 | 0     |
| LED Panel    | 2                 | 0     |
| 5V 4A adapter| 1                 | 0     |
| Jump wires   | a handful         | 0     |


## Reference

### Libraries

pygame

spotipy

spotifyd

rpi-rgb-led-marix

### Tutorials/Guides



### Code Appendix

```python
################### Libraries ###########################
from samplebase import SampleBase
from rgbmatrix import graphics
import time
from datetime import datetime
from PIL import Image
from PIL import ImageSequence
import pygame
import my_spotify
import os
import RPi.GPIO as GPIO
import sys
import my_spotify
import json

#################### Setup Environment ##################
os.putenv('SDL_VIDEODRIVER', 'fbcon')
os.putenv('SDL_FBDEV', '/dev/fb0') 
os.putenv('SDL_MOUSEDRV', 'TSLIB') # Track mouse clicks on piTFT
os.putenv('SDL_MOUSEDEV', '/dev/input/touchscreen')

print("start")

print("init")
pygame.init()
TFT_SIZE = width, height = 320, 240
print("setmode")
screen = pygame.display.set_mode(TFT_SIZE)

################## Constants ############################



print("initializing spotify")
# initialize spotify
sp = my_spotify.spotify_init()
my_spotify.add_playlist_to_queue(sp)
print("finished initializing spotify")

# pygame colors
BLACK = 0, 0, 0
WHITE = 255, 255, 255
RED = 252, 65, 3
ORANGE = 252, 154, 48
GREEN = 111, 224, 101
BLUE = 3, 148, 252
PURPLE = 209, 109, 227
GREY = 128, 128, 128

led_white = graphics.Color(255,255,255)



############## GLOBAL VARIABLES ###################
code_run = True
is_home = True
is_music_player = False
is_scribble = False
is_weather = False
playing = False
brush_color = WHITE

################### TFT ###########################

print("setup pygame")


# setup pygame on tft
screen.fill(BLACK)

# surfaces and icons
spotify_surface = pygame.Surface(TFT_SIZE)
home_surface = pygame.Surface(TFT_SIZE)


spotify_icon = pygame.image.load("img/spotify_icon.png")
spotify_icon = pygame.transform.scale(spotify_icon, (50,50))
spotify_rect = spotify_icon.get_rect().move((40,120))

scribble_icon = pygame.image.load("img/scribble.png")
scribble_icon = pygame.transform.scale(scribble_icon, (50,50))
scribble_rect = scribble_icon.get_rect().move((130,120))

weather_icon = pygame.image.load("img/weather_icon.png")
weather_icon = pygame.transform.scale(weather_icon, (50,50))
weather_rect = weather_icon.get_rect().move(220,120)

home_icon = pygame.image.load("img/home-icon.png")
home_icon = pygame.transform.scale(home_icon, (50,50))
home_rect = home_icon.get_rect().move(40,0)

pause_rect_1 = pygame.Rect(148,108,10,30)
pause_rect_2 = pygame.Rect(163,108,10,30)

spotify_surface.blit(home_icon, (40,0))
play_button = pygame.draw.circle(spotify_surface, WHITE, (width//2, height//2), 30)

offset = 80
next_track_rect_1 = pygame.draw.polygon(spotify_surface, WHITE, ((147+offset,135),(147+offset,105),(177+offset,120)))
offset = 105
next_track_rect_2 = pygame.draw.polygon(spotify_surface, WHITE, ((147+offset,135),(147+offset,105),(177+offset,120)))

offset = 80
prev_track_rect_1 = pygame.draw.polygon(spotify_surface, WHITE, ((147-offset,135),(147-offset,105),(177-offset-60,120)))
offset = 55
prev_track_rect_2 = pygame.draw.polygon(spotify_surface, WHITE, ((147-offset,135),(147-offset,105),(177-offset-60,120)))

def draw_music_player():
    screen.fill(BLACK)
    if not playing:
        pygame.draw.circle(spotify_surface, WHITE, (width//2, height//2), 30)
        pygame.draw.polygon(spotify_surface, BLACK, ((147,135),(147,105),(177,120)))
    elif playing:
        pygame.draw.circle(spotify_surface, WHITE, (width//2, height//2), 30)
        pygame.draw.rect(spotify_surface, BLACK, pause_rect_1)
        pygame.draw.rect(spotify_surface, BLACK, pause_rect_2)

    screen.blit(spotify_surface, (0,0))
    pygame.display.flip()

def draw_home():
    global spotify_rect
    screen.fill(BLACK)
    home_surface.blit(spotify_icon, spotify_rect)
    home_surface.blit(scribble_icon, scribble_rect)
    home_surface.blit(weather_icon, weather_rect)
    screen.blit(home_surface,(0,0))
    pygame.display.flip()




# scribble helper 
def color_pallete():
    # init pallette surface, fill it and draw circles on it, return the surface and the circle rects
    pallete_surface = pygame.Surface((40, 240))
    pallete_surface.fill(GREY)
    red_circle = pygame.draw.circle(pallete_surface, RED, (20, 20), 10)
    orange_circle = pygame.draw.circle(pallete_surface, ORANGE, (20, 60), 10)
    green_circle = pygame.draw.circle(pallete_surface, GREEN, (20, 100), 10)
    blue_circle = pygame.draw.circle(pallete_surface, BLUE, (20, 140), 10)
    purple_circle = pygame.draw.circle(pallete_surface, PURPLE, (20, 180), 10)
    white_circle = pygame.draw.circle(pallete_surface, WHITE, (20, 220), 10)
    return pallete_surface, red_circle, orange_circle, green_circle, blue_circle, purple_circle, white_circle

print("setup pallete surface")
#--------- Setting up the surface and prepare to enter drawing mode -----------------  
pallete_surface, red_circle, orange_circle, green_circle, blue_circle, purple_circle, white_circle = color_pallete()
print("finished setup pallete surface")

# init the draw surface
draw_surface = pygame.Surface((280, 240))
draw_surface.fill(BLACK)

scribble_surface = pygame.Surface((320,240))

# blit and flip both
scribble_surface.blit(pallete_surface, (0, 0))
scribble_surface.blit(draw_surface, (40, 0))
scribble_surface.blit(home_icon, (0,0))

print("finished setup pygame")

save_scribble = []
for i in range(64):
    save_scribble.append([])
    for j in range(32):
        save_scribble[i].append(BLACK)

def GPIO19_callback(channel):
    print("quit")
    global code_run
    code_run = False

GPIO.setmode(GPIO.BCM)
GPIO.setup(19, GPIO.IN, GPIO.PUD_DOWN)
GPIO.add_event_detect(19, GPIO.FALLING, callback = GPIO19_callback, bouncetime=300)

        
def map_led_matrix(canvas, led_matrix):
    canvas.Clear()
    for i in range(64):
        for j in range(32):
            r,g,b = led_matrix[i][j]
            canvas.SetPixel(64,32,r,g,b)

################# weather page #####################
f = open('weather.json')
weather_dict = json.load(f)
weather_dict_all = weather_dict
weather_dict = weather_dict["current"]
temp = weather_dict["temp"]
hum = 'Hum:' + str(weather_dict["humidity"]) + '%'
current_weather =  weather_dict["weather"][0]["main"]
sunrise_utc = int(weather_dict["sunrise"]) + int(weather_dict_all["timezone_offset"])
sunset_utc = int(weather_dict["sunset"]) + int(weather_dict_all["timezone_offset"])
sun_condition = ''
print(sunrise_utc)
f.close()

time_now = datetime.now()
hour = int(time_now.strftime("%H"))
if hour < 12:
    sun_condition = datetime.utcfromtimestamp(sunrise_utc).strftime('%H:%M')
    sun_condition = 'Sunrise:' + sun_condition
    weather_gif = ["sun.gif", "snow.gif", "rain.gif", "cloudy_day.gif"]
else:
    sun_condition = datetime.utcfromtimestamp(sunset_utc).strftime('%H:%M')
    sun_condition = 'Sunset:' + sun_condition
    weather_gif = ["moon.gif", "snow.gif", "rain.gif", "cloudy_night.gif"]

weathers = [["Clear"],["Snow"], ["Rain", "Thunderstorm", "Drizzle"],["Clouds"]]

gif = []
category_count = 0
found = False
while not found:
    for category in weathers:
        for weather in category:
            if current_weather == weather:
                img = weather_gif[category_count]
                found = True
                break
        category_count += 1
        
with Image.open("img/" + img) as im:
    for frame in ImageSequence.Iterator(im):
        frame_processed = frame.resize((64,32),Image.ANTIALIAS).convert("RGB")
        gif.append(frame_processed)           

def update_album_cover(sp):
    my_spotify.download_curr_album_cover(sp)
    im = Image.open("album_cover.jpg")
    album_cover = im.resize((26,26),Image.ANTIALIAS).convert("RGB")
    return album_cover
    
################### LED ###########################
class Dashboard(SampleBase):
    def __init__(self, *args, **kwargs):
        super(Dashboard, self).__init__(*args, **kwargs)

    def run(self):
        print("running dashboard")
        canvas = self.matrix

        font = graphics.Font()
        
        time_now = datetime.now()
        curr_date = time_now.strftime("%m/%d/%Y")
        curr_time = time_now.strftime("%H:%M")
        
        font.LoadFont("../rpi-rgb-led-matrix/fonts/4x6.bdf")

        # graphics.DrawText(canvas, font, 22, 10, led_white, curr_date)
        # graphics.DrawText(canvas, font, 30, 20, led_white, curr_time)
        # graphics.DrawText(canvas, font, 22, 28, led_white, greeting)
        
        # hour = int(time_now.strftime("%H"))
        # greeting = ""
        # if (hour >= 5 and hour < 12):
        #     greeting = "Good morning"
        # elif (hour >= 12 and hour < 18):
        #     greeting = "Good afternoon"
        # elif (hour >= 18 and hour < 22):
        #     greeting = "Good evening"
        # else:
        #     greeting = "Good night"
        
        # my_spotify.download_curr_album_cover(sp)
        # im = Image.open("album_cover.jpg")
        # album_cover = im.resize((26,26),Image.ANTIALIAS).convert("RGB")
        
        album_cover = update_album_cover(sp)
        track_name, artist = my_spotify.get_curr_track_info()
        
        start_time = time.time()

        global is_home
        global is_music_player
        global is_scribble
        global drawing
        global sp
        global code_run
        global playing
        
        print("running while loop")

        while(code_run):
                # print(str(code_run))
            try:
                for event in pygame.event.get():
                    if (event.type == pygame.KEYUP and event.key == pygame.K_c and event.mod & pygame.KMOD_CTRL):
                        print("quit event")
                        img = Image.new('RGB', (64,32))
                        for i in range(64):
                            for j in range(32):
                                img.putpixel((i,j),save_scribble[i][j])
                        img.save("output.png")
                        pygame.quit()
                        sys.exit()
                        quit()
                    
                    if event.type == pygame.MOUSEBUTTONUP:
                        mouse_pos = pygame.mouse.get_pos()
                        if is_home:
                            if spotify_rect.collidepoint(mouse_pos):
                                is_music_player = True
                                is_home = False
                                is_scribble = False
                                
                                album_cover = update_album_cover(sp)
                                track_name, artist = my_spotify.get_curr_track_info()

                                # print("music")

                            elif scribble_rect.collidepoint(mouse_pos):
                                is_music_player = False
                                is_home = False
                                is_scribble = True
                                # print("scribble")

                        elif is_music_player:
                            if (play_button.collidepoint(mouse_pos)):
                                if not playing:
                                    sp.start_playback()
                                    playing = True
                                else:
                                    sp.pause_playback()
                                    playing = False

                            elif(home_rect.collidepoint(mouse_pos)):
                                is_music_player = False
                                is_home = True

                            elif (next_track_rect_1.collidepoint(mouse_pos) or next_track_rect_2.collidepoint(mouse_pos)):
                                sp.next_track()
                                playing = True
                                # my_spotify.download_curr_album_cover(sp)
                                album_cover = update_album_cover(sp)
                                track_name, artist = my_spotify.get_curr_track_info()


                            elif (prev_track_rect_1.collidepoint(mouse_pos) or prev_track_rect_2.collidepoint(mouse_pos)):
                                sp.next_track()
                                playing = True
                                my_spotify.download_curr_album_cover(sp)
                                album_cover = update_album_cover(sp)
                                track_name, artist = my_spotify.get_curr_track_info()

                        elif is_scribble:
                            if home_rect.collidepoint(mouse_pos):
                                is_home = True
                                is_scribble = False
                            if (red_circle.collidepoint(mouse_pos)):
                                brush_color = RED
                            elif (green_circle.collidepoint(mouse_pos)):
                                brush_color = GREEN
                            elif (blue_circle.collidepoint(mouse_pos)):
                                brush_color = BLUE
                            elif (white_circle.collidepoint(mouse_pos)):
                                brush_color = WHITE
                            elif(orange_circle.collidepoint(mouse_pos)):
                                brush_color = ORANGE
                            elif(purple_circle.collidepoint(mouse_pos)):
                                brush_color = PURPLE
                            else: 
                                drawing = False
                    elif event.type == pygame.MOUSEBUTTONDOWN and is_scribble:
                        drawing = True
##################################################################################
                if(is_home):
                    time_now = datetime.now()
                    curr_date = time_now.strftime("%m/%d/%Y")
                    curr_time = time_now.strftime("%H:%M")

                    hour = int(time_now.strftime("%H"))
                    greeting = ""
                    if (hour >= 5 and hour < 12):
                        greeting = "Good morning"
                    elif (hour >= 12 and hour < 18):
                        greeting = "Good afternoon"
                    elif (hour >= 18 and hour < 22):
                        greeting = "Good evening"
                    else:
                        greeting = "Good night"

                    bear = Image.open("bear.png")
                    bear = bear.resize((32,32),Image.ANTIALIAS).convert("RGB")
                    
                    canvas.Clear()
                    canvas.setImage(bear, 32, 0)
                    graphics.DrawText(canvas, font, 0, 10, led_white, curr_date)
                    graphics.DrawText(canvas, font, 0, 20, led_white, curr_time)
                    graphics.DrawText(canvas, font, 0, 20, led_white, greeting)

                elif is_music_player:
                    #place holders
                    canvas.SetImage(album_cover, 3,3)
                    graphics.DrawText(canvas, font, 28, 10, led_white, track_name)    
                    graphics.DrawText(canvas, font, 28, 20, led_white, artist)     

                elif is_scribble:
                    map_led_matrix(canvas, save_scribble)

                elif is_weather:
                    canvas.Clear()
                    while True:
                        for frame in gif:
                            if not is_weather:
                                break
                            # canvas.SetImage(image, 0,15)
                            canvas.SetImage(frame, 0,0)
                            graphics.DrawText(canvas, font, 0, 18, led_white, current_weather)
                            graphics.DrawText(canvas, font, 0, 25, led_white, hum)
                            graphics.DrawText(canvas, font, 0, 31, led_white, sun_condition)
                
                            time.sleep(0.1)
                                    
################################ TFT Screen ########################################
                if is_music_player:
                    draw_music_player()
                elif is_home:
                    draw_home()
                elif is_scribble:
                    # draw_scribble()
                    if drawing:
                        mouse_pos = pygame.mouse.get_pos()
                        save_scribble[int(mouse_pos[0]/5)][int(mouse_pos[1]/7.5)] = brush_color
                        draw_surface.set_at(mouse_pos, brush_color)
                    screen.blit(draw_surface, (0, 0))
                    screen.blit(pallete_surface, (0, 0))
                    screen.blit(home_icon, (40,0))
                    pygame.display.flip()

                time.sleep(0.05)
            # except: 
            #     pygame.quit()
            #     quit()   
                
                
            except KeyboardInterrupt:
                print("keyboard interrupt")
                code_run = False
                break

        print("exit while loop")
        canvas.Clear()
        GPIO.cleanup()
        pygame.display.quit()
        return
        # pygame.quit()
        # sys.exit()
        # quit() 

if __name__ == "__main__":
    print("enter main")
    graphics_test = Dashboard()
    if (not graphics_test.process()):
        graphics_test.print_help()
```



