
<div class="min-w-screen min-h-screen bg-black">
    <div class="w-3/4 mx-auto pt-20">
        <h1 class="font-bold text-white text-5xl">earn credits by doing jumping jacks</h1>


        <br>
        <p class="text-gray-500 italic">no video is ever sent to the server, i'm not smart enough to figure out how to do that. all processing is done locally on your device, using tensorflow.js.</p> 
        <br>

        <p class="text-gray-300 italic">you currently have {formatter.format(profile.balance)} in your account. <a class="underline hover:decoration-wavy" href="/">go home?</a></p>  
        <p class="text-gray-300 italic">not working? make sure both of your wrists and shoulders are in the frame.</p> 

        <video bind:this={video} width="640" height="480" class="rounded-lg my-10 outline-2 outline-white"/>

        <p class="font-bold text-white ">you have completed {formatter.format(repsCompleted * 0.01)} worth of jumping jacks.</p>

        <button class="bg-black text-white rounded-md border-2 border-white my-2 p-1" onclick={cashOut}>save to balance</button>

    </div>
    
</div>


<script>
    // @ts-nocheck
    // ts pmo

    export const prerender = true
    export const ssr = false // disable server-side rendering

    import { supabase } from "$lib/supabase";
    import { onMount } from 'svelte'
    import * as poseDetection from '@tensorflow-models/pose-detection'
    import '@tensorflow/tfjs-backend-webgl'
	import { all } from "@tensorflow/tfjs";

    let user

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

    onMount(async () => {
        // check if already signed in
        const { data: { session } } = await supabase.auth.getSession()

        
        
        if (!session) {
            // only create new anon user if not already signed in
            let {data, error} = await supabase.auth.signInAnonymously();
            user = data.user
        } else {
            user = session.user
        }

        const { data: profileData } = await supabase
        .from('profiles')          // which table
        .select('*')         // which columns (or '*' for all)
        .eq('id', user.id)         // WHERE id = user.id
        .single()

        profile = profileData



        initCounter()

    })
  

    /// JUMPING JACKS ///

    let video
    let detector
    let repsCompleted = $state(0)
    let isUp = false // track jumping jack state

    const scoreThreshold = 0.4

    async function initCounter() {
        // get webcam
        const stream = await navigator.mediaDevices.getUserMedia({ 
        video: true 
        })
        video.srcObject = stream
        await video.play()
        
        // load pose model
        detector = await poseDetection.createDetector(
        poseDetection.SupportedModels.MoveNet
        )
        
        // start detection loop
        detectPoses()
    }

      async function detectPoses() {
        if (!detector) return
        
        const poses = await detector.estimatePoses(video)
        
        if (poses.length > 0) {

            const keypoints = poses[0].keypoints
            
            // get wrist and shoulder positions
            const leftWrist = keypoints.find(kp => kp.name === 'left_wrist')
            const rightWrist = keypoints.find(kp => kp.name === 'right_wrist')
            const leftShoulder = keypoints.find(kp => kp.name === 'left_shoulder')
            const rightShoulder = keypoints.find(kp => kp.name === 'right_shoulder')
            
            // check if arms are up (jumping jack up position)
            const armsUp = 
                leftWrist.y < leftShoulder.y && 
                rightWrist.y < rightShoulder.y

            const confident = leftWrist.score > scoreThreshold && leftShoulder.score > scoreThreshold && rightWrist.score > scoreThreshold && rightShoulder.score > scoreThreshold

                if (armsUp && !isUp && confident) {
                    repsCompleted++
                    repsCompleted++
                    isUp = true
                    
                    
                } else if (!armsUp) {
                    isUp = false
                }

            

        }
        
        // loop
        requestAnimationFrame(detectPoses)
    }

    async function cashOut() {
       await supabase.rpc('add_workout_credits', { reps: repsCompleted }) 
       window.location.href = "/"
    }

    

    
</script>
