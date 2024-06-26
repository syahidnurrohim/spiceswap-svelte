<script>
	import { goto } from '$app/navigation';
	import { RECIPE_TYPE } from '@spiceswap/common/constant';
	import Image from '@spiceswap/components/Image.svelte';
	import { t } from '@spiceswap/locale/i18n';
	import { loadingStore } from '@spiceswap/stores/loadingStore';
	import { convertToTitleCase, generatePageTitleMeta } from '@spiceswap/utils/common';
	import { Avatar, Button } from 'flowbite-svelte';
	import {
		ClockOutline,
		CogOutline,
		EditOutline,
		EyeOutline,
		FileCopyOutline,
		LinkOutline
	} from 'flowbite-svelte-icons';
	import { onMount } from 'svelte';
	import { writable } from 'svelte/store';
	import Bookmark from './Bookmark.svelte';
	import { authStore } from '@spiceswap/stores/authStore';
	import { browser } from '$app/environment';
	import RecipeReview from './RecipeReview.svelte';
	import Alert from '../Alert.svelte';
  import Sticksy from 'sticksy'

	export let methods = {
		fetchCopyRecipe: async () => {}
	};
	export let recipe = {};

	const isOriginal = writable(true);
	const isAuthenticated = writable(false);
	const isOwner = writable(false);
	let stick = null;

	onMount(() => {
		isOriginal.set(recipe.recipeType === RECIPE_TYPE.ORIGINAL);
	});

	const _fetchCopyRecipe = async () =>
		await loadingStore.wrapFn(async () => await methods.fetchCopyRecipe(recipe.recipeSlug));

	$: isAuthenticated.set($authStore.isAuthenticated);
	$: isOwner.set(recipe.owner === $authStore.user.username);
	$: if (browser && stick) {
		const stickyEl = new window.Sticksy('.stick', {
			topSpacing: 85
		});
		stickyEl.onStateChanged = (state) => {
			stickyEl.nodeRef.classList.toggle('widget--sticky', state === 'fixed');
		};
	}
</script>

<svelte:head>
	{@html generatePageTitleMeta(t('pages.dashboard.recipe.detail.title'))}
</svelte:head>

<div class="px-16 mt-8">
	<h1 class="font-bold">{t('pages.dashboard.recipe.detail.title')}</h1>
</div>
{#if recipe.recipeSlug}
	<div class="px-16 py-8 mt-8 bg-white rounded-lg shadow overflow-hidden">
		<div class="flex gap-16">
			<div class="w-3/4">
				<div style="height: 500px;" class="-mt-8 -ms-16">
					<Image
						props={{
							src: recipe.thumbnailImageLink,
							alt: recipe.recipeName,
							class: 'w-full h-full object-center object-cover rounded-br-lg'
						}}
					/>
				</div>
				<div class="flex items-center gap-2 mt-8">
					<span class="text-title text-3xl">{convertToTitleCase(recipe.recipeName)}</span>
				</div>
				<span class="text-sm italic block mt-2"
					>{t('pages.dashboard.recipe.detail.portion', {
						portion: recipe.portion
					})}</span
				>
				<p class="mt-4 flex gap-2 items-center text-sm">
					<Avatar size="sm" class="h-4 w-4" />{recipe.owner}
				</p>
				<p class="mt-8">{recipe.about}</p>
				<div class="mt-8 flex justify-between">
					<span class="font-semibold text-sm flex gap-1 items-center">
						{t('pages.dashboard.recipe.detail.type')}: {recipe.recipeType}
						{#if !$isOriginal}
							<a href={`/browse/recipes/${recipe.copyFromSlug}`}>
								<LinkOutline size="md" class="cursor-pointer text-black" />
							</a>
						{/if}
					</span>
					{#if $isOriginal}
						{#if $isAuthenticated && $isOwner === false}
							<Button color="none" class="flex gap-2" on:click={_fetchCopyRecipe}>
								<FileCopyOutline />
								{t('pages.dashboard.recipe.detail.copy')}
							</Button>
						{/if}
					{:else}
						<span class="text-sm font-medium flex gap-1 items-center">
							<EyeOutline size="sm" />
							{t('pages.dashboard.recipe.detail.visibility', {
								visibility: convertToTitleCase(recipe.visibility)
							})}</span
						>
					{/if}
				</div>
				<div class="flex gap-8 mt-8">
					<div class="w-1/2">
						<h3 class="text-2xl font-bold mb-2">
							{t('pages.dashboard.recipe.detail.ingredients')}
						</h3>
						<ul>
							{#each recipe.ingredientDetailResponses as ingredient}
								<li class="text-sm my-1">- {ingredient.dose} {ingredient.ingredientName}</li>
							{/each}
						</ul>
					</div>
					<div class="w-full">
						<h3 class="text-2xl font-bold mb-2">
							{t('pages.dashboard.recipe.detail.instructions')}
						</h3>
						<ul>
							{#each recipe.stepDetailResponses as instruction}
								<li class="text-sm my-1">
									<span class="font-semibold">
										{t('pages.dashboard.recipe.detail.step', {
											step: instruction.numberStep
										})},
									</span>
									{instruction.stepDescription}
								</li>
							{/each}
						</ul>
					</div>
				</div>
			</div>
			<div class="w-1/4">
				<div class=" flex flex-col gap-2 stick" bind:this={stick}>
					{#if $isAuthenticated && !$isOwner}
						<Bookmark
							{recipe}
							buttonClass="flex items-center gap-4 justify-start px-4 py-2 text-sm text-yellow-400 border border-yellow-400 dark:border-yellow-300 dark:text-yellow-300 rounded-lg"
						>
							{t('pages.dashboard.recipe.detail.bookmark')}
						</Bookmark>
					{/if}
					<Button class="flex gap-4 justify-start" color="yellow"
          on:click={() => goto(`/browse/recipes/${recipe.recipeSlug}/history`)}
						><ClockOutline />{t('pages.dashboard.recipe.detail.changes-history')}</Button
					>
					{#if $isAuthenticated && $isOwner}
						<Button
							class="flex gap-4 justify-start"
							on:click={() => goto(`/dashboard/recipe/update/${recipe.recipeSlug}`)}
							><EditOutline /> {t('pages.dashboard.recipe.detail.edit')}</Button
						>
					{/if}
					{#if $isAuthenticated && $isOwner}
						<Button
							class="flex gap-4 justify-start"
							color="alternative"
							on:click={() => goto(`/dashboard/recipe/settings/${recipe.recipeSlug}`)}
							><CogOutline />{t('pages.dashboard.recipe.detail.settings')}</Button
						>
					{/if}
				</div>
			</div>
		</div>
		<hr class="my-8" />
		<RecipeReview {recipe} />
	</div>
{:else}
	<Alert dismissed={false} dismissible={false} type="warning" alertClass="mt-8 flex justify-center">
		{t('pages.dashboard.recipe.detail.not-found')}
	</Alert>
{/if}
