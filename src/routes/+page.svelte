<h1>Welcome to SvelteKit</h1>
<p>Visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to read the documentation</p>

<script>

    import * as d3 from 'd3';
    
    import { onMount } from 'svelte';

    let energyData = [];

    onMount(async () => {

        const res = await fetch('raw.githubusercontent.com_owid_energy-data_master_owid-energy-data.csv'); 

        const csv = await res.text();

        energyData = d3.csvParse(csv, d3.autoType) 

        console.log(energyData);

    });


</script>
