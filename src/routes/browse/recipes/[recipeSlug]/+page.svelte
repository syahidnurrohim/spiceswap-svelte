<script>
	import { page } from '$app/stores';
	import { copyRecipe, getBrowseRecipeDetail } from '@spiceswap/api/recipe';
	import RecipeDetail from '@spiceswap/components/Recipe/RecipeDetail.svelte';
	import { t } from '@spiceswap/locale/i18n';
	import { loadingStore } from '@spiceswap/stores/loadingStore';
	import { generatePageTitleMeta, showToast } from '@spiceswap/utils/common';
	import { generateMessageFromResponse } from '@spiceswap/utils/fetch';
	import { Spinner } from 'flowbite-svelte';
	import { onMount } from 'svelte';
	import { writable } from 'svelte/store';

	const recipeSlug = $page.params.recipeSlug;
	const recipe = writable({});
	const loadingRecipe = writable(true);

	onMount(async () => {
		await loadingStore.wrapFn(async () => await fetchRecipeDetail());
	});

	const fetchRecipeDetail = async () => {
		const response = await getBrowseRecipeDetail(recipeSlug);
		const data = await response.json();
		if (response.ok) {
			recipe.set(data.results);
		} else {
			showToast(
				t('pages.dashboard.recipe.detail.error'),
				generateMessageFromResponse(data),
				'error'
			);
		}
		loadingRecipe.set(false);
	};

	const fetchCopyRecipe = async (recipeSlug) => {
		const response = await copyRecipe(recipeSlug);
		const data = await response.json();
		if (response.ok) {
			showToast(
				t('pages.dashboard.recipe.detail.copy-success'),
				generateMessageFromResponse(data),
				'success'
			);
		} else {
			showToast(
				t('pages.dashboard.recipe.detail.copy-error'),
				generateMessageFromResponse(data),
				'error'
			);
		}
	};
</script>

<svelte:head>
	{@html generatePageTitleMeta(t('pages.dashboard.recipe.detail.title'))}
</svelte:head>

{#if !$loadingRecipe}
	<RecipeDetail recipe={$recipe} methods={{ fetchCopyRecipe }} />
{:else}
	<div class="w-full h-full flex justify-center items-center mt-16">
		<Spinner />
	</div>
{/if}
