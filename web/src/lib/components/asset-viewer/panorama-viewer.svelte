<script lang="ts">
  import { fade } from 'svelte/transition';
  import LoadingSpinner from '../shared-components/loading-spinner.svelte';
  import { api, type AssetResponseDto } from '@api';

  export let asset: AssetResponseDto;

  const loadAssetData = async () => {
    const { data } = await api.assetApi.serveFile(
      { id: asset.id, isThumb: false, isWeb: false, key: api.getKey() },
      { responseType: 'blob' },
    );
    if (data instanceof Blob) {
      return URL.createObjectURL(data);
    } else {
      throw new TypeError('Invalid data format');
    }
  };
</script>

<div transition:fade={{ duration: 150 }} class="flex h-full select-none place-content-center place-items-center">
  <!-- the photo sphere viewer is quite large, so lazy load it in parallel with loading the data -->
  {#await Promise.all([loadAssetData(), import('./photo-sphere-viewer-adapter.svelte')])}
    <LoadingSpinner />
  {:then [data, module]}
    <svelte:component this={module.default} panorama={data} />
  {:catch}
    Failed to load asset
  {/await}
</div>
