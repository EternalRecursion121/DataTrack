<script>
    import { createEventDispatcher } from 'svelte';
    import Attribute from '$lib/Attribute.svelte';

    export let event = null;
    export let isOpen = false;

    let eventName = '';
    let attributes = [];
    let editingIndex = null;
    const dispatch = createEventDispatcher();

    // If an event is passed in, we're editing, otherwise, we're adding a new event.
    $: if (event) {
        editingIndex = event.index;
        eventName = event.name;
        attributes = [...event.attributes];
    } else {
        editingIndex = null;
        eventName = '';
        attributes = [];
    }

    function closeModal() {
        dispatch('close');
    }

    function saveEvent() {
        // Perform validation if necessary
        if (eventName.trim() === '') {
            alert('Please enter a name for the event.');
            return;
        }

        // Prepare the event data
        const eventData = {
            index: editingIndex,
            name: eventName,
            attributes
        };

        // Emit the save event with the event data
        dispatch('save', eventData);

        // Reset for next use
        eventName = '';
        attributes = [];
        editingIndex = null;
    }

    function addAttribute() {
        attributes = [...attributes, { name: '', value: '' }];
    }

    function deleteAttribute(index) {
        attributes.splice(index, 1);
        attributes = [...attributes];
    }

    function handleAttributeChange(event) {
        const { index, attribute } = event.detail;
        attributes[index] = attribute;
        attributes = [...attributes];
    }
</script>

{#if isOpen}
<div class="fixed inset-0 z-10 overflow-y-auto">
    <!-- Modal backdrop -->
    <div class="fixed inset-0 bg-black bg-opacity-50"></div>

    <!-- Modal content -->
    <div class="flex items-center justify-center min-h-screen">
        <div class="z-20 bg-gray-800 text-white rounded-lg shadow-lg max-w-md w-full m-4 p-6 min-w-[700px]">
            <h2 class="text-xl font-bold mb-4">
                {#if event} Edit Event {:else} Add Event {/if}
            </h2>

            <!-- Event name input -->
            <div class="mb-4">
                <label class="block text-sm font-bold mb-2" for="eventName">Event Name</label>
                <input id="eventName" class="w-full p-2 rounded bg-gray-700 focus:outline-none focus:shadow-outline" type="text" bind:value={eventName}>
            </div>

            <!-- Attribute inputs -->
            {#each attributes as attribute, index}
                <Attribute {attribute} {index} on:change={handleAttributeChange} onDelete={deleteAttribute} />
            {/each}

            <div class="mt-4 flex justify-between">
                <button class="flex-grow px-4 py-2 bg-blue-500 hover:bg-blue-600 rounded focus:outline-none" on:click={addAttribute}>
                    Add Attribute
                </button>
                <button class="flex-grow mx-4 px-4 py-2 bg-green-500 hover:bg-green-600 rounded focus:outline-none" on:click={saveEvent}>
                    Save Event
                </button>
                <button class="flex-grow px-4 py-2 bg-red-500 hover:bg-red-600 rounded focus:outline-none" on:click={closeModal}>
                    Close
                </button>
            </div>
        </div>
    </div>
</div>
{/if}
