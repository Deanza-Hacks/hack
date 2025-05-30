# Design Document

## Introduction

The project is a financial literacy and DEI/social justice education platform
disguised as a cookie clicker game.

The primary purpose is to educate how compound interest can have an enormous
impact on personal finances, both positively and negatively.  The secondary
purpose is to enlighten users about how systemic issues experienced by
other demographics interact with compound interest and in turn, with
personal finances.

The judging criteria are 1) creativity, 2) technical complexity, and 3) social
impact.

Creatively, the game will have memorable character classes for the player to
choose from, including at least one that's so eyecatching and ridiculous that
it will immediately hook the player from the initial character selection
screen.  The game's theme will be a parody of modern life in terms of work,
bills, wealth, and unexpected (mis-)fortunes.  Although the tone of the content
will be serious, the characters, art, and occasional parts of the gameplay loop
will deliver that content in a comedic way to fulfill the primary purpose of
the project.

In terms of technical complexity, simply building anything meeting the minimum
design requirements in 9 hours will likely be impressive for the mean
experience level of the team.  Building graphical applications this quickly
is difficult, but we have an ace up our sleeve.

The social impact criteria will be fulfilled by the secondary purpose of the
game.

## User Story

As a judge, I start the game and see an immediately eye-catching character
selection screen.  At least one character is obviously not supposed to be
taken seriously, but each of the others are interesting in their own way.

