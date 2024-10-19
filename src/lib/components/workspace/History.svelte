<script>
  import { onMount } from 'svelte';
  import { history, user } from '$lib/stores';
  import { getAllChatTags, getChatListByUserId, getMeaning, getAllChats } from '$lib/apis/chats';

  import nlp from 'compromise';

  let chats = [];
  let searchQuery = '';
  let searchResults = [];

// Define a list of programming languages
const programmingLanguages = [
  'javascript', 'python', 'java', 'c++', 'ruby', 'php', 'typescript', 'go', 'rust', 'swift', 'kotlin', 'c#'
];

// Define a dictionary of programming concepts
const programmingConcepts = {
  loops: ['loop', 'for', 'while', 'iteration'],
  functions: ['function', 'method', 'procedure', 'lambda'],
  classes: ['class', 'object', 'inheritance', 'constructor'],
  conditionals: ['if', 'else', 'switch', 'case', 'ternary'],
  variables: ['variable', 'var', 'const', 'let'],
  arrays: ['array', 'list', 'collection', 'map'],
};

  onMount(async () => {
    // Fetch initial chat history
    chats = await getChatListByUserId(localStorage.token, $user.id);
    history.set(chats);
  });

  const fetchAllChats = async () => {
    // Fetch chats by user ID using getAllChats
    chats = await getAllChats(localStorage.token);

    // Iterate through the messages and analyze the content
    chats.forEach(chat => {
      const messages = chat.chat.messages;

      messages.forEach(message => {
        const content = message.content;

        // Analyze the content using the compromise library
        const doc = nlp(content.toLowerCase());

        // Find the most used detected language
        const detectedLanguages = programmingLanguages.filter(lang => doc.has(lang));
        const mostUsedLanguage = detectedLanguages.length > 0 ? detectedLanguages[0] : null;

        // Find the most used detected concept
        let mostUsedConcept = null;
        let maxConceptCount = 0;

        for (const [concept, keywords] of Object.entries(programmingConcepts)) {
          const conceptCount = keywords.filter(keyword => doc.has(keyword)).length;

          if (conceptCount > maxConceptCount) {
            mostUsedConcept = concept;
            maxConceptCount = conceptCount;
          }
        }

        // Add the extracted information to the chat object
        chat.activity = mostUsedLanguage;
        chat.category = mostUsedConcept;
      });
    });

    console.log(chats);
    return chats;
  };

  const a = fetchAllChats()

  const handleSearch = async () => {
    if (searchQuery.trim() !== '') {
      // Call the API to fetch search results
      const response = await getMeaning(searchQuery);

      if (response && response.meaning) {
        const meaning = response.meaning;
        let results = '';

        if (meaning.Noun) {
          results += 'Meaning (Noun):\n';
          meaning.Noun.forEach((definition) => {
            results += `- ${definition}\n`;
          });
        }

        // Add other parts of the response to the results string if needed

        searchResults = results;
      } else {
        searchResults = 'No results found.';
      }
    }
  };
</script>

<svelte:head>
  <title>History</title>
</svelte:head>

<div class="px-4 py-2">
  <h1 class="text-xl font-semibold">Meaning Search</h1>

  <div class="flex items-center mt-2">
    <input
      type="text"
      bind:value={searchQuery}
      class="w-full px-2 py-1 border border-gray-300 rounded-l"
      placeholder="Search..."
    />
    <button
      on:click={handleSearch}
      class="px-2 py-1 border border-gray-300 rounded-r"
    >
    <svg
				xmlns="http://www.w3.org/2000/svg"
				viewBox="0 0 20 20"
				fill="currentColor"
				class="w-4 h-4"
			>
				<path
					fill-rule="evenodd"
					d="M9 3.5a5.5 5.5 0 100 11 5.5 5.5 0 000-11zM2 9a7 7 0 1112.452 4.391l3.328 3.329a.75.75 0 11-1.06 1.06l-3.329-3.328A7 7 0 012 9z"
					clip-rule="evenodd"
				/>
			</svg>
    </button>
  </div>

  {#if searchResults.length > 0}
      <p class="mt-2">{searchResults}</p>
  {/if}

  <br><hr class="my-2" /><br>
  <h1 class="text-xl font-semibold">Chat history</h1>

    <!-- <table class="w-full mt-2 border-collapse">
      <thead>
        <tr>
          <th class="px-2 py-1 border border-gray-300">Title</th>
          <th class="px-2 py-1 border border-gray-300">Date</th>
        </tr>
      </thead>
      <tbody>
        {#each $history as chat (chat.id)}
          <tr>
            <td class="px-2 py-1 border border-gray-300">
              <div class="truncate">{chat.title.slice(0, 60)}...</div>
            </td>
            <td class="px-2 py-1 border border-gray-300">{chat.time_range}</td>
          </tr>
        {/each}
      </tbody>
    </table> -->
    <table class="w-full mt-2 border-collapse">
      <thead>
        <tr>
          <th class="px-2 py-1 border border-gray-300">Title</th>
          <th class="px-2 py-1 border border-gray-300">Date</th>
          <th class="px-2 py-1 border border-gray-300">Activity</th>
          <th class="px-2 py-1 border border-gray-300">Category</th>
          <th class="px-2 py-1 border border-gray-300">Recommendations</th>
          <th class="px-2 py-1 border border-gray-300">Period</th>
        </tr>
      </thead>
      <tbody>
        {#each $history as chat (chat.id)}
          <tr>
            <td class="px-2 py-1 border border-gray-300">
              <div class="truncate">{chat.title.slice(0, 60)}...</div>
            </td>
            <td class="px-2 py-1 border border-gray-300">{chat.time_range}</td>
            <td class="px-2 py-1 border border-gray-300">{chat.activity}</td>
            <td class="px-2 py-1 border border-gray-300">{chat.category}</td>
            <td class="px-2 py-1 border border-gray-300">{chat.recommendations}</td>
            <td class="px-2 py-1 border border-gray-300">{chat.period}</td>
          </tr>
        {/each}
      </tbody>
    </table>
  
</div>