First time sharing code, sorry if I do it wrong. I see a lot of people posting single YAML blocks and I can never tell how it fits in. 

I'm not sure if I am doing this optimally, as its hard to know when to use YAML vs visual aids. Anyway here's how I did it.

First generate a room in chatGPT 4o. Give it a clean photo of your room and say "Generate an isometric 3-D model of this room and keep the composition as accurate as possible"

When you get one you like, feed it back in (must re-upload), and say "Take this exact image. Change nothing, except make the lighting blue"

For the color selectors, say "Generate me an icon (photo, with color selector at the bottom. circles in a row) for a blue and purple theme"

Do this for all colors, and a lights off option if you want.

In Home Assistant, go to Helpers, then create a new one > Template > Template a sensor

In the code part in State Template, put this. It uses Euclidean distance and predefined CSS RGB codes to classify colors. 

* Rename ```tv_light_1``` to whichever light entity you want to use. Although I set my lights to different colors, for now, I only need to read 1 to know the color of the room.
* This will only break if I have a light setting with many very different colors. Right now, all of my presets are all similar (e.g blues, reds, etc). In that case, Ill likely make more images, custom labels, and figure out how to incorporate multiple lights into this function

```
{% set color = state_attr('light.tv_light_1', 'rgb_color') %}
{% if color is none %}
  off
{% elif color | length == 3 and color[0] == 0 and color[1] == 0 and color[2] == 0 %}
  off
{% elif color | length == 3 %}
  {% set r = color[0] | int %}
  {% set g = color[1] | int %}
  {% set b = color[2] | int %}
  {% set named_colors = {
    'red': [255, 0, 0],
    'green': [0, 255, 0],
    'blue': [0, 0, 255],
    'yellow': [255, 255, 0],
    'pink': [255, 105, 180],
    'white': [255, 255, 255],
    'cyan': [0, 255, 255],
    'purple': [128, 0, 128],
    'orange': [255, 165, 0],
    'black': [0, 0, 0],
    'lightblue': [173, 216, 230],
    'magenta': [255, 0, 255],
    'lime': [50, 205, 50],
    'skyblue': [135, 206, 235],
    'navy': [0, 0, 128],
    'midnightblue': [25, 25, 112],
    'darkblue': [0, 0, 139],
    'indigo': [75, 0, 130]
  } %}
  {% set ns = namespace(closest_name='unknown', closest_distance=999999) %}
  {% for name, val in named_colors.items() %}
    {% set dr = r - val[0] %}
    {% set dg = g - val[1] %}
    {% set db = b - val[2] %}
    {% set dist = (dr * dr + dg * dg + db * db) %}
    {% if dist < ns.closest_distance %}
      {% set ns.closest_distance = dist %}
      {% set ns.closest_name = name %}
    {% endif %}
  {% endfor %}
  {{ ns.closest_name }}
{% else %}
  unavailable
{% endif %}
```
In the dashboard, make a new picture elements card. Put this in the code editor
```
type: picture-elements
image: /local/blank.png
elements:
  - type: image
    entity: sensor.living_room_light_sensor
    state_image:
      "off": /local/living_room_lights_off.png
      blue: /local/living_room_blue.png
      red: /local/living_room_red.png?v=2
      green: /local/living_room_green.png
      orange: /local/living_room_orange.png
      pink: /local/living_room_pink.png
      white: /local/living_room_lights_onn.png
    style:
      left: 50%
      top: 50%
      width: 100%
```

This is mine and its super incomplete. I only have 3 color rooms generated and 3 scenes (blue, red, white). But all I need to do is generate more images, and name them correctly and they will show up.

The color selector buttons are just button cards and are not connected to this. They control the lights (triggering a script that sets light colors), and because of the light changing, the sensor is triggered and the picture listens to that.

If you want, here's the code I use for the button rows. Requires custom button card from HACS.
````type: custom:button-card
entity: light.bcl_local
entity_picture: /local/blue_lights.png
show_entity_picture: true
tap_action:
  action: call-service
  service: script.blue_purple
styles:
  card:
    - background: none
    - box-shadow: none
    - border: none
    - padding: 0
    - margin: 0
  entity_picture:
    - width: 130px
    - height: auto
    - border-radius: 16px
    - object-fit: cover
  grid:
    - grid-template-areas: "\"i\""
    - justify-items: center
