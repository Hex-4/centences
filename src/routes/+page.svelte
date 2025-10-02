
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

            start with 50c in credits. buy more (<strike>yes, actually, i set up a stripe account just for this</strike>*), post more. likes cost 2 cents. the money from a like goes to the poster's balance.

            yes, this is like the stupidest thing I have ever made. yes, I am going to make no money off this. yes, i'm probably going to get ddosed and hacked.

            but if people are stupid enough to pay for PNGs of monkeys, they're stupid enough to pay for a few centences.

            <br>
            <br>

            <b>* UPDATE: turns out setting up a stripe account is much harder than I thought. because I do not want to do tax stuff and make a privacy policy and terms of service just for this, you now buy credits by doing jumping jacks.</b>

        </p>

        <br>
        <p class="text-gray-300 italic">you currently have {formatter.format(profile.balance)} in your account. earn more credits by <a class="underline hover:decoration-wavy" href="earn">doing jumping jacks.</a></p> 

        <p class="text-gray-300 italic">your username is "{profile.name}". change it below:</p>

        <input type="text" class="bg-black text-white rounded-md border-2 border-white my-2 p-1">
        <button class="bg-black text-white rounded-md border-2 border-white my-2 p-1">save</button>
        <br>

        <textarea rows="3" cols="50" class="bg-black text-white rounded-md border-2 border-white my-2 mt-8 placeholder:italic placeholder:text-gray-400 p-1" placeholder="make a post..." bind:value={post}></textarea> 

        {#if post.length > 0}
            <p class="text-white whitespace-pre-line">
                this post has
                <b>{post.length}</b> characters,
                which means it will cost
                <b>{formatter.format(post.length * 0.01)}</b>.
                {#if (post.length * 0.01) > profile.balance}
                    <br>
                    <span class="text-red-400">
                        you don't have enough credits to post this! shorten your message, or do <a class="underline hover:decoration-wavy" href="earn">do jumping jacks</a> to earn more credits.
                    </span>
                    
                {:else}
                    <br>
                    this post will leave you with {formatter.format(profile.balance - post.length * 0.01)} in your balance.

                    <button class="bg-black text-white rounded-md border-2 border-white my-2 p-1" onclick={newPost}>post it!</button>
                {/if}
            </p>
        {/if}

        <hr class="border-white my-8">
    </div>
    
</div>


<script>
    import { supabase } from "$lib/supabase";
    import { onMount } from 'svelte'

    let user

    let post = $state("")

    let preferredName = $state("")

    let profile = $state("loading...")

    let allPosts

    // Create our number formatter.
    const formatter = new Intl.NumberFormat('en-US', {
        style: 'currency',
        currency: 'USD',

        // These options can be used to round to whole numbers.
        trailingZeroDisplay: 'stripIfInteger'
    });

    // helper for "time ago" format
    function getTimeAgo(timestamp) {
        const date = new Date(timestamp)
        const seconds = Math.floor((new Date() - date) / 1000)
        
        if (seconds < 60) return `${seconds}s ago`
        
        const minutes = Math.floor(seconds / 60)
        if (minutes < 60) return `${minutes}m ago`
        
        const hours = Math.floor(minutes / 60)
        if (hours < 24) return `${hours}h ago`
        
        const days = Math.floor(hours / 24)
        return `${days}d ago`
    }

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

        allPosts = await supabase
        .from("posts")
        .select("*")
        .order("created_at", { ascending: false })

        console.log(allPosts.data)

        profile = profileData
    })

    async function newPost() {
        await supabase.rpc('create_post_with_payment', { content: post })
        window.location.href = "/"
    }

    

    
</script>
