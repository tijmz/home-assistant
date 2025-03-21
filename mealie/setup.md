# Introduction

I added the meal planner Mealie as an add-on to my Home Assistant installation. This add-on allows me to add recipes to a database, plan meals using those recipes and easily generate a shopping list. For more, see [mealie.io](mealie.io). With the add-on installed, it is possible to generate an API key and to expose data to Home Assistant.

## Exposing meal of the day
Using a REST sensor, data from Mealie can be put to use in Home Assistant. To do so, add the contents of ``configuration.yaml`` listed here to the corresponding file in your Home Assistant installation. Make sure to add the proper path to your Mealie installation and include your API key.

## Exposing meal tomorrow
Another REST sensor scans for tomorrow's meal. This is done by calling the API for a list from the mealplan, using parameters ``start_date`` and ``end_date`` to select the relevant time slice. Note that to include tomorrow's meal, ``end_date`` needs to be two days from now.

## Generating a card that shows the meal of the day
Once you have the REST sensor, you can easily display an image of the meal of the day. I used templating in a header card that contained the following:

```yaml
{% if now().hour < 19 %}
## Today's dinner
<b> {{ states('sensor.meal_of_the_day') }}! </b>

<figure>
<img src="<URL to mealie>/api/media/recipes/{{states('sensor.meal_of_the_day_id')}}/images/min-original.webp" width="50%">
<figcaption><i>Looking delicious!</i>😋</figcaption>
</figure>

{{states('sensor.meal_description')}}
{% endif %}
```
## Cooking automation
I created a helper (Toggle/boolean) to track whether I am cooking or not, which I inserted on my dashboard as a toggle. If toggle is set to On, the automation ``run_cooking.yaml`` activates a script called ``script_cooking.yaml``. This script does several things:
1. It increases the speed of the comfo fan.
2. It plays music on kitchen speaker.
3. It waits for the Toggle to be turned off.
4. Then, it makes an announcement about dinner being served to relevant speakers.
5. It reset the speed of the comfo fan to normal

## Ode to cooking
I have one more script, which plays an ode to the meal via speaker if activated. For this I use the Google Gemini integration. This is ``gemini_ode_to_food.yaml``.
