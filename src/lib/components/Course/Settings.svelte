<script lang="ts">
	import messageStore from "$lib/stores/message.store.js";
	import { decodeStream } from "@msgpack/msgpack";
	import type { Session, SupabaseClient } from "@supabase/supabase-js";
	import { onMount } from "svelte";

    export let showSettings:boolean;
    export let course:any;
    export let supabase: SupabaseClient;

    let originalTitle = course?.title
    let originalDescription = course?.description;

    let bannerInput;
    let bannerImage;

    let publicBannerImageUrl;
    
    let loading = false;

    onMount(() => {
        if (course.banner_path) {
            publicBannerImageUrl = supabase.storage.from("courses").getPublicUrl(course.banner_path).data.publicUrl
        }
    })

    function resize(event) {
        if (!event.target) return;

        event.target.style.height = 'auto';
        event.target.style.height = event.target.scrollHeight + 'px';
    }

    async function onDropImage(event) {
        event.preventDefault();
        
        if (!event.dataTransfer.files) return;

        const file = event.dataTransfer.files[0]

        if (!file.type.includes("image")) return;

        loading = true;

        const reader = new FileReader();
        reader.addEventListener("load", function() {
            publicBannerImageUrl = reader.result;
            bannerImage.setAttribute("src", reader.result)
        });

        messageStore.showLoading("Uploading...", () => loading)

        const bannerPath = `${course.id}/banner.${file.type.split('/')[1]}`;
        const result = await supabase
        .storage
        .from("courses")
        .upload(bannerPath, file, {
            upsert: true
        })

        const result2 = await supabase.from("courses").update({
            banner_path: bannerPath,
        }).eq("id", course.id)

        // once uploaded show preview
        reader.readAsDataURL(file);

        loading = false;
    }

    async function onChangeBanner(event) {
        if (!event.target.files) return;

        const file = event.target.files[0]

        if (!file.type.includes("image")) return;

        loading = true;

        const reader = new FileReader();
        reader.addEventListener("load", function() {
            publicBannerImageUrl = reader.result;
            bannerImage.setAttribute("src", reader.result)
        });

        messageStore.showLoading("Uploading...", () => loading)

        const bannerPath = `${course.id}/banner.${file.type.split('/')[1]}`;
        const result = await supabase
        .storage
        .from("courses")
        .upload(bannerPath, file, {
            upsert: true
        })

        const result2 = await supabase.from("courses").update({
            banner_path: bannerPath,
        }).eq("id", course.id)

        // once uploaded show preview
        reader.readAsDataURL(file);

        loading = false;
    }

    async function saveChanges() {
        await supabase.from("courses").update({
            title: course.title,
            description: course.description,
        }).eq("id", course.id)

        originalTitle = course.title
        originalDescription = course.description;
    }
</script>

<svelte:head>
    <title>{`Dashboard / ${course.title}`}</title>
</svelte:head>

<div class="bg-[--dark-800] rounded-lg fixed top-0 left-0 right-0 bottom-0 z-40 overflow-y-scroll">
    <div class="p-5 flex gap-3">
        <div>
            Settings for "{course.title}"
        </div>
        <div class="ml-auto">
            <button on:click={() => {showSettings = false}}>Save</button>
        </div>
    </div>
    <div class="p-5 flex flex-col gap-3">
        <label for="banner" class="max-w-xl" ondragover="return false" on:drop={onDropImage}>
            <div class="uppercase text-xs font-bold py-2">Banner</div>
            {#if publicBannerImageUrl}
                <div
                    bind:this={bannerImage}
                    style="background-image:url({publicBannerImageUrl})"
                    class="bg-cover bg-left max-h-52 rounded-lg aspect-video overflow-hidden flex flex-col gap-3 border border-dashed border-gray-200 w-full bg-[--dark-700]"
                >
                    <div class="h-full w-full bg-gradient-to-r from-black via-70% via-transparent to-transparent p-4 flex flex-col">
                        <h3 class="text-white text-3xl font-medium md:text-4xl p-3 w-2/3">{course.title}</h3>
                    </div>
                </div>
            {:else}
                <div bind:this={bannerImage} class="bg-cover bg-left md:max-h-52 rounded-lg aspect-video overflow-hidden flex flex-col gap-3 border border-dashed border-gray-200 w-full bg-[--dark-700]"
                >
                    <div class="h-full w-full bg-gradient-to-r from-black via-70% via-transparent to-transparent p-4 flex md:flex-row flex-col">
                        <h3 class="text-white text-3xl font-medium md:text-4xl p-3 w-2/3">{course.title}</h3>
                        <div class="flex flex-col content-center justify-center text-center">
                            <div class="mx-auto">
                                <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6">
                                    <path stroke-linecap="round" stroke-linejoin="round" d="M2.25 15.75l5.159-5.159a2.25 2.25 0 013.182 0l5.159 5.159m-1.5-1.5l1.409-1.409a2.25 2.25 0 013.182 0l2.909 2.909m-18 3.75h16.5a1.5 1.5 0 001.5-1.5V6a1.5 1.5 0 00-1.5-1.5H3.75A1.5 1.5 0 002.25 6v12a1.5 1.5 0 001.5 1.5zm10.5-11.25h.008v.008h-.008V8.25zm.375 0a.375.375 0 11-.75 0 .375.375 0 01.75 0z" />
                                </svg>
                            </div>
                            <div>click to upload image</div>
                        </div>
                    </div>
                </div>
            {/if}
            <input
                name="banner"
                bind:this={bannerInput}
                on:change={onChangeBanner}
                id="banner"
                type="file"
                multiple
                accept=".jpg, .jpeg, .png, .webp"
                class="hidden"
                required
            />
        </label>
        <label>
            <div class="uppercase text-xs font-bold py-2">Name</div>
            <input class="text-white p-3 w-full bg-[--dark-600] rounded-md" type="text" placeholder="eg. This Course Sucks Don't Buy it" bind:value={course.title}>
        </label>
        <label>
            <div class="uppercase text-xs font-bold py-2">Tagline</div>
            <textarea on:click={resize} on:input={resize} on:keyup={resize} class="text-white p-3 w-full bg-[--dark-600] rounded-md" placeholder="eg. Get ready to build monstrously lavish and exciting worlds that players can't wait to explore!" bind:value={course.description}></textarea>
        </label>
        {#if course.title !== originalTitle || course.description !== originalDescription}
            <div class="flex">
                <button on:click={saveChanges} class="ml-auto rounded-md p-2 bg-red-500 hover:text-white text-gray-100">Save Changes</button>
            </div>
        {/if}
    </div>
</div>