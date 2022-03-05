<script context="module">
    // Get this page name, which will be used
    // for getting all available rows from the
    // table with the same name
    export const load = async ( { params } ) => {
        return { 
            props: { 
                tableName: params.table
            } 
        };
    }
</script>

<script>
    /**
    *  Imports
    */

    import { fade } from "svelte/transition";
    import { showAddRowModal, showConfirmDeleteRowModal, showEditRowModal } from "$stores/stores.js";
    import { Loading, Warning, LeftArrow } from "$icons/svg.js";
    import TableSearch from "$components/table/TableSearch.svelte";
    import ViewTable from "$components/table/ViewTable.svelte";
    import TablePage from "$components/table/TablePage.svelte";
    import AddRow from "$components/modal/AddRow.svelte";
    import ConfirmDeleteRow from "$components/modal/ConfirmDeleteRow.svelte";
    import EditRow from "$components/modal/EditRow.svelte";


    /**
    *  Variables
    */

    export let tableName; // Prop variable(s)
    tableName = tableName.charAt( 0 ).toUpperCase() + tableName.substring( 1 ); // Uppercase first letter only

    let tableRows = {};
    let tableHeaders = []; // Table columns
    let rowIndex = undefined; // Will be used to manipulate a specific table row
    let rowData = {}; // Row data that is/are deleted/added, to be sent to the database
    let selectedTableData = {}; // Data from the selected row
    let pageNumber = 0; // Current page number
    let lastPage = false; // Check if last page


    /**
    *  Functions
    */

    const changeRowIndex = ( index ) => {
        rowIndex = index + 1;
        rowData = JSON.stringify( Object.entries( tableRows )[ index ][ 1 ] );
    }

    const openAddRowModal = () => $showAddRowModal = true;

    const openConfirmDeleteRowModal = () => {
        if ( rowIndex === undefined ) return;
        $showConfirmDeleteRowModal = true;
    }

    const openEditRowModal = () => {
        if ( rowIndex === undefined ) return;
        $showEditRowModal = true;
    }

    const setSelectedTableData = ( rowIndex ) => selectedTableData = Object.entries( tableRows )[ rowIndex ];

    const tableRefresh = async () => {
        tableRows = {}; // Clear the existing table rows

        try {
            // Check if table rows is empty, then get table rows
            // TODO: rename [getTables()] -> [getRows()]
            if ( Object.keys( tableRows ).length === 0 ) await getRows();
            // Check if table headers are empty, then fill up with table columns
            if ( tableHeaders.length === 0 ) getHeaders();
        } catch ( err ) {
            throw new Error( `Something went wrong with getting the contents of this table: ${ tableName }` );
        }
    }

    const getHeaders = () => {
        // Loop to get table columns
        for ( const header of Object.keys( tableRows[0] ) ) {
            tableHeaders.push( header );
        }
    }

    const getRows = async () => {
        const req = await fetch( `http://localhost:8093/api/v1/tables/${ tableName.toLowerCase() }`, {
                method: 'GET',
                headers: {
                    'Content-Type': 'application/json'
                }
            } );

        tableRows = await req.json();

        // Check if returned JSON data is less than
        // 10, then that means it's the last page
        if ( Object.keys( tableRows ).length < 10 ) lastPage = true;
    }

    const searchRow = async ( column, query ) => {
        const req = await fetch( `http://localhost:8093/api/v1/tables/${ tableName.toLowerCase() }/?column=${ column }&search=${ query }`, {
                method: 'GET',
                headers: {
                    'Content-Type': 'application/json'
                }
            } );
        
        tableRows = await req.json();
        tableRefresh();
    }

    const switchPageRow = async ( page, move ) => {
        const req = await fetch( `http://localhost:8093/api/v1/tables/${ tableName.toLowerCase() }/${ move ? --page : ++page }`, {
                method: 'GET',
                headers: {
                    'Content-Type': 'application/json'
                }
            } );
        
        tableRows = await req.json();

        // Check if returned JSON data is less than
        // 10, then that means it's the last page
        if ( Object.keys( tableRows ).length < 10 ) lastPage = true;
        else lastPage = false; // else, it's not the last page

        pageNumber = page; // Switch page
        rowIndex = undefined; // Change back to the default value of [rowIndex]
        tableRefresh();
    }
