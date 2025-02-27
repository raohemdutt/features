<script lang="ts">
	import { page } from "$app/stores";
	import Studio from "$lib/components/Studio/Studio.svelte";
	import { json } from "@sveltejs/kit";
	import { onMount } from "svelte";
    import marked from "$lib/markdown/markdown.js";
	import messageStore from "$lib/stores/message.store";
	import { decompressByteArray } from "$lib/utils/compress.js";
	import { decode } from "@msgpack/msgpack";
	import Codecast from "$lib/components/Codecast/Codecast.svelte";
	import { draw } from "svelte/transition";
	import { browser } from "$app/environment";

    export let data;
    let { supabase, session, section, markdown } = data
    $: ({ supabase, session, section, markdown } = data) // listen to changes

    let showStudioView = false;
    $: if (showStudioView === false) {
        loadSection($page.params.sectionId)
    }

    let embedId = "";
    $: loadSection($page.params.sectionId);

    let duration:number = section?.section_codecasts ? section?.section_codecasts[0]?.duration : 0.0;
    let audioURL:string;
    let changes:{
		time: [],
		text: [],
		selection: { head: [], anchor: [] }
	};
    async function loadSection(sectionID:String) {
        const{data} = await supabase.from("sections").select("*, section_embeds(*), section_codecasts(*)").eq("id", sectionID)
        if (data)
        {
            section = data[0];
            if (section?.markdown) {
                markdown = await marked.parse(section.markdown)
            }

            if (section?.section_embeds.length > 0) {
                if (!section.section_embeds[0].url) return;

                const url = new URL(section.section_embeds[0].url);
                if (url.hostname.includes("youtu")) {
                    const v = url.searchParams.get("v")
                    if (v) {
                        embedId = v;
                    } else {
                        messageStore.showError("Url missing video id.")
                    }
                }
            }

            if (section?.section_codecasts.length > 0)
            {
                const section_codecast = section.section_codecasts[0];
                const path = `courses/${section.id}`;
                const changesPath = `${path}/codecast.mpack.gz`
                const audioPath = `${path}/codecast.mp4`

                const { data:audioData } = supabase.storage.from('codecasts').getPublicUrl(audioPath)
                const { data:changesData, error } = await supabase.storage.from('codecasts').download(changesPath)
                if (error) {
                    console.error(error);
                    messageStore.showError(error.message);
                }

                if (audioData && changesData) {
                    const mpack = await decompressByteArray(new Uint8Array(await changesData.arrayBuffer()), "gzip");
                    changes = decode(mpack);
                    audioURL = audioData.publicUrl;
                    duration = section_codecast.duration;
                }
            }
        }
    }
</script>

<div class="overflow-scroll">
    {#if section?.section_codecasts.length > 0 && browser}
        <Codecast bind:recordingLengthSeconds={duration} bind:audioURL={audioURL} bind:changes={changes}></Codecast>
    {/if}
    {#if section?.section_embeds.length > 0}
        <iframe style="width:100%;" 
        class="aspect-video max-h-[80vh]" 
        src="https://www.youtube.com/embed/{embedId}?si=Jtz7-tzqf7Z8JU9M&showinfo=0&modestbranding=1&autoplay=1" 
        title="YouTube video player" frameborder="0"
        allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" 
        allowfullscreen></iframe>
    {/if}

    <div class="p-2 flex min-w-max">
        <button on:click={() => {showStudioView = true}} class="flex items-center justify-center ml-auto hover:opacity-100 opacity-70 text-gray-200 p-1 px-2 rounded">
            <span>Edit </span>
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16" fill="currentColor" class="w-4 h-4">
                <path d="M13.488 2.513a1.75 1.75 0 0 0-2.475 0L6.75 6.774a2.75 2.75 0 0 0-.596.892l-.848 2.047a.75.75 0 0 0 .98.98l2.047-.848a2.75 2.75 0 0 0 .892-.596l4.261-4.262a1.75 1.75 0 0 0 0-2.474Z" />
                <path d="M4.75 3.5c-.69 0-1.25.56-1.25 1.25v6.5c0 .69.56 1.25 1.25 1.25h6.5c.69 0 1.25-.56 1.25-1.25V9A.75.75 0 0 1 14 9v2.25A2.75 2.75 0 0 1 11.25 14h-6.5A2.75 2.75 0 0 1 2 11.25v-6.5A2.75 2.75 0 0 1 4.75 2H7a.75.75 0 0 1 0 1.5H4.75Z" />
            </svg>
        </button>
    </div>
    
    <div class="mx-auto p-3 markdown-body">
        {@html markdown}
    </div>
</div>

{#if showStudioView}
    <Studio {supabase} section={section} bind:showStudioView={showStudioView}></Studio>
{/if}

<style lang="scss">
    .max-h-video {
        max-height: calc(90vh);
    }

    .markdown-body {
        box-sizing: border-box;
		min-width: 200px;
		max-width: 980px;
		margin: 0 auto;
		padding: 45px;
        background-color: transparent;
    }

    .markdown-body ul {
        @apply list-disc;
    }
    
    .markdown-body li::marker {
        content: initial;
    }

    @media (max-width: 767px) {
		.markdown-body {
			padding: 15px;
		}
	}
</style>