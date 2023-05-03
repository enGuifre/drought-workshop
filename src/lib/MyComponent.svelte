<script>
    import PopupBar from './PopupBar.svelte';
	import LineChart from './Charts/LineChart.svelte';
    
    export let data;
	export let points_data;
    export let prevYearData;
    console.log(data)
    console.log(prevYearData)
    
    let delTime=20;
    let delay=0;
    
    // data.map(d=>d.delTime=delay+10)
    
    // let this_=data;
    //.slice(0,6)
    
    
    </script>
     {#if data && data.length>0}
        
                {#each data as {perc_volume, dia}}
                <PopupBar {perc_volume} 
                                    {dia}
                                    
                                
                                    />
                {/each}
        
            
        
    {/if}	 

	{#if points_data}
		<LineChart data={points_data}/>
	{/if}
    {#if prevYearData}
    <button on:click={() => {
        
         const event = new CustomEvent('updateCompare', {
        detail: {
          map_compare: true,
          currentInfo:data,
          prevYearInfo:prevYearData?prevYearData:'2022'
        }
      });
      dispatchEvent(event);
    }
        
      }>Compare satellite images</button>
    
    {/if}
    
    
    
    