Once I start the game, I see a simple but engaging UI which explains that the
game simulates a character about 21-25 years old, who had an early life which
I can either easily identify with, or has a story with a hook that propels me
to want to know more about the hook.  ("Sure, you're a hobo, but before that
you were an actual Nigerian Prince and no one belives you.")

The intro UI also explains that the game is about making choices that affect
the character's finances, especially in terms of compound interest, and that
the choices are based on personal finance experiences involving compound
interest that many people don't understand the practical implications of.

Finally, the UI shows me the character's current financial situation,
including their current income, expenses, and net worth, and explains that
the goal of the game is to increase the character's net worth ("level up")
by keeping income higher than expenses, and by making good selections when
presented with choices such as home and student loans, credit cards, REITs,
cryptocurrency, and other financial instruments with compound interest.

I like the catchy label on the "do work" button, which makes it clear that
clicking it will start the game by making my character do work and earn money.

I'm highly motivated to keep clicking the button because bills appear and
take my hard-clicked money away.  Some of the bills are for funny things, but
others are for things that I can identify with, such as student loans, credit
cards, and rent.

I also like the way the game presents choices, because although they start out
serious, some of them are callbacks to the hooks built into each character's
design, and start to tell an interesting side story that makes me want to keep
clicking the button to see how the side story plays out.

I like how the art, although simple, is colorful, smoothly animated, and
lampshades the time pressure of the hackathon.  Any placeholder art contains
at least humorous and memorable copy.

When my character levels up, I'm surprised and delighted by the colorful and
festive UI which celebrates the achievement.  I also like how my character
is described as having become more successful, and (without deducting from
their net worth) is given a memorable and catchy new job title, even if
my character doesn't have an actual job.  (Ex: "Executive Hobo")

As the midgame approaches, I can begin to see how compound interest has caused
my early decisions to have a larger impact on my character's finances than I
thought they would, because of how they were described.  I notice that
some of those impacts are positive, such as how my character's
investments are growing, but others are negative, such as how my character's
credit card debt is growing, and how my character's student loans are
growing faster than my character's income.

I also notice that the game is starting to present me with more complex
choices, such as whether to take out a second mortgage on my character's
home, or whether to invest in a REIT that has a high yield but is also
highly volatile.  I like how these choices are presented in a way that
makes me think about the long-term implications of my decisions, and how
they will affect my character's finances in the future.

I am excited to see that my character is approaching the apex of their life
path, which is either filthy rich or so destitute it causes a collapse of the
financial system.  I am also highly interested in how the side story for my
character will resolve, independently of their financial situation.

I feel a sense of accomplishment when I reach the end of the game, but the
end game UI is also a bit sobering because it reminds me of choices I may have
made in the past, or I will have to consider in the future.  I want to apply
the lessons learned in the game to those choices, and try to rectify previous
choices.

If I chose a character I can identify with, I am also enlightened by how their
demographic had systemic issues that affected their finances, both positively
and negatively.  I want to be more considerate of those issues both in my own
life and in the lives of others.  The game presented my character's experience
in a way that was entertaining, and I want to share it with others.

## Technical Design

The team has previous experience with [Flask](https://pypi.org/project/Flask/)
and [Love2d](https://love2d.org/).

Initial discussions leaned in the direction of having a completely self-
contained SPA using the localstorage API for persistence.

The game design requires the following:

- Frontend:
    - Fullscreen canvas which can display full screen animations for key game
    events.  When not displaying a full screen animation,
        - A sidebar with the character's current financial situation, including
        income, expenses, and net worth
        - A main content stage (in the theater sense)
        - Event queue
        - Navigation bar with breadcrumbs
    - API for calling UI callbacks when the backend event queue dispatches
    messages of interest to the UI.
    - Each dynamic UI element shall define a reset function that will reset itself based on state stored in per-element objects, and the current game and character state.
    - Static assets.
        - Character sprites.
        - Event sprites.
        - Fonts.
        - Backgrounds.
        - UI elements.
    - Animation library for assets.

- Backend:
    - Event queue represented by an EventTarget object associated with the frontend UI element for the event queue.
        - Consumers use setEventListener to listen for messages.
            - Non-UI consumers and producers will be represented by a hidden DIV with a 1:1 association to a class representing itself.
                - These will have visibility: hidden, but when displayed for
                debugging, will show their message log.
        - Producers use dispatchEvent to inject messages into the queue.
    - Master clock.
        - Driven at 1 Hz by setInterval.
        - Primarily used to schedule delayed events paired with an originating
        event, such as a "payday" event that occurs every 30 seconds.
        - Targets a 5, 10, or 15 minute game session (startup option: short, medium, or long game, doubling as difficulty setting), emitting a floating point number indicating the percentage completion of the game.  Note that the game session is necessarily bounded due to the real-time nature of how bills accumulate, and the user will automatically lose if they run out of money.
        - Pause feature.  Because all event consumers and producers are driven by the master clock, it will not be necessary to pause them individually.
    - Character model event consumer with a FSM that changes state based on messages in the event queue.  Driven by the event queue and a callback from the master clock.
        - Character state includes:
            - Current income
            - Current expenses
            - Current net worth
            - Current job title
            - Current character class
            - State of the character's side story
        - Serialization of the character state to JSON for persistence in localstorage.
        - Event FSM driver for event messages which are part of a shared event FSM state.
    - Game model event producer with a FSM that does goal-seeking to reach a
    terminal FSM state.  Also driven by the event queue and a callback from the master clock.
        - Randomish but user-friendly random event generator (e.g. don't have the same event happen twice in a row)
        - Most events will be represented as a bell curve probability function of the master clock centered around a particular phase of the game.  E.g. f(x) = 1/(σ√(2π)) e^(-(x-μ)²/(2σ²)), where μ is the current game phase and σ is a small number representing the width of the bell curve.  (I have no idea if this is right, this came from Claude 4.)
        - Serialization of the game state to JSON for persistence in localstorage.
    - UI elements will dispatch events directly to the event queue.  UI elements will not directly dispatch events to other UI elements.
    - Event database.
        - UI:
            - List of:
                - Animation function
                - UI element target(s) (if any, or main content stage, or fullscreen)
                - Assets compatible with that animation function
        - Textual description implemented as templatized text, like handlebars or jinja2 or something.  Template variables come directly from the game and character state.  This will be displayed verbatim in the event queue element.
        - Optional reference to a FSM by name and corresponding state change logic.  This will be called by the character model when events tagged with this FSM name are dispatched to the event queue.
        - Start callback function implementing game and character state changes represented as messages produced into the event queue.
        - One of:
            - No stop callback function, which means the event is a one-shot event that will be removed by the event queue by a consumer, or
            - Optional time delay and stop callback function which will be used by the master clock to produce any state change messages, and an API call to the event queue to remove the event.
            - Cron-style scheduling for recurring events, such as bills.
                - Note that each of these must almost certainly need a stop callback function, because the event queue will not be able to remove them automatically.
    - FSM database.  Each object will have:
        - Name
        - List of states
        - List of transitions
            - Source state
            - Target state
            - Event trigger(s)
            - Event dispatcher(s), such as broadcast events
            - Callback function to be called when the transition occurs
        - Start state
        - Types:
            - Game
            - Character
            - Events
    - Character database.
        - List of:
            - Character class
            - Character class description
            - Character class assets (both nav and hero)
            - Initial character state
                - Job
                - Income
                - Expenses
                - Net worth
                - Buffs/nerfs from the event database
            - List of events to be enqueued at startup for the character side story
            - List of events to be enqueued at startup for the character's financial situation

## Story Design

Characters:
- Buffs and nerfs compatible with the compound interest theme.  Each character's buffs and nerfs should be complementary and different from the others.
- A story for how they naturally benefit from compound interest.
- A story for how they are disproportionally negatively affected by compound interest.
- Midgame goal (fancy car, luxury hobo cardboard palace, artesinal NFT...).
- Endgame goal (retirement, financial independence, maybe the character *wants* to do a fight club so utopian communism is their *bad* ending...).
- Side story.
- Midgame transition.
- Endgame transition.
- Good ending.
- Bad ending.f
- Event databases should be designed to gradually increase exponential coefficients of probability functions of the master clock, so the game is guaranteed to end close to but not after the master clock hits 100%.
- Let's try not to have the game end with a cliffhanger, but rather with a satisfying conclusion to the character's story, even if it is a bad ending.  The player should feel like they learned something from the character's experience, even if it was a negative one.
- The character's side story should be independent of the character's financial situation, so the player can see how the character's choices affect their finances, but also how the character's life story plays out independently of their finances.
- Instead of perpetuating stereotypes, the characters should be designed to subvert them, so that the player is surprised by the character's story and how it relates to their finances.  E.g. the majority prestige demographic in the real world is presented as the hobo.  Let's play this completely straight with no self-referential awareness.  Just have the text and assets wildly subvert expectations.
- Maybe something about the Indian class system would resonate with the target audience, idk.

# Notes:
- The reason the user and game states are independent is so the user can save the character for a NG+ feature allowing them to revisit previous choices.
- Storytelling will be built out of events from the event database, optionally paired with a FSM specific to that story in the game or character FSM database.

