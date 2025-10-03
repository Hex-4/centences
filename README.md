# centences

centences is a tiny little social experiment/joke, based on a twitter clone.

*it's the attention economy, but you're paying.*

start with $0.50 in credit. posts cost $0.01 per character. likes cost $0.03, and the money from them goes to the poster's balance. do jumping jacks (yes, really) to earn more credits because i'm too lazy to set up an actual Stripe account.

good luck.


[deployed here.](https://centences.pages.dev)


## credits

centences was built with [supabase](https://supabase.com/) and [sveltekit](https://svelte.dev/docs/kit/introduction).

tensorflow.js and the prebuilt pose detection model was used for jumping jack counting.

AI was used more heavily than usual in this project, due to the tight timeline and unfamiliar technologies. Claude generated SQL for most of the backend and helped with some debugging, as well as most of the jumping jack detection. 
