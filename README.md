# 5 Day Generative AI | Capstone Project Blogpost
My capstone project upon completing a 5-day intensive course by KagglexGoogle on using and building GenAI tools.

# 🤯 My 72-Hour Sprint to a Personal French Vocabulary Learning Assistant 🇫🇷

## The "Why" (or Maybe "Whyyyy?")

Let's be honest, the code you're about to (maybe) skim through isn't winning any awards. Picture this: You are a management consultant at KPMG, it is the end of March and the beginning of April, and you have a pile of work to complete as 5 separate projects are en route at the same time. And what did yours truly decide to do? Enroll in a 5-day Generative AI intensive course WITH a 2-week capstone project in a field brand new to you. Genius, right?

With the minimal Python knowledge I have, zero experience in anything API related, a fear that AI will take over my job so I need to learn it, and ✨a dream✨, I managed to put together this project in 3 days (which is literally the only free time I can squeeze out before I go back to work tomorrow), although the project isn't due until 2 weeks. So, if the code looks like it was written by a sleep-deprived, caffeinated squirrel learning Python and Gemini API on a sugar rush... well, you're not entirely wrong.

## The (Slightly Hectic) Use Case: Personalized French Learning

When I received the project descriptions 3 days ago (which was so open-ended I had no grounds to brainstorm on), I thought about what I can actually use. And since I'm going to France to study AI next year (pretty ironic, or coincidental?), why not a French vocabulary learning bot?

Despite the chaotic development, the core idea is pretty straightforward: **making French vocabulary learning more personalized and interactive using the magic of Generative AI.**

Tired of generic vocabulary lists? This project aims to create a conversational assistant that:

1.  **Understands your level:** It starts by chatting with you to gauge your CEFR French level (A1-C2).
2.  **Offers flexible learning:** You can choose to learn new words tailored to your level (active mode) or test your existing knowledge (passive mode).
3.  **Leverages AI for content:** The assistant uses a Large Language Model (LLM) to generate relevant vocabulary, example sentences, and even short writing exercises, allowing for dynamic conversational interaction with the AI instead of fixed touchpoints processes.

Think of it as a friendly, albeit slightly frantic, AI tutor in your pocket.

## Glimpse into the Engine Room (Code Snippets - Brace Yourselves!)

I did manage to pull off a full explanation of how the code works in the Jupyter Notebook (or at least how I understand it), so you can take a look at that for deeper insights. But here's a peek at how the AI manages to actually do things, instead of just saying fancy words (which by itself is already quite a marvel, if you think about it).

**Tool Definition (@tool):**

Before anything, the bot has to know the user's language level to tailor-made the lesson. To do this, not only does it have to understand the user's input (which the language model will handle), but it also has to update the `UserState` class, which you can imagine is the user themselves traversing through the flow of the session. To do this, tools are given to the agent. It may sound scary but these tools (hopefully) won't be enough for the AI to take over the world (or my job) just yet, since they have to be predefined by me (even scarier).

To do this, a ✨fake✨ python function is defined:

```python
@tool
def set_user_level(level: str) -> str:
    """Set the user's CEFR level as "A1", "A2", "B1", "B2", "C1", or "C2" according to user's response.
    Returns:
        The updated CEFR level of the user, and begin the content generation stage.
    """
```

**Actual Processing:**

The `@tool` function itself doesn't do anything, since the module LangGraph (used to build the flow of information) doesn't allow the AI to directly update the `UserState`, but only make a "tool call" where it says that it wants to use which tool(s). And then that information gets passed onto a node (again, defined my me) to then perform that action for the AI.

That node looks something like this:

```python
def process_tool_calls(state: UserState) -> UserState:
    """The user state node. This is where the user state is manipulated."""

    tool_msg = state.get("messages", [])[-1]
    
    for tool_call in tool_msg.tool_calls:
    #... various other tools #
        elif tool_call["name"] == "set_user_level":
            if tool_call["args"] and isinstance(tool_call["args"]["level"], str):
                level = tool_call["args"]["level"]
                state["level"] = level
                response = f"The user's CEFR level has been set to {level}. If no learning mode is selected yet, ask the user to choose one."
                stage = "CONTENT"
                state["stage"] = stage
            else:
                response = "Invalid arguments for set_user_level."
    #... more various other tools #

    return {"messages": outbound_msgs, "learned_vocab": learned_vocab, "mode": mode, "level": level, "route_to": route_to, "stage": stage}
```

