<script>
	import { browseRecipesPaginate } from '@spiceswap/api/recipe';
	import { t } from '@spiceswap/locale/i18n';
	import { loadingStore } from '@spiceswap/stores/loadingStore';
	import { generatePageTitleMeta, showToast } from '@spiceswap/utils/common';
	import { generateMessageFromResponse } from '@spiceswap/utils/fetch';
	import { writable } from 'svelte/store';
	import _ from 'lodash';
	import { page } from '$app/stores';
	import RecipeGrid from '@spiceswap/components/Recipe/RecipeGrid.svelte';
	import RecipePagination from '@spiceswap/components/Recipe/RecipePagination.svelte';
	import { Label, Spinner } from 'flowbite-svelte';

	const recipes = writable([]);
	const pageNumber = writable(0);
	const sortBy = writable('newest');
	const totalPages = writable(0);
	const loadingRecepies = writable(true);

	const fetchRecepies = async (pageNumber, keyword, sortBy) => {
		const response = await browseRecipesPaginate(pageNumber, keyword, sortBy);
		const data = await response.json();
		if (response.ok) {
			recipes.set(data.results.content);
			totalPages.set(data.results.totalPages);
		} else {
			showToast(
				t('pages.dashboard.recipe.browse-recipes.error'),
				generateMessageFromResponse(data),
				'error'
			);
			recipes.set([]);
			totalPages.set(0);
		}
		loadingRecepies.set(false);
	};

	async function browseRecipes(pageNumber, keyword, sortBy) {
		await loadingStore.wrapFn(async () => fetchRecepies(pageNumber, keyword, sortBy));
	}

	const debouncedBrowseRecipes = _.debounce((pageNumber, keyword, sortBy) => {
		browseRecipes(pageNumber, keyword, sortBy);
	}, 300);

	function handlePageChange(_pageNumber) {
		pageNumber.set(_pageNumber - 1);
	}

	function detailLink(recipe) {
		return `/browse/recipes/${recipe.recipeSlug}`;
	}

	$: {
		const urlParams = new URLSearchParams($page.url.search);
		const keyword = urlParams.get('keyword') ? urlParams.get('keyword') : '';
		debouncedBrowseRecipes($pageNumber, keyword, $sortBy);
	}
</script>

<svelte:head>
	{@html generatePageTitleMeta(t('pages.dashboard.recipe.browse-recipes.title'))}
</svelte:head>

<div class="my-8 px-16">
	<h1 class="font-bold">{t('pages.dashboard.recipe.browse-recipes.title')}</h1>
	{#if !$loadingRecepies}
		<form class="flex flex-col gap-2 mt-8 w-1/4">
			<Label class="font-light">{t('pages.dashboard.recipe.browse-recipes.sort-by')}</Label>
			<select
				id="sort-by"
				bind:value={$sortBy}
				class="cursor-pointer bg-gray-50 border-0.25 border-gray-300 font-light text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500"
			>
				<option value="newest">{t('pages.dashboard.recipe.browse-recipes.newest')}</option>
				<option value="oldest">{t('pages.dashboard.recipe.browse-recipes.oldest')}</option>
				<option value="popular">{t('pages.dashboard.recipe.browse-recipes.popular')}</option>
			</select>
		</form>
		<RecipeGrid recipes={$recipes} {detailLink} />
		<div class="flex justify-center mt-8">
			<RecipePagination
				number={$pageNumber + 1}
				totalPages={$totalPages}
				onPageChange={handlePageChange}
			/>
		</div>
	{:else}
		<div class="flex justify-center items-center mt-8">
			<Spinner />
		</div>
	{/if}
</div>
