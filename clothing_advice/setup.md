# Introduction
``clothing_advice.yaml`` is a script I made for my daughter and which she can call via a voice command. I adapted it to make use of Home Assistant's AI Task action, rather than directly calling a specific LLM. What the script does is bring in the weather forecast, recombine it with a list of items (available clothing) and then provide clothing advice for the day.

I left out a lot of concrete particulars here, to respect my daughter's privacy. One thing to note is that the script works better if a) there are plenty of items to choose from and b) the items are listed as descriptively as possible.

# Known issues
- LLMs can repeatedly gravitate towards specific items if those items are strongly associated with a weather type. E.g. a summer dress is likely to picked again and again during sunny days. Tweaking the description can stave off this attractor state, as can competing items.
- The AI task action does not allow specification of a path that containts a .txt of the clothing items, which is why they are declared in the prompt instructions. My more roundabout script did call a separate file, which was more easily maintained by my daughter.
# Future plans
- AI Task supports multimodal prompting, so images of clothing could also work. Submitting all clothing items might be heavy duty prompting, but perhaps a camera could be used to assess appropriateness of a specific choice of outfit, or to compare two or three distinct sets. I'd like to play with such options if I ever connect a camera to my setup.
- Running a local LLM with access to local files might be a way to fetch the inventory of clothing on the fly from something that can be maintained more easily than updating the script.