This function then extracts what the AI want to do, the relevant parameters (args), update the user's level, then tell the AI that it has successfully done so. So no taking over the world yet today!

## The Elephant in the Room (Limitations and Future Dreams)

This whirlwind development process has highlighted a few areas that could be significantly improved with more time (and sleep):

- **Resource Intensive (Token Gobbler):** The current approach likely burns through tokens faster than a tourist in a Parisian bakery. Each turn involves feeding the LLM a significant chunk of conversation history, a lengthy prompt, which can get costly and hit context window limits. Some testing indicates that one API call currently costs me 1,000 tokens each (which I think is a lot, can't say for sure though because I don't even have time to think about efficiency yet, and thankfully Gemini is still free, for now)

- **State Management Spaghetti:** Having both `route_to` and `stage` in the UserState to handle similar aspects of the conversation flow is, shall we say, less than ideal. It adds unnecessary complexity and makes the routing logic a bit of a tangled mess. Originally they were meant to do different things, but one day in and I realized it's a bit more than I can possibly chew in 3 days, so bye bye optimization. You have to understand, with my minimal Python experience, writing new code will probably be faster than debugging existing mess.

- **Error Handling (Mostly Wishful Thinking):** Robust error handling? In a 72-hour sprint? Let's just say we're operating on a "fingers crossed" basis. I did manage to make it easier for myself to set a `debug` global variable though. I have no idea if it is best practice or not, probably not, nobody told me to do this (or not to do this in that matter). But I used it, and it was helpful enough for me to tell my code whether I want to see the massive wall of current-con'text', or just the pretty conversation between my and my best friend Frenchie (yes I called the bot Frenchie, but you can change it to whatever you want, there's a global variable for that, it's ✨personalized✨ after all).

- **Injection Security:** I just saw a YouTube video on secured code, so I feel like I need to write something on this too. Currently there are a whole bunch of variables that are just existing and innocently minding its business, doing its job and feeding into the Gemini model. Again, "fingers crossed", don't take advantage of my code, pretty please.

## The Art of the Possible (Future, Sleep-Permitting Edition):

Given more time and a few solid nights of sleep, here's what could be on the horizon:

- **Token Optimization:** Implement strategies to reduce token usage per API call, such as summarizing conversation history or being more selective about the context passed to the LLM. I think there's also a thing called ✨cache✨, no idea what it is, but it sounds like something that can help me (not sure though).

- **Unified State Management:** Refactor the UserState to use a single, clear mechanism for controlling the conversation flow (farewell, redundant route_to and stage!). But this may require me to rewrite the routing and staging logic so that the system understand which step it currently is, so you probably won't be seeing this soon (or ever).

- **Some other things that Gemini told me to include**: 
+ Robust Error Handling: Implement proper error handling and recovery mechanisms.
+ Comprehensive Testing: Write unit and integration tests to ensure the assistant functions reliably.
+ Personalized Learning Paths: Introduce more sophisticated personalization based on user performance and learning preferences.
+ Integration of External Knowledge: Connect the assistant to external resources for richer vocabulary and context.

## The Grand Finale (for Now...)
So there you have it – a Personal French Vocabulary Learning Assistant born out of a beautiful (and slightly insane) commitment to a 5-day intensive course during the most inconvenient time imaginable. The code might not be pretty, but the spirit (and the caffeine levels) were high.

## Appreciations

- Thanks **Google and Kaggle team**, for providing this free course, allowing non-CS-majors like me to take a dip in this territory to at least know how AI can take my job.
- Thanks **The 1975**, for the live album *Still... At Their Very Best (Live From The AO Arena, Manchester, 17.02.24)*, which was on repeat as I coded along the saxophones.
- Thanks **GrabFood**, for feeding me so I don't starve when I don't have time to cook (although it was very unhealthy and it still costs money, not recommended).
- Thanks **Starbucks** and **The Coffee House**, for providing me with a public place that I can sit and be surrounded by people so I don't just stay at home with my bed right next to my desk. (And sorry for camping 6 hours each day drinking 1 cup of Matcha Latte).
- Thanks **Mom and Dad**, for providing half the fund needed to buy my Macbook (which is ridiculously expensive), which has since allowed me to vibe my way into coding.
- Thanks **me**, for having to go through this unhealthy (but very fun) experience. Hopefully you got something out of this (you sure did get a tool to learn French)!

And thank you all, for (briefly) exploring this 72-hour adventure in Gen AI.

Now, if you'll excuse me, I hear a pillow calling my name... Bonne nuit!