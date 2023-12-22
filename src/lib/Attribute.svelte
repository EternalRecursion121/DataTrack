<script>
    export let attribute, index;
    export let onDelete; // Passed in from the parent to delete the attribute

    import { createEventDispatcher } from 'svelte';

    const dispatch = createEventDispatcher();

    function emitChange() {
        dispatch('change', { attribute, index });
    }

    function emitDelete() {
        onDelete(index);
    }
</script>

<div class="flex items-center gap-2 mb-2">
    <input
        class="flex-grow p-2 bg-gray-800 border border-gray-600 rounded-md"
        type="text"
        placeholder="Attribute Name"
        bind:value={attribute.name}
        on:input={emitChange}
    />
    <input
        class="flex-grow p-2 bg-gray-800 border border-gray-600 rounded-md"
        type="text"
        placeholder="Attribute Value"
        bind:value={attribute.value}
        on:input={emitChange}
    />
    <button class="p-2 bg-red-500 rounded-full hover:bg-red-600 focus:outline-none focus:ring-2 focus:ring-red-500 transition duration-200" on:click={emitDelete}>
        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-4 h-4">
            <path stroke-linecap="round" stroke-linejoin="round" d="M6 18 18 6M6 6l12 12" />
        </svg>
    </button>
</div>
