<script>
    let { content = "skibidi dop dop yes yes skibidi dop dop yes yes yes skibidi toilet skibidi toilet skibidi toilet yes", userid = "", timestamp = "2025-10-02 02:18:05.987733+00", cost = 0.67, likes = 67, id = "a" } = $props();

    import { supabase } from "$lib/supabase";
    import { onMount } from 'svelte'

    // Create our number formatter.
    const formatter = new Intl.NumberFormat('en-US', {
        style: 'currency',
        currency: 'USD',

        // These options can be used to round to whole numbers.
        trailingZeroDisplay: 'stripIfInteger'
    });

    let userProfile = $state()

    onMount(async () => {

        const { data: profileData } = await supabase
        .from('profiles')          // which table
        .select('*')         // which columns (or '*' for all)
        .eq('id', userid)         // WHERE id = user.id
        .single()

        console.log(profileData)

        userProfile = profileData
    })


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
</script>
<div class="ml-4 w-140 my-8">
    <div class="flex justify-between flex-row text-gray-400 italic mb-1">
        <span class="text-gray-400 italic">@{userProfile?.name ?? 'loading...'} spent {formatter.format(cost)}</span>
        <span class="text-gray-400 italic">{getTimeAgo(timestamp)}</span>
    </div>
    <p class="text-white whitespace-pre-wrap text-wrap">{content}</p>


</div>
