<script context="module">
    export const load = async ( { params } ) => {
        // Get this page name, to 
        // get available rows from
        // the specified table
        return { 
            props: { 
                tableName: params.table
            } 
        };
    }
</script>

<script>
    /**
     * Imports
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
     * Variables
     */
    export let tableName; // Prop variable(s)
    tableName = tableName.charAt( 0 ).toUpperCase() + tableName.substring( 1 ); // Uppercase first letter only

    let tableRows = {};
    let tableHeaders = []; // Table columns
    let rowIndex = undefined; // Will be used to manipulate a specific table row
    let rowData = {}; // Row data that is/are deleted/added, to be sent to the database
    let selectedRowData = {}; // Data from the selected row
    let pageNumber = 0; // Current page number
    let lastPage = false; // Check if last page


   /**
    * Will be used to select
    * row, based on an index
    * in the table tag
    * 
    * @param index The position of the row inside the table 
    */
    const changeRowIndex = ( index ) => {
        rowIndex = index + 1;
        rowData = JSON.stringify( Object.entries( tableRows )[ index ][1] );
    }

    /**
     * Will open the modal for
     * adding rows
     */
    const openAddRowModal = () => $showAddRowModal = true;

    /**
     * Will open the modal for
     * deleting a row
     */
    const openConfirmDeleteRowModal = () => {
        if ( rowIndex === undefined ) return;
        $showConfirmDeleteRowModal = true;
    }

    /**
     * Will open the modal for
     * editing a row
     */
    const openEditRowModal = () => {
        if ( rowIndex === undefined ) return;
        $showEditRowModal = true;
    }

    /**
     * Will be used to pass in
     * selected row data for
     * modifying
     * 
     * @param rowIndex The position of the row inside the table 
     */
    const setSelectedRowData = ( rowIndex ) => selectedRowData = Object.entries( tableRows )[ rowIndex ];

    /**
     * Will be used to initialize
     * table rows for the table
     * tag
     */
    const tableRefresh = async () => {
        try {
            if ( Object.entries( tableRows ).length === 0 ) await getRows(); // Initialize [tableRows]
            if ( tableHeaders.length === 0 ) getHeaders(); // If [tableHeaders] is empty, fill it with table columns

            // Check if returned JSON data is less 
            // than 10, then that means we're
            // on the last page
            if ( Object.keys( tableRows ).length <= 10 ) lastPage = true;
            else lastPage = false; // Else, we're on the last page
        } catch ( err ) {
            throw new Error( err.message ?? `Please check if this table exists; ideally, double check your database` );
        }
    }

    /**
     * Will be used to initialize
     * table headers for the table
     * tag
     */
    const getHeaders = () => {
        for ( const header of Object.keys( tableRows[0] ) ) { // Loop to get table columns
            tableHeaders.push( header );
        }
    }

    /**
     * Will be used to fetch available
     * rows from the database
     */
    const getRows = async () => {
        const req = await fetch( `http://localhost:8093/api/v1/tables/${ tableName.toLowerCase() }`, {
                method: 'GET',
                headers: {
                    'Content-Type': 'application/json'
                }
            } );
        
        // Check if the code is 302, that means no rows exist in this table
        if ( req.status == 302 ) throw new Error( 'No rows exist in this table, please add one to procceed' );
        tableRows = await req.json();
    }

    /**
     * Will be used to fetch certain
     * rows from the database, based
     * on a keyword(s)
     * 
     * @param column Where we want to find the value in the table
     * @param query Keyword(s) used to find a certain value
     */
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

    /**
     * Will be used to fetch rows,
     * starting from a specific
     * offset
     * 
     * @param page Offset used to get rows from a specific starting index
     * @param move True means previous page, false means next page
     */
    const switchPageRow = async ( page, move ) => {
        const req = await fetch( `http://localhost:8093/api/v1/tables/${ tableName.toLowerCase() }/${ move ? --page : ++page }`, {
                method: 'GET',
                headers: {
                    'Content-Type': 'application/json'
                }
            } );
        
        tableRows = await req.json();

        pageNumber = page; // Switch page offset
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
        selectedRowData={ selectedRowData }
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
            setSelectedRowData={setSelectedRowData}/>

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
                <h3 class="loading-error-message">Something went wrong with getting the contents of this table: { tableName }</h3>
                <h3 class="loading-error-submsg">{ err.message }</h3>
            </div>
        </header>
    </section>
{ /await }