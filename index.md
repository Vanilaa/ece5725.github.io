# Dashboard Decor

This project is managed by Yaqi (yg298) and Alina (lw584) during Fall 2023 and demonstrate on 12/7/2023.
<center><img src="/img/grouppic.jpg" width="300" height="400"></center>

## Objective
In this project, we want to make our home or study area more comfortable, creating this safe space where we would love to study or live in, therefore we came up with the idea of creating a desktop gadget. We envision this to be a versatile, fun, and aesthetic decor for someone who is into Pixel art. This dashboard will combine different functionalities like controlling Spotify song playback, playing drawing and guessing game, entering Zen mode with weather forecast, and displaying cute animations
<center><img src="/img/welcome.png" width="400" height="320"></center>

## Introduction
We imagine this dashboard to be like a small widget where the user will be able to check for the current date, time, a short greeting based on the time of the day, current weather conditions, temperature/humidity, and sunrise/sunset time. For other interactive functions, we want it to be able to connect to the user’s Spotify account so it can control the playlist, skip back and forth in the playlist, and display the album cover. We also want to add a game to the dashboard, at first we wanted to implement Dianasour’s Jumping game in Google, but the LED panel’s pitch is too large and the animation on it doesn’t look very good, so we pivoted to a drawing game in which the player can either play with a friend, or it can be single player and just let the LED display the artwork user draws.

In terms of the technical components involved in implementation, we want to make use of the PyGame library and TFT screen as we have learned during this semester’s lab sections. Those hardware and software can be customized in so many different ways, giving us many options of how to use them. We also wanted to explore aspects of human-centered deisgn, since this is a highly interactive project. In addition, we want to try out some additional hardware like LED Panel and learn to access external APIs for real-time data and HTTP requests.


## Design and Testing

We followed modular design principles throughout the project. 
<center><img src="/img/page.jpg" width="850" height="340"></center>

### LED Panel

#### Wiring

To wire up the LED panel to Raspberry Pi, we referenced the pinouts described in Adafruit's [RGB matrix bonnet tutorial](https://learn.adafruit.com/adafruit-rgb-matrix-bonnet-for-raspberry-pi/pinouts). 

| left               | right              |
| ------------------ | ------------------ |
| LED# - Func - GPIO | LED# - Func - GPIO |
| 1-R1-5             | 9-G1-13            |
| 2-B1-6             | 10-GND-GND         |
| 3-R2-12            | 11-G2-16           |
| 4-B2-23            | 12-E-24            |
| 5-A-22             | 13-B-26            |
| 6-C-27             | 14-D-20            |
| 7-CLK-17           | 15-LAT-21          |
| 8-OE-4             | 16-GND-GND         |

We also check to make sure that the GPIO pins used for the LED panel does not overlap pins used by TFT. It does turnout that the GPIO pins connected to the buttons on the TFT screen is used by the LED panel. Therefore we incorporated an external button to function as the quit button. 

#### Installing library and running provided examples

We took advantage of hzeller's [Raspberry Pi RGB matrix library](https://github.com/hzeller/rpi-rgb-led-matrix) to control the LED panel with Raspberry Pi. After installing the library, we tried running a few example, with flags `--led-cols=64 --led-rows=32`We observed some issues, where the bottom half of the panel does not light up at all, and we couldn't really tell what is on the top half, which looks like it is glitching. So we went back to double check our wiring and configuration. We found a misplaced wire for the matrix E pin. Plugging it in the right place gives us pixels lighting up on the bottom half. We then found suggestions in the tutorial that with Pi 4, the matrix control speed needs to be dialed back slightly. We changed the `--led-slowdown-gpio` setting to 4, which fixed the glitches. 

#### Creating views individually

We study the examples that came with the library to understand the flow of projecting an image to the LED panel. There are three steps in preparing an image: importing, resizing, and converting format to RGB. We also explored playing animations on the panel by importing gif, process each frame, adding all the frames to a list, loop over the frames and display them on the panel one after the other. We also discovered that the library came with an abundant set of fonts available for use, which allow us to display text on the LED panel.