</script>

<svelte:head>
    <title>Table - { tableName }</title>
</svelte:head>


<section class="back" 
in:fade={ { duration: 300 } }>
    <!-- Back button -->
    <a class="back-button" 
    href="/">
        <figure>
            <!-- Left arrow SVG component -->
            <LeftArrow class={ "back-arrow" } />
        </figure>

        <!-- Back button text -->
        <div class="back-text">
            <h3>Go back</h3>
        </div>
    </a>
</section>

{ #await tableRefresh() }
    <section class="loading">
        <header class="loading-header">
            <!-- Loading animated SVG component -->
            <figure>
                <Loading class={ "loading-icon" }/>
            </figure>

            <!-- Loading message -->
            <h3 class="loading-message">Please wait for the table to load...</h3>
        </header>
    </section>
{ :then _ }
    { #if $showAddRowModal }
        <AddRow tableName={ tableName }
        tableHeaders={ tableHeaders }
        tableRefresh={ tableRefresh }/>
    { /if }

    { #if $showConfirmDeleteRowModal }
        <ConfirmDeleteRow tableName={ tableName } 
        rowData={ rowData }
        tableRefresh={ tableRefresh }/>
    { /if }

    { #if $showEditRowModal }
        <EditRow tableName={ tableName } 
        tableHeaders={ tableHeaders }
        selectedTableData={ selectedTableData }
        tableRefresh={ tableRefresh }/>
    { /if }

    <main class="table" in:fade={ { duration: 300 } }>
        <div class="table-content">
            <!-- Table name -->
            <section class="table-overview">
                <header class="table-overview-header">
                    <h1>{ tableName }</h1>
                </header>

                <!-- Table rows buttons -->
                <div class="table-overview-options">
                    <!-- Add row button -->
                    <button 
                    class="table-overview-add"
                    on:click={ () => openAddRowModal() }>
                        <h3>Add row/s</h3>
                    </button>

                    <!-- Edit button -->
                    <button class="table-overview-button
                    { rowIndex === undefined ?
                    'table-overview-inactive' :
                    'table-overview-edit' }"
                    on:click={ () => openEditRowModal() }>
                        <h3>Edit row</h3>
                    </button>

                    <!-- Delete row button -->
                    <button class="table-overview-button
                    { rowIndex === undefined ? 
                    'table-overview-inactive' :
                    'table-overview-delete' } 
                    px-6"
                    on:click={ () => openConfirmDeleteRowModal() }>
                        <h3>Delete row</h3>
                    </button>
                </div>

                <TableSearch columns={ tableHeaders }
                searchRow={ searchRow }/>
            </section>

            <!-- View table component -->
            <ViewTable tableHeaders={ tableHeaders }
            tableRows={ tableRows }
            rowIndex={ rowIndex }
            changeRowIndex={ changeRowIndex }
            setSelectedTableData={ setSelectedTableData }/>

            <!-- Table search component -->
            <TablePage page={ pageNumber }
            lastPage={ lastPage }
            switchPageRow={ switchPageRow }/>
        </div>
    </main>
{ :catch err }
    <section class="loading">
        <header class="loading-header">
            <!-- Loading animated SVG component -->
            <figure>
                <Warning class={ "warning-icon" }/>
            </figure>

            <!-- Loading messages -->
            <div class="loading-error">
                <h3 class="loading-error-message">{ err.message }</h3>
                <h3 class="loading-error-submsg">Please check if this table exists, or rows exist in this table; ideally check your database</h3>
            </div>
        </header>
    </section>
{ /await }