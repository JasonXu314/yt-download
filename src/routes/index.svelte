<script lang="ts">
	import Button from '$lib/components/Button.svelte';
	import Input from '$lib/components/Input.svelte';
	import axios from 'axios';
	import type { videoFormat, videoInfo } from 'ytdl-core';

	let link: string = '';
	let format: videoFormat | null = null;
	let info: videoInfo | null = null;

	function lookup() {
		axios.get<videoInfo>(`https://yt-download.us.aldryn.io/info?url=${link}`).then((res) => {
			info = res.data;
			console.log(info);
		});
	}

	function download() {
		axios
			.get<Blob>(`https://yt-download.us.aldryn.io?url=${link}&format=${format.container}&quality=${format.qualityLabel}`, { responseType: 'blob' })
			.then((res) => {
				console.log(res.data);
				const downloadUrl = URL.createObjectURL(res.data);
				const a = document.createElement('a');
				a.download = `${info.videoDetails.title}.${format.container}`;
				a.href = downloadUrl;
				a.click();
			});
	}
</script>

<Input bind:value={link} label="YouTube Link" />
<Button color="blue" on:click={lookup}>Search</Button>
{#if info}
	<div class="options-menu">
		Download Options:
		<select name="format" id="format" bind:value={format}>
			{#each info.formats.filter((format) => {
				if (format.container === 'mp4' && !format.hasVideo) {
					return false;
				}
				if (!format.qualityLabel) {
					return false;
				}
				return true;
			}) as format}
				<option value={format}
					>{format.container}
					{format.qualityLabel}
					{#if !format.hasAudio}
						(No Audio)
					{/if}</option
				>
			{/each}
		</select>
		<Button color="green" on:click={download}>Download</Button>
	</div>
{/if}

<style lang="scss">
	.options-menu {
		margin-top: 1em;
	}
</style>
