<script>
    import { createEventDispatcher } from 'svelte';

    export let events;
    export let event = null;
    export let isOpen = false;

    let eventName = '';
    let attributes = [];
    let editingIndex = null;
    const dispatch = createEventDispatcher();

    let showSuggestions = false;

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
      console.log(eventName)
      if (eventName.trim() === '') {
          alert('Please enter a name for the event.');
          return;
      }

      const creationTime = new Date();

      // Prepare the event data
      const eventData = {
          index: editingIndex,
          event: {
            name: eventName,
            attributes,
            creationTime: creationTime
          }
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


    let suggestedTitles = [];
    // Function to suggest event titles based on the input value
    function suggestEventTitles(inputValue) {
        if (inputValue.length > 0) {
            suggestedTitles = events
                .map(event => event.name)
                .filter((name, index, self) => self.indexOf(name) === index)
                .filter(name => name.toLowerCase().startsWith(inputValue.toLowerCase()))
                .slice(0, 5); // Limit the number of suggestions
        } else {
            suggestedTitles = []; // Clear suggestions if the input is empty
        }
    }

    function onEventNameInput(event) {
        eventName = event.target.value;
        suggestEventTitles(eventName);
        showSuggestions = true;
    }


    function useSuggestedTitle(title) {
        eventName = title;
        const lastEventWithSelectedTitle = [...events].reverse().find(event => event.name === title);
        attributes = lastEventWithSelectedTitle ? lastEventWithSelectedTitle.attributes.map(attr => ({ ...attr })) : [];
        showSuggestions = false; // Hide suggestions after selection
    }

    function handleBlur() {
        // Use a timeout to delay hiding suggestions
        setTimeout(() => {
            showSuggestions = false;
        }, 5); // Delay in milliseconds
    }

</script>

{#if isOpen}
<div class="fixed inset-0 z-10 overflow-y-auto">
  <!-- Modal backdrop -->
  <div class="fixed inset-0 bg-black bg-opacity-50"></div>

  <!-- Modal content -->
  <div class="flex items-center justify-center min-h-screen">
    <div class="z-50 bg-gray-800 text-white rounded-lg shadow-lg w-full max-w-lg m-4 p-6 space-y-4">
      <h2 class="text-xl font-bold mb-4">
        {#if event} Edit Event {:else} Add Event {/if}
      </h2>

      <!-- Event name input -->
      <div class="relative">
          <input
              type="text"
              bind:value={eventName}
              on:input={onEventNameInput}
              on:blur={() => handleBlur()}
              on:focus={() => showSuggestions = true}
              class="w-full p-2 rounded bg-gray-700 focus:outline-none focus:shadow-outline"
              placeholder="Event Name"
          />
          {#if suggestedTitles.length > 0 && showSuggestions}
            <div class="absolute left-0 right-0 bg-gray-700 mt-1">
                {#each suggestedTitles as title}
                    <div
                        class="p-2 hover:bg-gray-600 cursor-pointer"
                        on:click={() => useSuggestedTitle(title)}
                    >
                        {title}
                    </div>
                {/each}
            </div>
            {/if}
        </div>

      <!-- Attribute inputs -->
      {#each attributes as attribute, index}
        <div class="flex flex-col sm:flex-row gap-2 mb-2">
          <input
            type="text"
            placeholder="Attribute Name"
            bind:value={attribute.name}
            class="flex-grow p-2 rounded bg-gray-700 focus:outline-none focus:shadow-outline"
          />
          <input
            type="text"
            placeholder="Attribute Value"
            bind:value={attribute.value}
            class="flex-grow p-2 rounded bg-gray-700 focus:outline-none focus:shadow-outline"
          />
          <button
            class="bg-red-500 hover:bg-red-600 rounded-full p-2 focus:outline-none focus:shadow-outline"
            on:click={() => deleteAttribute(index)}
          >
            <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor" class="w-6 h-6">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
            </svg>
          </button>
        </div>
      {/each}

      <!-- Action buttons -->
      <div class="flex flex-wrap justify-between gap-2">
        <button
          class="px-4 py-2 bg-blue-500 hover:bg-blue-600 rounded focus:outline-none w-full sm:w-auto sm:flex-grow"
          on:click={addAttribute}
        >
          Add Attribute
        </button>
        <button
          class="px-4 py-2 bg-green-500 hover:bg-green-600 rounded focus:outline-none w-full sm:w-auto sm:flex-grow"
          on:click={saveEvent}
        >
          Save Event
        </button>
        <button
          class="px-4 py-2 bg-red-500 hover:bg-red-600 rounded focus:outline-none w-full sm:w-auto sm:flex-grow"
          on:click={closeModal}
        >
          Close
        </button>
      </div>
    </div>
  </div>
</div>
{/if}
