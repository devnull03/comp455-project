<script lang="ts">
	import type { PageData, ActionData } from './$types';
	import { fly, slide, scale, fade } from 'svelte/transition';
	import { onMount } from 'svelte';

	import { Button } from '$lib/components/ui/button';
	import { Input } from '$lib/components/ui/input';
	import { toast } from 'svelte-sonner';
	import * as Table from '$lib/components/ui/table';
	import * as Pagination from '$lib/components/ui/pagination';
	import type { SearchResultsRes } from '$lib/model';
	import { currentQuery, currentSearchResults } from '$lib/stores.svelte';
	import { getPage } from '$lib/api';
	import { goto } from '$app/navigation';

	let inputFocus: boolean = $state(false);
	let buttonHover: boolean = $state(false);
	let loadDelay: boolean = $state(true);

	let query: string = $state('');

	const handlePagination = async (page: number) => {
		loadDelay = true;
		let result = await getPage($currentSearchResults?.search_id as string, page);

		if (result.error) {
			toast.error(result.message || '');
		} else {
			$currentSearchResults = result;
		}
		loadDelay = false;
	};

	let { data, form }: { data: PageData; form: ActionData } = $props();
	onMount(() => {
		console.log(form);

		if (form?.error) {
			toast.error(form?.message || '');
		}

		if (form?.queryText) $currentQuery = form?.queryText as string;

		query = $currentQuery;

		if (form?.result) $currentSearchResults = form?.result;

		setTimeout(() => {
			loadDelay = false;
		}, 50);
	});
</script>

<main class="flex flex-col items-center justify-center h-screen">
	<section class="flex flex-col gap-8 sticky top-12 items-center">
		<h1 class="text-3xl font-light self-center">Search Engine</h1>
		<form
			method="POST"
			class="group flex gap-1 transition-all duration-300 ease-in-out"
			onsubmit={(e) => {
				if (query === '') {
					toast.error('Invalid search');
					e.preventDefault();
				}
			}}
		>
			<Input
				type="text"
				placeholder="what do you wanna search?"
				class="rounded-lg outline-none transition-all ease-in-out duration-300 {inputFocus
					? 'w-[70vw]'
					: 'w-[45vw]'}"
				onfocus={() => {
					inputFocus = true;
				}}
				onblur={() => {
					inputFocus = buttonHover;
				}}
				name="query"
				bind:value={query}
			/>
			<Button
				onmouseover={() => {
					buttonHover = true;
				}}
				onmouseleave={() => {
					buttonHover = false;
				}}
				type="submit">Search</Button
			>
		</form>
		{#if !$currentSearchResults}
			<p class="text-sm text-gray-500 italic w-[50vw] text-center">
				We have a comprehensive database sourced from Kaggle, containing over 8,000 articles from
				various news outlets. Each article is meticulously labeled to indicate whether it is a
				genuine piece or contains false information. This allows users to search and identify the
				authenticity of news articles effectively.
			</p>
		{/if}
		<div></div>
	</section>

	{#if !loadDelay && $currentSearchResults}
		<section transition:slide class="h-auto flex flex-col overflow-auto max-h-[90vh] mt-16 px-16">
			<Table.Root class="w-[99%] mb-16">
				<Table.Caption>Search Results for "{$currentQuery}"</Table.Caption>
				<Table.Header>
					<Table.Row>
						<Table.Head class="w-[100px]">Document ID</Table.Head>
						<Table.Head class="w-[35%]">Title</Table.Head>
						<Table.Head>Content</Table.Head>
						<Table.Head class="text-right">Real</Table.Head>
					</Table.Row>
				</Table.Header>
				<Table.Body>
					{#each $currentSearchResults.data as item (item.data.id)}
						<Table.Row
							class="cursor-pointer"
							onclick={() => {
								goto(`/record/${item.data.id}`);
							}}
						>
							<Table.Cell class="font-medium">{item.data.id}</Table.Cell>
							<Table.Cell>{item.data.title}</Table.Cell>
							<Table.Cell
								>{item.data.text.slice(0, Math.min(300, item.data.text.length))}...
								<i class="font-light">click to read more</i>
							</Table.Cell>
							<Table.Cell class="text-right">{item.data.label}</Table.Cell>
						</Table.Row>
					{/each}
				</Table.Body>
			</Table.Root>

			<Pagination.Root
				count={$currentSearchResults.number_of_results}
				perPage={20}
				let:pages
				let:currentPage
				class="my-4"
				onPageChange={handlePagination}
			>
				<Pagination.Content>
					<Pagination.Item>
						<Pagination.PrevButton />
					</Pagination.Item>
					{#each pages as page (page.key)}
						{#if page.type === 'ellipsis'}
							<Pagination.Item>
								<Pagination.Ellipsis />
							</Pagination.Item>
						{:else}
							<Pagination.Item>
								<Pagination.Link {page} isActive={currentPage == page.value}>
									{page.value}
								</Pagination.Link>
							</Pagination.Item>
						{/if}
					{/each}
					<Pagination.Item>
						<Pagination.NextButton />
					</Pagination.Item>
				</Pagination.Content>
			</Pagination.Root>
		</section>
	{/if}
</main>