To allow the user to create their own art piece with this panel, we mapped each pixel drawn on the piTFT screens to a position on the LED screen, store the RGB values in a 2D matrix, loop through the matrix and set each Pixel individually. 

##### Spotify
Upon entering music player mode, the album cover of the current track is downloaded and stored in `album_cover.jpg`. We were careful to make sure that we do not always have to download the cover at each update of the frame, since it takes some time for download to complete. The cover is then resized and displayed on the LED panel, along with name of the track and the artist. 
<center><img src="/img/spotify.png" width="400" height="320"></center>

##### Drawing
To allow the user to create their own art piece with this panel, we mapped each pixel drawn on the piTFT screens to a position on the LED screen, store the RGB values in a 2D matrix, loop through the matrix and set each Pixel individually.
<center><img src="/img/draw.JPG" width="300" height="420"></center>

##### Weather
The weather screen displays a weather animation which is done by isolating each frame of a weather gif and displaying them in a loop to create the animated effect. On top of the weather animation, we will display the weather information obtained by requesting from an Open API, described below.

#### Accessing real-time weather information with API

For real-time weather info, we decided to use Open Weather API which allows us to make HTTP requests hourly to get accurate weather information.

We store the weather info as a JSON file since it's long and detailed. We only needed a few information like weather condition, temperature, and humidity. Therefore, we can index into the JSON, pull the information out from the JSON file, and display it on top of the animation frames.
<center><img src="/img/weather.png" width="400" height="320"></center>

### TFT Screen

