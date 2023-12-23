<script>
    import { onMount } from 'svelte';
    import EventModal from '$lib/EventModal.svelte';

    let events = [];
    let attributes = [];
    let eventName = '';
    let currentAttribute = { name: '', value: '' };
    let editingIndex = null; // Index of the event being edited or null if adding

    onMount(() => {
        loadEvents();
    });

    $: console.log(events)

    $: if (attributes.length > 0) {
        saveAttributes();
    }

    let isModalOpen = false;
    let currentEvent = null;

    function handleModalClose() {
        isModalOpen = false;
        currentEvent = null;
    }

    function handleSaveEvent(e) {
        let eventData = e.detail;
        if (eventData.index !== null) {
            // Immutable update of the events array
            events = [
                ...events.slice(0, eventData.index),
                eventData.event,
                ...events.slice(eventData.index + 1)
            ];
            console.log(eventData)
        } else {
            events = [...events, eventData.event];
        }

        saveEvents();
        handleModalClose();
    }

    function loadEvents() {
        const storedEvents = localStorage.getItem('events');
        if (storedEvents) {
            events = JSON.parse(storedEvents);
        }
    }

    function saveAttributes() {
        localStorage.setItem('attributes', JSON.stringify(attributes));
    }

    function saveEvents() {
        localStorage.setItem('events', JSON.stringify(events));
    }

    function deleteEvent(index) {
        events.splice(index, 1);
        events = events.slice();
        saveEvents();
    }

    function editEvent(index) {
        editingIndex = index;
        currentEvent = events[index];
        isModalOpen = true;
    }

    function exportToJSON() {
        const jsonString = JSON.stringify(events, null, 2);
        const blob = new Blob([jsonString], { type: 'application/json' });
        const href = URL.createObjectURL(blob);
        const link = document.createElement('a');
        link.href = href;
        link.download = 'events.json';
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
        URL.revokeObjectURL(href);
    }
</script>

<div class="min-h-screen bg-gray-900 text-white">
    <div class="container mx-auto p-4 space-y-8">
        <button
            class="fixed bottom-4 right-4 flex items-center justify-center p-3 bg-blue-500 text-white rounded-full shadow hover:bg-blue-600 focus:outline-none focus:ring focus:ring-blue-300 transition duration-200"
            on:click={()=>isModalOpen = true}
        >
            <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4" />
            </svg>
            <span class="ml-2">Add Event</span>
        </button>
        <button
            class="px-4 py-2 bg-green-500 rounded hover:bg-green-600 transition-colors text-white"
            on:click={exportToJSON}
        >
            Export to JSON
        </button>
        <EventModal {events} isOpen={isModalOpen} event={currentEvent} on:close={handleModalClose} on:save={handleSaveEvent} />
        {#each events as event, index (event.creationTime)}
            <div class="p-4 bg-gray-800 rounded-lg shadow-lg space-y-4" key={event.creationTime}>
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
                <div class="text-sm text-gray-400">{new Date(event.creationTime).toLocaleString()}</div>
                <div class="space-y-2">
                    {#each event.attributes as attr, attrIndex}
                        <div class="text-gray-300">{attr.name}: {attr.value}</div>
                    {/each}
                </div>
            </div>
        {/each}
    </div>
</div>
