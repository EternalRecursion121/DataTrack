<script>
    import { onMount } from 'svelte';
    import Attribute from '$lib/Attribute.svelte';
    import EventModal from '$lib/EventModal.svelte';

    let events = [];
    let attributes = [];
    let eventName = '';
    let currentAttribute = { name: '', value: '' };
    let editingIndex = null; // Index of the event being edited or null if adding

    onMount(() => {
        loadEvents();
    });

    $: if (attributes.length > 0) {
        saveAttributes();
    }

    let isModalOpen = false;
    let currentEvent = null;

    function openModal(event = null) {
        currentEvent = event;
        isModalOpen = true;
    }

    function handleModalClose() {
        isModalOpen = false;
        currentEvent = null;
    }

    function handleSaveEvent(eventData) {
        // Use eventData.index to determine if you're adding a new event or updating an existing one
        // Perform the save operation here
        handleModalClose();
    }

    function loadEvents() {
        const storedEvents = localStorage.getItem('events');
        if (storedEvents) {
            events = JSON.parse(storedEvents);
        }
    }

    function saveEvent() {
        if (eventName.trim() === '') {
            alert('Please enter a name for the event.');
            return;
        }
        if (attributes.some(attr => attr.name.trim() === '' || attr.value.trim() === '')) {
            alert('Please fill in all attribute names and values before saving an event.');
            return;
        }
        
        const eventToSave = {
            name: eventName,
            time: editingIndex === null ? new Date().toISOString() : events[editingIndex].time, // Preserve the original time if editing
            attributes: [...attributes]
        };

        if (editingIndex !== null) {
            // Immutable update of the events array
            events = [
                ...events.slice(0, editingIndex),
                eventToSave,
                ...events.slice(editingIndex + 1)
            ];
        } else {
            events = [...events, eventToSave];
        }
        
        saveEvents();
        eventName = ''; // Clear the event name input
        attributes = []; // Clear attributes for the next event
        editingIndex = null; // Reset editing index
    }

    function addAttribute() {
        attributes = [...attributes, { ...currentAttribute }];
        currentAttribute = { name: '', value: '' };
    }

    function saveAttributes() {
        localStorage.setItem('attributes', JSON.stringify(attributes));
    }

    function saveEvents() {
        localStorage.setItem('events', JSON.stringify(events));
    }

    function handleAttributeChange(event) {
        const { index, attribute } = event.detail;
        attributes[index] = attribute;
        attributes = attributes.slice();
    }

    function deleteEvent(index) {
        events.splice(index, 1);
        events = events.slice();
        saveEvents();
    }

    function editEvent(index) {
        editingIndex = index;
        const eventToEdit = events[index];
        eventName = eventToEdit.name;
        attributes = [...eventToEdit.attributes];
    }

    function deleteAttribute(index) {
        attributes.splice(index, 1);
        attributes = attributes.slice(); // Ensure reactivity by creating a new array
        saveAttributes(); // You might want to save the attributes after deletion
    }
</script>

<div class="min-h-screen bg-gray-900 text-white">
    <div class="container mx-auto p-4 space-y-8">
        <div class="space-y-4">
            <div class="text-xl font-semibold">Event Name:</div>
            <input type="text" bind:value={eventName} class="w-full px-4 py-2 bg-gray-800 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"/>
        </div>
        <button
            class="fixed bottom-4 right-4 flex items-center justify-center p-3 bg-blue-500 text-white rounded-full shadow hover:bg-blue-600 focus:outline-none focus:ring focus:ring-blue-300 transition duration-200"
            on:click={()=>isModalOpen = true}
        >
            <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4" />
            </svg>
            <span class="ml-2">Create Event</span>
        </button>
        <div class="flex space-x-4">
            <button class="px-6 py-2 text-sm font-semibold bg-blue-500 rounded-lg hover:bg-blue-600 focus:outline-none focus:ring-2 focus:ring-blue-500 transition duration-200" on:click={addAttribute}>
                Add Attribute
            </button>
            <button class="px-6 py-2 text-sm font-semibold bg-green-500 rounded-lg hover:bg-green-600 focus:outline-none focus:ring-2 focus:ring-green-500 transition duration-200" on:click={saveEvent}>
                Save Event
            </button>
        </div>
        <EventModal isOpen={isModalOpen} event={currentEvent} on:close={handleModalClose} on:save={handleSaveEvent} />
        {#each events as event, index (event.time)}
            <div class="p-4 bg-gray-800 rounded-lg shadow-lg space-y-4" key={event.time}>
                <div class="flex justify-between items-center">
                    <div class="text-lg font-semibold">{event.name}</div>
                    <div class="flex space-x-2">
                        <button class="p-2 bg-purple-700 rounded-full hover:bg-purple-800 focus:outline-none focus:ring-2 focus:ring-purple-800 transition duration-200" on:click={() => editEvent(index)}>
                            <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-4 h-4">
                                <path stroke-linecap="round" stroke-linejoin="round" d="m16.862 4.487 1.687-1.688a1.875 1.875 0 1 1 2.652 2.652L6.832 19.82a4.5 4.5 0 0 1-1.897 1.13l-2.685.8.8-2.685a4.5 4.5 0 0 1 1.13-1.897L16.863 4.487Zm0 0L19.5 7.125" />
                            </svg>                     
                        </button>
                        <button class="p-2 bg-red-500 rounded-full hover:bg-red-600 focus:outline-none focus:ring-2 focus:ring-red-500 transition duration-200" on:click={() => deleteEvent(index)}>
                            <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-4 h-4">
                                <path stroke-linecap="round" stroke-linejoin="round" d="M6 18 18 6M6 6l12 12" />
                            </svg>     
                        </button>
                    </div>
                </div>
                <div class="text-sm text-gray-400">{new Date(event.time).toLocaleString()}</div>
                <div class="space-y-2">
                    {#each event.attributes as attr, attrIndex}
                        <div class="text-gray-300">{attr.name}: {attr.value}</div>
                    {/each}
                </div>
            </div>
        {/each}
    </div>
</div>
