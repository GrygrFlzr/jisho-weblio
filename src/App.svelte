<script>
	const BASE_CORS = "https://cors-anywhere.herokuapp.com/";
	const BASE_WEBLIO = "https://ejje.weblio.jp/content/";
	const BASE_JISHO = "https://jisho.org/api/v1/search/words";

	let search = "";
	let lastTimeout;
	let controller;
	let status = "initial";
	let data;
	let selection;

	const triggerSearch = () => {
		if (search === "") {
			status = "initial";
			return;
		}
		if (controller) {
			controller.abort();
		}
		clearTimeout(lastTimeout);
		controller = new AbortController();
		lastTimeout = setTimeout(async () => {
			status = "loading";
			const url =
				`${BASE_CORS}${BASE_JISHO}?` +
				new URLSearchParams({
					keyword: search,
				});
			let res = await fetch(url, { signal: controller.signal });
			data = await res.json();
			if (data.data.length > 0) {
				selection = data.data[0].slug;
				status = "ready";
			} else {
				status = "empty";
				selection = undefined;
			}
		}, 300);
	};
</script>

<main>
	{#if status === 'initial'}
		<p>Enter a search term:</p>
	{:else if status === 'loading'}
		<p>Loading...</p>
	{:else if status === 'ready'}
		<p>Search result for:</p>
	{:else if status === 'empty'}
		<p>No results found on Jisho.</p>
	{/if}

	<label>
		<input
			type="textbox"
			placeholder="search weblio with romaji"
			bind:value={search}
			on:input={triggerSearch} />
	</label>

	<section>
		{#if data}
			{#each data.data as { slug } (slug)}
				<button
					class:selected={selection === slug}
					on:click={() => {
						selection = slug;
					}}>
					{slug}
				</button>
			{/each}
		{/if}
	</section>

	{#if selection}
		<iframe title={selection} src="{BASE_WEBLIO}{selection}" />
	{/if}
</main>


<style>
	main {
		width: 100%;
		height: 100%;
		display: grid;
		grid-template-rows: 2rem 2.5rem 3rem 1fr;
	}
	iframe {
		width: 100%;
		height: 100%;
		margin: 0;
		padding: 0;
		border: 2px solid #ea9034;
	}
	label {
		margin: 0 auto;
		width: 100%;
		max-width: 960px;
		position: relative;
	}
	label:before {
		content: "";
		position: absolute;
		left: 10px;
		top: 0;
		bottom: 0;
		width: 1rem;
		background: url("data:image/svg+xml,<svg xmlns=\"http://www.w3.org/2000/svg\" fill=\"none\" viewBox=\"0 0 24 24\" stroke=\"currentColor\"><path stroke-linecap=\"round\" stroke-linejoin=\"round\" stroke-width=\"2\" d=\"M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z\" /></svg>") center / contain no-repeat;
	}
	label input {
		width: 100%;
		padding-left: 2rem;
	}
	p {
		margin: 0.25rem auto 0;
		width: 100%;
		max-width: 960px;
	}
	section {
		display: grid;
		grid-auto-flow: column;
		gap: 2px;
	}
	section button {
		margin-bottom: 2px;
	}
	.selected {
		border: #ea9034;
		background: #ea9034;
		border-radius: 0.25rem 0.25rem 0 0;
		margin-bottom: 0;
		color: #fff;
		font-weight: 600;
	}
</style>