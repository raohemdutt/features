<script lang="ts">
	import type { SupabaseClient } from "@supabase/supabase-js";

    export let supabase:SupabaseClient
    export let sectionId:string;
    export let courseId:string;
    let file;

    async function uploadImage() {
        if (!file) return;

        supabase.storage.from("courses").list(`${courseId}/${sectionId}`)
    }
</script>

<div class="flex gap-2">
    <div>
        <button class="mx-auto text-gray-700 bg-gray-300 opacity-70 hover:opacity-100 hover:text-black p-2 rounded">
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16" fill="currentColor" class="w-4 h-4">
                <path fill-rule="evenodd" d="M2 4a2 2 0 0 1 2-2h8a2 2 0 0 1 2 2v8a2 2 0 0 1-2 2H4a2 2 0 0 1-2-2V4Zm10.5 5.707a.5.5 0 0 0-.146-.353l-1-1a.5.5 0 0 0-.708 0L9.354 9.646a.5.5 0 0 1-.708 0L6.354 7.354a.5.5 0 0 0-.708 0l-2 2a.5.5 0 0 0-.146.353V12a.5.5 0 0 0 .5.5h8a.5.5 0 0 0 .5-.5V9.707ZM12 5a1 1 0 1 1-2 0 1 1 0 0 1 2 0Z" clip-rule="evenodd" />
            </svg>
        </button>
    </div>
    <label>
        Add Image(s)
        <input type="file" bind:value={file} class="my-auto">
    </label>
</div>