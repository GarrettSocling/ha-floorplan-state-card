# Floorplan state card for Home Assistant

![floorplan](https://cloud.githubusercontent.com/assets/2073827/25836731/868fed80-34cc-11e7-9f1a-7ec1619d2d2a.png)

## Background

In the Home Assistant [front end](https://home-assistant.io/docs/frontend/), a [state card](https://home-assistant.io/developers/frontend_add_card/) is displayed for each entity. This works great as it is, but in some cases, you may want a single state card to display multiple entities. One such example is a floorplan of your home, on which you'd like to view a range of binary sensors and switches. This project helps you achieve that, thanks to the ability of customizing state cards in Home Assistant.

With Floorplan state card for Home Assistant, you can:

- Display your floorplan image as a state card
- Include any number of binary sensors and switches on your floorplan
- Specify colors for rendering the 'on' and 'off' states
- Gradually transition from 'on' to 'off' using color gradients
- Display the last triggered binary sensor
- Display hover-over text for each binary sensor or switch


## Usage

### Create the floorplan SVG file

To get things up and running, you first need to create an SVG file of your floorplan. [Inkscape](https://inkscape.org/en/develop/about-svg/) is a free application that lets you create vector images. You can make your floorplan as simple or as detailed as you want. The only requirement is that you create a shape (i.e. `rectangle`, `path`, etc.) for each binary sensor or switch you want to display on your floorplan. Each of these shapes needs to have its `id` set to the entity name in Home Assistant.

For example, below is what the shape looks like for a Front Hallway binary sensor. The `id` of the shape is set to the entity name `binary_sensor.front_hallway`. This allows the shape to automatically get hooked up to the right entity when the state card is displayed.

```html
<path id="binary_sensor.front_hallway" d="M650 396 c0 -30 4 -34 31 -40 17 -3 107 -6 200 -6 l169 0 0 40 0 40
-200 0 -200 0 0 -34z"/>
```
Once you've created your SVG file, save it a directory within your Home Assistant installation. For example:
```
<path_to_home_assistant>/www/custom_ui/floorplan/floorplan.svg
```

### Add an entity to represent the floorplan

Since Home Assistant assumes a one-to-one mapping between entities and state cards, we need to create a 'dummy' entity to represent the floorplan. The MQTT binary sensor does the job. Add the following to your Home Assistant configuration. You can set the `state_topic` to any value. The name is the only part that matters as it will be used to add the floorplan as a state card in the next step.

```
binary_sensor:
  - platform: mqtt
    state_topic: dummy/floorplan/sensor
    name: Floorplan
```

### Add the floorplan entity to a group

To get the floorplan to display within a group, add the `binary_sensor.floorplan` to a group. In the example below, it's being added to a group called `Zones`.

```
group:
  zones:
    name: Zones
    entities:
      - binary_sensor.floorplan
```

As an optional step, you can create a 'last motion' entity to keep track of which binary sensor or switch was triggered last. To do so, add the following:

```
sensor:
  - platform: template
    sensors:
      template_last_motion:
        friendly_name: 'Last Motion'
        value_template: >
          {%- set sensors = [states.binary_sensor.theatre_room, states.binary_sensor.back_hallway, states.binary_sensor.front_hallway, states.binary_sensor.kitchen] %}
          {% for sensor in sensors %}
            {% if as_timestamp(sensor.last_changed) == as_timestamp(sensors | map(attribute='last_changed') | max) %}
              {{ sensor.name }}
            {% endif %}
          {% endfor %}
```

You can then get this 'last motion' entity to also display in the `Zones` group:

```
group:
  zones:
    name: Zones
    entities:
      - sensor.template_last_motion
      - binary_sensor.floorplan
```

Refer to the next section for information about how to style the apperance of the 'lasy motion' entity.

### Customize the floorplan state card

The final step is to get Home Assistant to display the floorplan using its own customized state card.

Copy the floorplan state card HTML file (included in this repository) to the Custom UI directory within Home Assistant:

```
<path_to_home_assistant>/www/custom_ui/state-card-floorplan.html
```

You can also copy the floorplan CSS file (included in this repository) to the location shown below. This allows you to apply any styling to the floorplan, SVG elements, etc.

```
<path_to_home_assistant>/www/custom_ui/floorplan/floorplan.css
```

The sample CSS file already contains a rule for styling the appearance of the 'last motion' entity (i.e. adds a grey border).

Then add the following customization:

```
homeassistant:
  customize: 
    binary_sensor.floorplan:
      custom_ui_state_card: floorplan
      floorplan_image: /local/custom_ui/floorplan/floorplan.svg
      entities:
        - switch.doorbell
        - binary_sensor.front_hallway
        - binary_sensor.hair_salon
        - binary_sensor.kitchen
        - binary_sensor.theatre_room
```

The above example shows the minimum needed to get it working using the defaults. It's configured to display a doorbell (i.e. switch) and a bunch of zones in the house (i.e. binary sensors).

The configuration below is an example of how to further customize the floorplan state card, making use of all the available fields.

```
homeassistant:
  customize: 
    binary_sensor.floorplan:
      custom_ui_state_card: floorplan
      floorplan_image: /local/custom_ui/floorplan/floorplan.svg
      password: <password_goes_here>
      stylesheet: /local/custom_ui/floorplan/floorplan.css
      color_on: '#F8B9BE'
      color_off: '#C4EDB1'
      track_duration: 10
      last_motion_entity: sensor.template_last_motion
      entities:
        - switch.doorbell
        - binary_sensor.front_hallway
        - binary_sensor.hair_salon
        - binary_sensor.kitchen
        - binary_sensor.theatre_room
```

Each field is described below:

|Field|Required|Description|Default|
|-|-|-|-|
|custom_ui_state_card|required|The name of the floorplan state card||
|floorplan_image|required|The path to the SVG image||
|password|optional|The Home Assistant password||
|stylesheet|optional|The path to the CSS file for styling the floorplan 
|color_on|optional|The color used to represent the 'on' state|'#F8B9BE'|
|color_off|optional|The color used to represent the 'off' state||
|track_duration|optional|The number of seconds to transition from 'on' to 'off' colors (if color_off defined)|10|
|last_motion_entity|optional|The name of the 'last motion' entity (if created above)||
|entities|required|The list of binary sensors and switches to display on the floorplan||

After restarting Home Assistant, you should be able to see your floorplan state card in action.
