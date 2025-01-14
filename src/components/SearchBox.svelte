<script lang="ts">
	// SPDX-FileCopyrightText: Magnus Anderssen <magnus@magooweb.com>
	// SPDX-License-Identifier: AGPL-3.0-or-later

	import { createEventDispatcher, onMount } from 'svelte';
	import { get } from 'svelte/store';
	import { PROVIDER_ALL, PROVIDER_ALL_LABEL } from '../lib/search';
	import { _t } from '../lib/translate';
	import providers from '../states/providers';
	import { providerId, terms } from '../states/query';

	let userQuery = get(terms);
	let userProviderId = get(providerId);

	const dispatch = createEventDispatcher();
	let input: HTMLInputElement;

	onMount(() => {
		if (get(terms) !== '' && get(providerId)) {
			search();
		} else {
			input.focus();
		}
	});

	function clear() {
		terms.set('');
		providerId.set(PROVIDER_ALL);
		userQuery = '';
		userProviderId = PROVIDER_ALL;
		dispatch('clear');
		input.focus();
	}

	function search() {
		providerId.set(userProviderId);
		terms.set(userQuery);
		dispatch('search');
	}

	function onkeydown(e: KeyboardEvent) {
		if (e.ctrlKey && e.code === 'KeyF') {
			// avoid Nextcloud page search
			e.stopPropagation();
			// avoid browser in-page search
			e.preventDefault();

			input.select();
		}
	}
</script>

<svelte:body on:keydown={onkeydown} />

<form method="get" on:submit|preventDefault={search}>
	<input bind:this={input} type="text" name="terms" bind:value={userQuery} />
	<select name="provider" bind:value={userProviderId}>
		<option value={PROVIDER_ALL} selected={userProviderId === PROVIDER_ALL}>
			{_t(PROVIDER_ALL_LABEL)}
		</option>
		{#each $providers as provider}
			<option value={provider.id} selected={userProviderId === provider.id}>{provider.name}</option>
		{/each}
	</select>
	<button type="button" on:click={clear} disabled={!userQuery && userProviderId === PROVIDER_ALL}>
		{_t('Clear')}
	</button>
	<button type="submit" disabled={!userQuery}>
		{_t('Search')}
	</button>
</form>

<style>
	input {
		width: 200px;
	}
</style>
