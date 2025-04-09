# Introduction
Each morning I check the todo's of the day and frankly, it's a bit boring. So I decided to create a script that would grab these tasks, make a story about them and read them to me like a peptalk. Then I decided to have it run every morning, but I somehow think I will get rid of that portion soon.

## What you need
- **Todoist and its integration.** I use the app Todoist to manage my tasks and so I installed the [https://www.home-assistant.io/integrations/todoist/](Todoist integration) first. I then **disabled** all the calendars and todo lists, because what I needed was a custom project. Now you might think a custom project would lead to a custom todo list, but that it not the case: the integration allows you to add a custom calendar to ``configuration.yaml``. You can use the same API key as you used when installing the Todoist integration to set up this custom calendar.
- **Google Gemini and its integration.** I had Google Gemini generate the text. Set up the [https://www.home-assistant.io/integrations/google_generative_ai_conversation/](Google Generative AI integration) and you're ready to go.
- You need some playback device that can be reached by Home Assistant

## Setting it up

1. Add the Todoist API key to ``secrets.yaml``
2. Add and adapt the content of the ``configuration.yaml`` file in this folder to set up a custom calendar based on Todoist. For more details on options you can use, see the documentation of the Todoist integration.
3. Create a script that follows ``script_peptalk.yaml``. Adapt to make it fit with your own requirements, such as your own media player.
4. Automate it (e.g. you can use the timer from ``automation_peptalk.yaml``, but perhaps your use case varies).
