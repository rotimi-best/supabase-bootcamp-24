<script>
  import { onMount } from 'svelte';
  import { supabase } from '$lib/supabase';

  /**
   * @type {any[]}
   */
  let lessons = [];

  let isLoading = false;

  async function getLessons() {
    isLoading = true;
    const { data, error } = await supabase
      .from('lesson')
      .select(
        `
      id,
      title,
      author,
      created_at,
      slug
    `
      )
      .order('id', { ascending: true });

    if (error) {
      isLoading = false;
      lessons = [];
      console.error('Error fetching lessons', error);

      // alert('Something went wrong');
      return;
    }

    lessons = data.map((l) => ({
      ...l,
      createdAt: l.created_at,
    }));
    isLoading = false;

    console.log({ lessons });
  }

  /**
   * @param {any} id
   */
  async function deleteLesson(id) {
    const { error } = await supabase.from('lesson').delete().eq('id', id);

    if (error) {
      console.error('Error deleting lesson', error);
      return;
    }

    lessons = lessons.filter((lesson) => lesson.id !== id);
  }

  onMount(() => {
    getLessons();
  });
</script>

<section class="w-[90vw] flex flex-col items-center my-10">
  <div class="flex justify-end w-full mb-3">
    <button
      class="bg-gray-600 text-white px-5 py-2 rounded-md"
      on:click={() => getLessons()}
      disabled={isLoading}
    >
      Refresh
    </button>
  </div>
  {#if isLoading}
    Loading...
  {:else}
    {#each lessons as lesson}
      <div class="bg-gray-100 rounded-md mb-5 px-10 py-5 w-[500px]">
        <h3 class="text-xl mb-0">
          <a href="/lessons/{lesson.slug}">{lesson.id}. - {lesson.title}</a>
        </h3>
        <p class="text-sm mb-0">By {lesson.author}</p>
        <div class="flex items-center justify-between">
          <p class="text-xs">{lesson.createdAt}</p>

          <button
            class="bg-red-600 text-white px-5 py-2 rounded-md"
            on:click={() => deleteLesson(lesson.id)}
          >
            Delete
          </button>
        </div>
      </div>
    {/each}
  {/if}
</section>