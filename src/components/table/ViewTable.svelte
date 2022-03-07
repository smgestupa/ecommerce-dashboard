<script>
    /**
     * Imports
     */
    import { fade } from "svelte/transition";


    /**
     * Variables
     */
    export let tableHeaders, tableRows, rowIndex; // Prop variable(s)
    export let changeRowIndex, setSelectedRowData; // Prop function(s)

    let rowLength = Object.keys( tableRows ).length; // Pass in the length of [tableRows]
    if ( rowLength > 10 ) rowLength--; // If the rows are 11, then decrement the length since we only want 10 rows to show up
</script>

<section in:fade={ { duration: 300 } }>
    <!-- Table headers & rows -->
    <table class="view-table">
        <thead>
            <tr class="table-header">
                { #each tableHeaders as header }
                    <th>{ header }</th>
                { /each }
            </tr>
        </thead>
        <tbody>
            { #each { length: Object.keys( tableRows ).length } as _, index }
                <tr class="table-row 
                { rowIndex === index + 1 ?
                'bg-blue-100' :
                'bg-white' }"
                on:click={ () => { changeRowIndex( index ); setSelectedRowData( index ) } }>
                    { #each Object.values( tableRows[ index ] ) as column }
                        <td>{ column.length > 15 ? column.slice( 0, 16 ) + "..." : column }</td> 
                    { /each }
                </tr>
            { /each }
        </tbody>
    </table>
</section>