<script>
   
    export let data;
    export let rScale;
   
</script>

<div class="tooltip" style="position:absolute; left:{rScale(11)}px; top:{rScale(12)}px;">
    <div>Territory: {(data.territory)}</div>
    <div>Avg Delivery Time: {(data.averageDeliveryTime)}</div>
</div>
<style>
.tooltip{
    background-color: #1F2544;
    color: white;
    padding:0.45em;
    border-radius: 4px;
    transition: top 300ms ease, left 300ms ease;
}
</style>
