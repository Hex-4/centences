
<div class="min-w-screen min-h-screen bg-black">
    <div class="w-3/4 mx-auto pt-20">
        <h1 class="font-bold text-white text-5xl">centences</h1>

        <p class="text-white whitespace-pre-line mt-5">hi.

            welcome to centences. this is a joke/social experiment/terrible business idea based on the following premise:
            <br>
            <br>
            <i>
                twitter, but every character costs one cent to post. or, the attention economy, but you're the one paying.
            </i>

            <br>
            <br>

            start with 50c in credits. buy more (yes, actually, i set up a stripe account just for this), post more. likes cost 2 cents. the money from a like goes to the poster's balance.

            yes, this is like the stupidest thing I have ever made. yes, I am going to make no money off this. yes, i'm probably going to get ddosed and hacked.

            but honestly? silence has never been cheaper.

        </p>

        <br>
        <p class="text-gray-300 italic">you currently have {formatter.format(profile.balance)} in your account.</p>

        <p class="text-gray-300 italic">your username is "{profile.name}". change it below:</p>

        <input type="text" class="bg-black text-white rounded-md border-2 border-white my-2 p-1">
        <button class="bg-black text-white rounded-md border-2 border-white my-2 p-1">save</button>
    </div>
    
</div>


<script>
    import { supabase } from "$lib/supabase";
    import { onMount } from 'svelte'

    let user

    let preferredName = $state("")

    let profile = $state("loading...")

    // Create our number formatter.
    const formatter = new Intl.NumberFormat('en-US', {
        style: 'currency',
        currency: 'USD',

        // These options can be used to round to whole numbers.
        trailingZeroDisplay: 'stripIfInteger'
    });

    onMount(async () => {
        // check if already signed in
        const { data: { session } } = await supabase.auth.getSession()

        user = session.user
        
        if (!session) {
            // only create new anon user if not already signed in
            let {data, error} = await supabase.auth.signInAnonymously();
            user = data
        }

        const { data: profileData } = await supabase
        .from('profiles')          // which table
        .select('*')         // which columns (or '*' for all)
        .eq('id', user.id)         // WHERE id = user.id
        .single()

        profile = profileData
    })

    

    
</script>