#### Installing and testing Spotipy library
The [Spotipy](https://spotipy.readthedocs.io/en/2.22.1/) library is a lightweight python library for the [Spotify Web API](https://developer.spotify.com/documentation/web-api). It has the capability of accessing authorized user’s playlist, devices and controlling playback on the devices by sending http requests and receiving http responses. 
We followed the example on Spotipy documentation and first created and testsed a simple python script to control playback on our personal device through command line. We then organized the code into functions and created our own `my_spotify` library. The library has four functions: initialize, download album cover, add playlist to queue, and fetch current playback’s track title and artist. 


#### Creating user interfaces with Pygame

##### Home Screen

Three icons are centered on the TFT home page: spotify, draw, and weather. Each icon will direct the user to the corresponding page. For implementation, we created these Pygame rects and detected mouse collision with the rects. After switching to the other screen, we will use the pygame’s clear function to clean it.

##### Drawing

For the drawing screen, we want to have a color palette screen where the user can change the color of the paintbrush and a drawing area. Therefore we have 2 main Pygame screens, the color palette consists of the color circles which detects mouse collision to determine which color the user is trying to change to. The drawing screen will blit all the pixel user drawn on it and append the pixel’s coordinate to an RGB matrix which will be mapped to the LED screen at the same time. Therefore, both TFT and LED screens will be able to see the drawing live-time. 

### Other

#### Spotifyd

We also followed a Youtube tutorial on [how to turn Raspberry Pi device into a device on Spotify account](https://www.youtube.com/watch?v=GGXJuzSise4)

Mainly, we added a config file in which we specify the Spotify account logins and added the device we wanted to use to play music which is RPi. 



## Result
<iframe width="500" height="295" src="https://www.youtube.com/embed/et91Gea6CPk?rel=0" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

## Conclusion

We successfully achieved 90% of the functionality we proposed in our proposal and substituted the 10% with functionalities we thought would be more fun to implement as we worked on it. We were able to solve most of the bugs and we pushed through the project even when we lost all of our files when crontab messed up. 
We found out that due to Spotify’s security policy, we won’t be able to use Crontab, since it requires Spotify authorization after each reboot or shutdown. We also learned more about how to restore files if Crontab didn’t function properly, and always backup or at least download our files before reboot. 


## Future Work
There are still a lot to do if we want to improve this project.
1. We can clean up the wires and build a display case for it
2. Add the functionality to save user's artwork
3. Scrolling song title and artist name when they are too long. 

## Budget

| Part         | Quantity          | Our Cost  | Actual Cost|
|:-------------|:------------------|:----------|:-----------|
| RPi          | 1                 | 0         | 45        |
| TFT screen   | 1                 | 0         | 45        |
| Button       | 1                 | 0         | 0.5        |
| LED Panel    | 2                 | 0         | 45        |
| 5V 4A adapter| 1                 | 0         | 0        |
| Jump wires   | a handful         | 0         | 0        |
Total: $135.5


## Reference

### Libraries

1. pygame
2. spotipy
3. spotifyd
4. rpi-rgb-led-marix

### Tutorials/Guides
These are linked though out the webpage for seamless reading.


### Code Appendix
Below is our master code where we integrated everything.
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
drawing = True
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

home_icon = pygame.image.load("img/home_icon.png")
home_icon = pygame.transform.scale(home_icon, (50,50))
home_rect = home_icon.get_rect().move(40,0)

clear_icon = pygame.image.load("img/clear_icon.png")
clear_icon = pygame.transform.scale(clear_icon, (50,50))
clear_rect = clear_icon.get_rect().move(40,80)

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
scribble_surface.blit(clear_icon, (40,80))
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
    # canvas.Clear()
    for i in range(64):
        for j in range(32):
            r,g,b = led_matrix[i][j]
            canvas.SetPixel(i,j,r,g,b)

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
        

        
        start_time = time.time()

        global is_home
        global is_music_player
        global is_scribble
        global is_weather
        global drawing
        global sp
        global code_run
        global playing
        global brush_color
        
        album_cover = update_album_cover(sp)
        track_name, artist = my_spotify.get_curr_track_info(sp)
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
                    
                    if event.type == pygame.MOUSEBUTTONUP:
                        mouse_pos = pygame.mouse.get_pos()

                        if is_home:
                            if spotify_rect.collidepoint(mouse_pos):
                                is_music_player = True
                                is_home = False
                                is_scribble = False
                                
                                album_cover = update_album_cover(sp)
                                track_name, artist = my_spotify.get_curr_track_info(sp)

                            elif scribble_rect.collidepoint(mouse_pos):
                                is_music_player = False
                                is_home = False
                                is_scribble = True
                            
                            elif weather_rect.collidepoint(mouse_pos):
                                is_home = False
                                is_weather = True
                                print("detected is weather")

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
                                track_name, artist = my_spotify.get_curr_track_info(sp)


                            elif (prev_track_rect_1.collidepoint(mouse_pos) or prev_track_rect_2.collidepoint(mouse_pos)):
                                sp.next_track()
                                playing = True
                                # my_spotify.download_curr_album_cover(sp)
                                album_cover = update_album_cover(sp)
                                track_name, artist = my_spotify.get_curr_track_info(sp)

                        elif is_scribble:
                            if home_rect.collidepoint(mouse_pos):
                                is_home = True
                                is_scribble = False
                            elif clear_rect.collidepoint(mouse_pos):
                                canvas.Clear()
                                for i in range(64):
                                    for j in range(32):
                                        save_scribble[i][j] = BLACK
                                draw_surface.fill(BLACK)
                                screen.blit(draw_surface, (0,0))
                                screen.blit(pallete_surface,(0,0))
                                screen.blit(home_icon, (40,0))
                                screen.blit(clear_icon, (40,80))
                                pygame.display.flip()
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
                        
                        elif is_weather:
                            if home_rect.collidepoint(mouse_pos):
                                is_home = True
                                is_weather = False

                    elif event.type == pygame.MOUSEBUTTONDOWN and is_scribble:
                        drawing = True

################################ TFT Screen ########################################
                if is_music_player:
                    draw_music_player()
                elif is_home:
                    draw_home()
                elif is_scribble:
                    screen.fill(BLACK)
                    if drawing:
                        mouse_pos = pygame.mouse.get_pos()
                        save_scribble[int(mouse_pos[0]/5)][int(mouse_pos[1]/7.5)] = brush_color
                        draw_surface.set_at(mouse_pos, brush_color)
                    screen.blit(draw_surface, (0, 0))
                    screen.blit(pallete_surface, (0, 0))
                    screen.blit(home_icon, (40,0))
                    screen.blit(clear_icon, (40,80))
                    pygame.display.flip()

                elif is_weather:
                    screen.fill(BLACK)
                    screen.blit(home_icon, (40,0))
                    pygame.display.flip()

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

                    bear = Image.open("img/bear.png")
                    bear = bear.resize((32,32),Image.ANTIALIAS).convert("RGB")
                    
                    canvas.Clear()
                    canvas.SetImage(bear, 32, 0)
                    graphics.DrawText(canvas, font, 2, 10, led_white, curr_date)
                    graphics.DrawText(canvas, font, 2, 20, led_white, curr_time)
                    graphics.DrawText(canvas, font, 2, 30, led_white, greeting)

                elif is_music_player:
                    canvas.Clear()
                    canvas.SetImage(album_cover, 3,3)
                    graphics.DrawText(canvas, font, 30, 10, led_white, track_name)    
                    graphics.DrawText(canvas, font, 30, 20, led_white, artist)     

                elif is_scribble:
                    canvas.Clear()
                    map_led_matrix(canvas, save_scribble)

                elif is_weather:
                    canvas.Clear()
                                    
                    for frame in gif[0:10]:
                        if not is_weather:
                            break
                        # canvas.SetImage(image, 0,15)
                        canvas.SetImage(frame, 0,0)
                        graphics.DrawText(canvas, font, 2, 18, led_white, current_weather)
                        graphics.DrawText(canvas, font, 2, 25, led_white, hum)
                        graphics.DrawText(canvas, font, 2, 31, led_white, sun_condition)
            
                        time.sleep(0.1)
                                    


                time.sleep(0.05)
                
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


Below is the Spotify API code where we defined our own functions to help us download album cover, switch between songs, and pulling song infos.

``` python
import spotipy
from spotipy.oauth2 import SpotifyOAuth
import json
import requests
import os

#set environment variables
os.system("export CLIENT_ID=e522f9c5a6c0487c9e17600b3dc47473")
os.system("export CLIENT_SECRET=cdee41afeb544e5688eaceaf45ad7781")
os.system("export REDIRECT_URI=http://localhost:8080")

scope = "streaming,user-read-playback-state,user-library-read,user-modify-playback-state,app-remote-control"

client_id = "e522f9c5a6c0487c9e17600b3dc47473"
client_secret = "cdee41afeb544e5688eaceaf45ad7781"
redirect_url = "http://localhost:8080"

def spotify_init():
    scope = "streaming,user-read-playback-state,user-library-read,user-modify-playback-state,app-remote-control"
    sp = spotipy.Spotify(auth_manager=SpotifyOAuth(scope=scope, client_id = client_id, client_secret = client_secret, redirect_uri = redirect_url, open_browser=False))
    device_id = sp.devices()["devices"][0]["id"]
    sp.transfer_playback(device_id)
    sp.pause_playback()
    return sp


def add_playlist_to_queue(sp):
    playlist = sp.current_user_playlists(limit = 50, offset = 0)["items"][0]
    play_list_id = playlist["id"]
    items = sp.playlist_items(play_list_id)
    tracks = items["items"]
    num_songs = len(tracks)
    first_id = tracks[0]["track"]["id"]
    for track in tracks:
        track_id = track["track"]["id"]
        sp.add_to_queue(track_id)
    # while(sp.queue()["currently_playing"]["id"] != first_id):
    #     sp.next_track()
    
    try:
        sp.start_playback()
    except:
        pass
    download_curr_album_cover(sp)
    # sp.seek_track(0)


def download_curr_album_cover(sp):
    album_cover_url = sp.current_playback()["item"]["album"]["images"][-1]["url"]
    response = requests.get(album_cover_url)
    with open("album_cover.jpg", 'wb') as f:
        f.write(response.content)



def get_curr_track_info(sp):
    curr_playback = sp.current_playback()
    artist = curr_playback["item"]["artists"][0]["name"]
    track_name = curr_playback["item"]["name"]
    return track_name, artist

# print(json.dumps(sp.current_playback(), indent = 4))
sp = spotify_init()
add_playlist_to_queue(sp)
print(get_curr_track_info(sp))
```
